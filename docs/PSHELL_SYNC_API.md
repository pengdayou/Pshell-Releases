# Pshell 自建同步接口

本文以 **Pshell 2.7.6** 为接入基线，使用工作区协议 `protocolVersion: 2` 和数据结构 `schemaVersion: 1`。服务端可以用任意语言实现。同步内容是 SSH 会话配置、会话分组和可选的账号偏好，SSH 连接仍由客户端直接建立。

## 1. 服务地址与请求约定

准备一个 HTTPS 服务和测试账号，为该账号创建一个已启用的空工作区，初始 `revision` 为 `0`。在客户端打开「设置 → 云同步 → 启用同步」，把 `https://sync.example.com/sessions` 换成自己的完整服务地址，填入「服务器地址」并登录。下文的接口路径拼接在此地址之后。例如健康检查位于 `https://sync.example.com/sessions/health`，登录位于 `https://sync.example.com/sessions/api/auth/token`。

- 如果反向代理添加了路径前缀，把前缀一并填入服务器地址。例如服务器地址为 `https://sync.example.com/api/sessions` 时，登录路径为 `/api/sessions/api/auth/token`。
- 请求体使用 UTF-8 JSON，写请求用 `Content-Type: application/json`。健康检查和登录为公开路由。其余请求用 `Authorization: Bearer <token>`，服务端从令牌取得账号和设备身份。
- 工作区请求使用 `Accept: application/vnd.pshell.workspace+json`。客户端的版本标识形如 `X-Requested-With: Pshell/2.7.6`。若部署需处理跨域预检，允许来源发出 `GET`、`POST` 和这些请求头，并将预检与业务认证分开。
- 界面语言通过查询参数 `?locale=en-US` 或 `?locale=zh-CN` 传递。缺省或未知语言使用英文，响应以 `Content-Language` 声明实际文案语种。工作区写入在 `availableActions.localizedMessages` 为 `true` 时发送此参数。语言参数由接口层消费，提交数据和幂等摘要只处理业务字段。
- 登录和工作区响应通常含 `status`、`code`、`message`、`data`，成功用 HTTP 200、`status: true`、`code: 0`。健康检查用 `code: 0` 和 `data.status: "healthy"`，设备成功响应使用 `status`、`message`、`data`。认证、工作区和设备相关响应应禁止缓存，`message` 用于显示，`code` 用于程序判断。

## 2. 健康检查和登录

### `GET /health`

响应 HTTP 200。启动自建服务后可先用此接口核对路由和 TLS，再用客户端登录验证账号与数据库是否可用。

```json
{
  "code": 0,
  "message": "The service is healthy.",
  "data": {"status": "healthy", "timestamp": "2026-11-23 10:00:00"}
}
```

### `POST /api/auth/token`

请求体示例。用户名最长 64 字符，密码最长 128 字符。设备 ID 使用固定的小写 UUID，`platform` 取 `win32`、`darwin` 或 `linux`，应用版本最长 32 字符。经 HTTPS 登录时，客户端也可能发送设备名 `computerName` 和系统用户名 `systemUsername`。

```json
{
  "username": "demo-user",
  "password": "example-account-password",
  "device": {
    "id": "a2aa5678-0000-4000-8000-123456789abc",
    "platform": "win32",
    "version": "2.7.6",
    "computerName": "DEMO-PC",
    "systemUsername": "demo"
  }
}
```

成功响应示例。为每次设备登录签发随机不透明令牌，格式为 `psd1_` 加 64 个小写十六进制字符。客户端用此前缀识别受管设备，并为这类登录打开账号工作区。服务端应绑定账号与设备、保存令牌摘要，并让同一账号在同一设备上重新登录时替换旧令牌。以下令牌仅展示格式。

```json
{
  "status": true,
  "code": 0,
  "message": "Signed in.",
  "data": {
    "token": "psd1_0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
    "deviceManaged": true,
    "username": "demo-user",
    "really_name": "Demo User"
  }
}
```

登录失败示例。现有后端用 HTTP 200 和业务错误响应表示用户名或密码不正确。对这两种情况统一提示，并对重复登录失败限流。

```json
{"status": false, "code": 400, "message": "Incorrect username or password.", "data": null}
```

### `GET /api/auth/status`

返回当前账号、登录剩余秒数、服务能力和可选公告。账号标识 `accountId` 使用稳定非空字符串，须与工作区能力响应一致。`announcements` 每条包含稳定的 `id`、`title`、`message`，时间字段 `occurredAt` 使用 Unix 秒级时间戳。同一公告的 ID 应持续复用，以便客户端维持已读状态。最多返回 20 条，无公告时返回 `[]`。

```json
{
  "status": true,
  "code": 0,
  "message": "Your sign-in is valid.",
  "data": {
    "accountId": "demo-account-1",
    "username": "demo-user",
    "expiresIn": 86400,
    "protocolVersion": 1,
    "announcements": [],
    "encryptionPolicy": {"required": false, "revision": 0, "verifier": null},
    "capabilities": ["authStatus", "workspaceCapabilities", "deviceManagement"]
  }
}
```

身份探测只返回元信息。探测和读取工作区能力均不延长设备登录的闲置期限。账号工作区加密后，`encryptionPolicy` 应返回当前策略、版本和校验密文。策略无法读取时用 `null` 表示未知。用户名参与客户端密钥派生，已有加密账号的用户名应保持稳定。

## 3. 工作区能力、读取、提交与回执

### `GET /api/workspace/capabilities`

新建自托管账号可使用以下能力组合。看到 `state: "active"` 后，客户端会走新版工作区协议。`snapshot`、`commit`、`receipt`、`preview`、`activate`、`activationReceiptReplay` 以及两个 `legacyAccess` 字段必须显式提供布尔值。可选能力按实际实现声明，启用方法见后文。

```json
{
  "status": true,
  "code": 0,
  "message": "Operation completed.",
  "data": {
    "protocolVersion": 2,
    "schemaVersion": 1,
    "supportedSchemas": [1],
    "accountId": "demo-account-1",
    "state": "active",
    "schemaReady": true,
    "revision": 0,
    "availableActions": {
      "snapshot": true,
      "commit": true,
      "receipt": true,
      "preview": false,
      "activate": false,
      "activationReceiptReplay": false,
      "accountPreferences": false,
      "sealCredentials": false,
      "rotateCredentials": false,
      "purgeSessions": true,
      "localizedMessages": true
    },
    "legacyAccess": {"read": false, "write": false}
  }
}
```

### `GET /api/workspace`

返回当前版本与完整快照。下例为新账号的初始内容。实际的 `sessions` 和 `groups` 为记录数组。偏好以键值对象表示，空的 `preferences` 和 `devicePreferences` 也可用 `[]`。工作区协议 v2 中 `devicePreferences` 保持为空。

```json
{
  "status": true,
  "code": 0,
  "message": "Operation completed.",
  "data": {
    "protocolVersion": 2,
    "revision": 0,
    "snapshot": {
      "schemaVersion": 1,
      "encryptionRequired": false,
      "sessions": [],
      "groups": [],
      "preferences": [],
      "devicePreferences": []
    }
  }
}
```

### `POST /api/workspace/commit`

每次带上读取时的 `expectedRevision` 和唯一 `requestId`。请求 ID 区分大小写，由 16 至 64 个字母、数字、下划线或连字符组成，可以使用 UUID。下面的提交在空工作区内建立一个分组和一条 SSH 会话。

```json
{
  "protocolVersion": 2,
  "schemaVersion": 1,
  "requestId": "91bda498-55aa-4b73-8870-90ceaf051611",
  "expectedRevision": 0,
  "operations": [
    {
      "entity": "group",
      "action": "upsert",
      "id": "demo_ops",
      "value": {"id": "demo_ops", "name": "Operations", "sortOrder": 0}
    },
    {
      "entity": "session",
      "action": "upsert",
      "id": "demo_ssh_01",
      "value": {
        "id": "demo_ssh_01",
        "name": "Demo web server",
        "host": "ssh.example.com",
        "port": 22,
        "username": "operator",
        "authType": "password",
        "password": "",
        "encVersion": 0,
        "protocol": "ssh",
        "encoding": "utf-8",
        "termType": "xterm-256color",
        "keepaliveInterval": 30,
        "groupId": "demo_ops",
        "sortOrder": 0
      }
    }
  ]
}
```

成功返回新版本。下一次读取快照会看到该分组和会话，以及服务端写入的时间字段，例如 `"createdAt": "2026-11-23T02:00:00+00:00"`、`"updatedAt": "2026-11-23T02:00:00+00:00"`、`"deletedAt": null`。

```json
{
  "status": true,
  "code": 0,
  "message": "Operation completed.",
  "data": {
    "requestId": "91bda498-55aa-4b73-8870-90ceaf051611",
    "revision": 1,
    "currentRevision": 1,
    "replayed": false
  }
}
```

#### 实体与操作规则

- 会话和分组的 `id` 为 1 至 64 个字母、数字、下划线或连字符。分组需要 `id`、`name`，可选 `color`、`sortOrder` 及下述时间字段。
- 会话需要 `id`、`name`、`host`、`port`、`username`、`authType`、`encVersion`。端口为 1–65535 的整数，认证使用 `password` 或 `privateKey`。可选字段为 `password`、`privateKey`、`passphrase`、`protocol`、`encoding`、`termType`、`keepaliveInterval`、`groupId`、`groupName`、`color`、`sortOrder` 和下述时间字段。`protocol` 为 `ssh`，`encoding` 可用 `utf-8`、`gbk`、`gb2312`、`big5`，`termType` 可用 `xterm-256color`、`xterm`、`vt100`、`linux`，`keepaliveInterval` 为 0 到 300 的整数秒数。
- 名称和 SSH 用户名最长 100 字符，主机最长 255 字符，颜色最长 32 字符。排序值为非负整数。凭据字段中，私钥最长 131072 字符，密码和私钥口令各最长 16384 字符。与现有后端保持一致时，每批常规操作 1–500 项，单个快照最多各 500 条会话和分组，请求及快照上限分别为 4 MiB 和 8 MiB。
- `upsert` 与 `restore` 提供完整记录作为 `value`，并使 `value.id` 与操作 ID 一致。由服务端维护 ISO 8601 格式的 `createdAt`、`updatedAt`、`deletedAt`，正常记录的 `deletedAt` 为 `null`。已删除记录只能通过 `restore` 恢复。
- `delete` 和 `purge` 只传 `entity`、`action`、`id`。软删除保留记录并标记 `deletedAt`，客户端以回收站呈现。只有会话支持 `purge`。永久删除时移除整条记录及其凭据，把 ID 加入快照的 `purgedSessions` 字符串数组，阻止旧设备用相同 ID 恢复已经永久删除的会话。
- 每批操作不要重复处理同一个 `entity` 和 `id`。验证整个提交后的分组引用。有效会话引用的分组必须存在且处于有效状态，`groupName` 由服务端按分组名称填写。删除分组时，在同一批次移动或删除其中受影响的会话。

#### 版本和重试

1. 在同一个账号的事务内，先查 `(accountId, requestId)` 的回执，再检查当前版本。
2. 同一请求 ID 与相同内容重试时，返回原 `revision`、最新的 `currentRevision` 和 `replayed: true`。内容不同则返回 HTTP 409 和 `REQUEST_ID_REUSED`。
3. 新请求的 `expectedRevision` 必须匹配当前版本。成功时将新快照、递增后的版本和回执同时写入。任何一步失败都回滚这次提交。
4. `expectedRevision` 过期时，返回 HTTP 409 和 `REVISION_CONFLICT`，客户端会重新读取并核对差异。请求超时或响应丢失时，保留原请求 ID 和原内容等待重试。

### `GET /api/workspace/receipt?requestId=91bda498-55aa-4b73-8870-90ceaf051611`

从当前账号查询已有回执。成功响应提供该请求实际提交的版本。未找到时返回 HTTP 404、`code: "REQUEST_NOT_FOUND"`、`data: null`。

```json
{
  "status": true,
  "code": 0,
  "message": "Operation completed.",
  "data": {"requestId": "91bda498-55aa-4b73-8870-90ceaf051611", "revision": 1}
}
```

## 4. 登录设备接口

受管登录的设备列表只列出当前账号的有效设备。客户端读取 `currentDeviceId` 与 `devices`，并用 `signedInAt` 辨认新的登录。设备时间均为 Unix 秒级时间戳。

### `GET /api/devices`

```json
{
  "status": true,
  "message": "Operation completed.",
  "data": {
    "currentDeviceId": "a2aa5678-0000-4000-8000-123456789abc",
    "keyConfigured": false,
    "metadataSupported": true,
    "devices": [
      {
        "id": "a2aa5678-0000-4000-8000-123456789abc",
        "platform": "win32",
        "version": "2.7.6",
        "alias": null,
        "recognized": false,
        "computerName": "DEMO-PC",
        "systemUsername": "demo",
        "signedInAt": 1795428000,
        "lastSeenAt": 1795428000,
        "expiresAt": 1795514400
      }
    ]
  }
}
```

### 设备写入请求

以下请求成功均为 HTTP 200。JSON 示例中的密码、安全匙和设备 ID 仅用于说明字段，操作时使用各自测试账号的值。

- `POST /api/devices/logout` 接受 `{}`，使当前令牌失效。返回 `{"status":true,"message":"Signed out.","data":null}`。
- `POST /api/devices/metadata` 接受 `{"computerName":"DEMO-PC","systemUsername":"demo"}`，更新当前设备。返回 `{"status":true,"message":"Device name updated.","data":null}`。支持此接口时，在列表中提供 `metadataSupported: true`。
- `POST /api/devices/acknowledge` 接受 `{"deviceId":"a2aa5678-0000-4000-8000-123456789abc","alias":"Office PC"}`，确认当前账号下的设备并保存备注。返回 `{"status":true,"message":"Device confirmed.","data":null}`。空备注可清除别名。
- `POST /api/devices/security-key` 接受 `{"password":"example-account-password","securityKey":"example-device-key"}`。设置安全匙须校验账号密码，更换时另带 `currentSecurityKey` 校验旧值。安全匙为 12 至 64 字节，应与账号密码不同，服务端保存其密码哈希。返回 `{"status":true,"message":"Operation completed.","data":null}`。
- `POST /api/devices/revoke` 接受 `{"deviceId":"b2aa5678-0000-4000-8000-123456789abc","securityKey":"example-device-key","rememberToday":false}`，校验安全匙后撤销同一账号的另一台设备。普通成功响应与设置安全匙相同。当前设备可用退出接口结束自己的登录。

撤销请求若带 `rememberToday: true`，可在 `data` 中返回 `{"grant":"0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef","expiresAt":1795449600}`。这是随机、短时的验证票，绑定请求方登录令牌和当前安全匙。客户端后续可以只发送 `deviceId` 和 `grant`。票据至多有效到服务端当日结束或该登录到期，以先到者为准。安全匙更换后旧票据应失效。

## 5. 可选能力

### 端到端加密

在客户端启用后，`password`、`privateKey`、`passphrase` 三个字段的非空值为 `zv1:<Base64>`，其中二进制内容为 12 字节随机 IV 及带 16 字节认证标签的 AES-GCM 密文。`encVersion: 1` 表示加密，`0` 表示明文。服务端按原字符串保存和返回密文，主密码和密钥始终留在客户端。本协议的加密范围是凭据，主机、分组、会话名称等元数据仍参与服务端校验。

- 实现 `sealCredentials` 后，在能力响应将其置为 `true`。已有工作区启用加密时，客户端调用同一个提交接口，使用空 `operations` 和 `sealCredentials: { "verifier": "...", "sessions": [...] }`。列表按 `id` 覆盖该版本全部会话，包括回收站，只包含 ID 及三个凭据字段。空字段维持为空。一次事务更新全部密文、`encVersion: 1`、`encryptionRequired: true` 和 `encryptionVerifier`。
- 实现 `rotateCredentials` 后将其置为 `true`。更换主密码的提交使用同形状的 `rotateCredentials`、空 `operations`、旧 `expectedEncryptionVerifier` 和 `currentAccountPassword`。后者是当前云账号密码，按次校验后丢弃。原子更换全部密文和校验密文。以后携带会话值的普通提交也要核对 `expectedEncryptionVerifier`，避免旧设备用过期密钥写回。
- 启用后，`GET /api/workspace` 返回快照的布尔值 `encryptionRequired: true` 与 `encryptionVerifier`。身份检查的 `encryptionPolicy` 返回相同策略、当前版本和 `verifier`。服务端必须拒绝明文凭据写入和过期校验密文下的写入。未加密时省略快照的 `encryptionVerifier` 字段。

下面是独立的新工作区启用加密的请求示例。示例校验密文由 Pshell 客户端为账号 `demo-user` 和演示主密码 `docs-only-example-passphrase` 生成，仅用于测试空工作区。成功响应与普通提交同形，版本递增 1。

```json
{
  "protocolVersion": 2,
  "schemaVersion": 1,
  "requestId": "f35ab84c-6c3d-41bb-a706-cacac9df8419",
  "expectedRevision": 0,
  "operations": [],
  "sealCredentials": {
    "verifier": "zv1:QBfkqfr0YwYx/Lz2hFNpGnDbq4zmQPv/TV0AXUv758iNi9nI4iXeRgThDWix0KQBE8NYqQQgaJhjmuaY2nQc",
    "sessions": []
  }
}
```

### 账号偏好

支持时将 `availableActions.accountPreferences` 置为 `true`，接收工作区提交中的 `{"entity":"preference","action":"upsert","id":"terminal-font-size","value":{"value":14}}`。快照用键值对象表示，例如 `"preferences": {"terminal-font-size": 14}`。

- `terminal-font-size` 为 10 到 24 的整数。
- `appearance-theme` 为 `dark` 或 `light`。
- `appearance-eye-protection`、`terminal-cursor-blink`、`terminal-output-highlight` 为布尔值。

会话提交要保留未改动的账号偏好，本机的语言、背景图片和偏好覆盖由客户端保存。

### 历史迁移

全新自建服务可直接为账号开通空工作区，使用第 3 节的 `active` 能力响应。历史服务可能保留 `GET /api/index`、`GET /api/status`、`POST /api/save`、`POST /api/sync`，以及 `POST /api/workspace/migration/preview` 和 `POST /api/workspace/migration/activate`。这些路径用于已有旧协议数据的迁移。新版已启用工作区的 `legacyAccess.read` 和 `legacyAccess.write` 均为 `false`。

## 6. 错误与接入验证

重要错误采用对应的 HTTP 状态码及稳定的业务码。设备被撤销时返回 HTTP 401。

```json
{
  "status": false,
  "code": "DEVICE_REVOKED",
  "message": "This device’s access has been revoked.",
  "data": {"accountId": "demo-account-1"}
}
```

- **401** 使用 `TOKEN_EXPIRED` 表示失效登录，用 `DEVICE_REVOKED` 表示明确撤销。撤销时提供 `data.accountId`，客户端据此清理对应账号工作区。
- **409** 使用 `REVISION_CONFLICT` 表示版本已变更，`REQUEST_ID_REUSED` 表示请求 ID 与已保存的内容不匹配，`E2E_POLICY_CHANGED` 表示加密策略已变化。普通冲突响应的 `data` 为 `null`。
- **400** 用于拒绝无效字段或 `E2E_REQUIRED` 等确切的校验错误。**426** 可用 `CLIENT_UPDATE_REQUIRED` 要求更新客户端，并在 `data.minSupportedVersion` 返回最低版本。**5xx** 用于服务故障，使客户端保留掉线恢复所需的状态。

提交前读到的版本已被另一设备更新时，返回 HTTP 409。

```json
{
  "status": false,
  "code": "REVISION_CONFLICT",
  "message": "The cloud configuration has changed. Review the differences and confirm again.",
  "data": null
}
```

可按以下顺序验收自建实现。

1. 用两个独立的测试账号和设备验证登录、身份、能力和空快照，确认记录及回执按令牌所绑定的账号隔离。
2. 用第一台设备提交分组和会话，再用第二台读取并确认变化。重放同一提交返回原回执，旧版本的新请求返回冲突。
3. 测试删除、回收站恢复及永久删除。将一个旧快照中的已永久删除 ID 再次提交，确认它仍保持移除状态。
4. 如支持端到端加密，用测试账号完成首次加密和更换主密码，验证存储及日志无明文 SSH 凭据、账号密码或令牌，另一设备可用新主密码解锁。
5. 撤销一个设备并复查该令牌，随后模拟一次服务不可达，确认客户端分别显示撤销和服务不可达。
