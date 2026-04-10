---
name: dingtalk-doc-enterprise
description: 钉钉文档企业版（多用户支持）。通过钉钉企业 API 管理文档，自动从钉钉连接器获取当前用户身份。支持读取、创建、编辑、删除文档。Design by Ash。
requires:
  bins:
    - node  # 需要 Node.js 运行环境
env:
  read:
    - "DINGTALK_CLIENTID (必需，钉钉应用 ClientId，从 ~/.openclaw/.env 加载)"
    - "DINGTALK_CLIENTSECRET (必需，钉钉应用 ClientSecret，从 ~/.openclaw/.env 加载)"
    - "DINGTALK_OPERATOR_ID (可选，操作人 unionId，不设置则自动获取)"
    - "OPENCLAW_SENDER_ID (可选，由 OpenClaw 自动注入)"
    - "OPENCLAW_SENDER_NAME (可选，由 OpenClaw 自动注入)"
---

# 钉钉文档企业版技能 (dingtalk-doc-enterprise)

## 功能描述

通过钉钉开放平台企业 API 管理钉钉文档，**支持多用户场景**，自动从钉钉连接器消息中获取当前用户身份。

**核心特性：**
- ✅ 自动获取当前用户身份（从钉钉连接器）
- ✅ 支持企业内所有用户使用
- ✅ 完整的增删改查功能
- ✅ 权限隔离（每个用户只能操作自己有权限的文档）

## 触发场景

### 自动触发（推荐）
- **用户发送钉钉文档链接** → 自动读取
  ```
  https://alidocs.dingtalk.com/i/nodes/xxx
  ```

- **用户发送文档链接 + 操作指令** → 自动执行
  ```
  帮我读取这篇文档：https://alidocs.dingtalk.com/i/nodes/xxx
  在这篇文档里添加一段：https://alidocs.dingtalk.com/i/nodes/xxx
  ```

### 指令触发
| 操作 | 指令示例 |
|------|---------|
| 读取文档 | `读取钉钉文档 <链接>` |
| 创建文档 | `创建钉钉文档 <标题> [内容]` |
| 更新文档 | `更新文档 <链接> 内容为...` |
| 追加内容 | `在文档末尾追加：<链接> 内容...` |
| 删除块元素 | `删除文档第 X 段：<链接>` |
| 查询结构 | `列出文档结构：<链接>` |

## 配置

### 环境变量

**配置方式：** 在 `~/.openclaw/.env` 文件中添加（OpenClaw 会自动加载）

```bash
# ~/.openclaw/.env
DINGTALK_CLIENTID=dingxxxxxx
DINGTALK_CLIENTSECRET=your_secret
```

| 变量 | 说明 | 必需 | 获取方式 |
|------|------|------|---------|
| `DINGTALK_CLIENTID` | 企业内部应用 ClientId | ✅ | 钉钉开放平台 → 应用详情 |
| `DINGTALK_CLIENTSECRET` | 企业内部应用 ClientSecret | ✅ | 钉钉开放平台 → 应用详情 |

**注意：** 不需要在技能代码中直接读取 `.env` 文件，OpenClaw 会在启动时自动加载 `~/.openclaw/.env` 到环境变量。

### OpenClaw 集成

在 OpenClaw 钉钉连接器场景下，**无需手动配置 operatorId**，系统会自动从消息元数据中提取发送者信息。

**自动注入的环境变量：**
- `OPENCLAW_SENDER_ID` - 发送者 ID（由 OpenClaw 注入）
- `OPENCLAW_SENDER_NAME` - 发送者名称（由 OpenClaw 注入）

### 权限要求

| 权限码 | 用途 | 申请方式 |
|--------|------|---------|
| `Storage.File.Read` | 读取文档 | 开放平台 → 权限管理 |
| `Storage.File.Write` | 编辑文档 | 开放平台 → 权限管理 |
| `qyapi_get_member` | 获取用户信息（可选） | 开放平台 → 权限管理 |

### 配置示例（Gateway 环境）

在 OpenClaw Gateway 配置中添加：

```powershell
$env:DINGTALK_CLIENTID="dingaixxxxxxxxxxxxxx"
$env:DINGTALK_CLIENTSECRET="9qe2buxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

## 使用示例

### 读取文档
```
读取这篇文档：https://alidocs.dingtalk.com/i/nodes/amweZ92Pxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 创建文档
```
创建一个新文档，标题是"周报模板"，写入本周工作总结
```

### 更新文档
```
更新文档内容为：https://alidocs.dingtalk.com/i/nodes/xxx
# 新标题
新内容
```

### 追加内容
```
在这篇文档末尾追加一段：https://alidocs.dingtalk.com/i/nodes/xxx
## 新增章节
这是新增的内容
```

### 查询结构
```
列出这篇文档的结构：https://alidocs.dingtalk.com/i/nodes/xxx
```

## 多用户支持

### 工作原理

```
用户 A 发送消息 → 钉钉连接器 → OpenClaw
                          ↓
                   提取 sender_id
                          ↓
                   查询用户 unionId
                          ↓
                   作为 operatorId 调用 API
                          ↓
                   返回结果给用户 A
```

### 权限隔离

| 用户 | 可访问的文档 |
|------|-------------|
| 用户 A | A 有权限的文档 |
| 用户 B | B 有权限的文档 |
| 机器人 | 机器人账号有权限的文档 |

**每个用户只能操作自己有权限的文档**，不会越权访问。

### 用户身份获取

**优先级：**
1. **钉钉连接器消息元数据** - `sender_id`（最准确）
2. **通讯录 API 查询** - 通过手机号查询 unionId
3. **环境变量默认值** - `DINGTALK_OPERATOR_ID`（备用）

## API 参考

### 支持的 API 端点

| 操作 | API 端点 | 方法 |
|------|---------|------|
| 覆写文档 | `/v1.0/doc/suites/documents/{docKey}/overwriteContent` | POST |
| 查询块元素 | `/v1.0/doc/suites/documents/{dentryUuid}/blocks` | GET |
| 插入块元素 | `/v1.0/doc/suites/documents/{dentryUuid}/blocks` | POST |
| 更新块元素 | `/v1.0/doc/suites/documents/{dentryUuid}/blocks/{blockId}` | PUT |
| 删除块元素 | `/v1.0/doc/suites/documents/{dentryUuid}/blocks/{blockId}` | DELETE |
| 追加文本 | `/v1.0/doc/suites/documents/{dentryUuid}/paragraphs/{blockId}/text` | POST |

### 请求参数

| 参数 | 位置 | 说明 |
|------|------|------|
| `docKey` / `dentryUuid` | Path | 文档 ID |
| `operatorId` | Query | 操作人 unionId（自动获取） |
| `x-acs-dingtalk-access-token` | Header | 访问凭证（自动获取） |
| `content` | Body | 文档内容（更新时） |

## 错误处理

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| `paramError` | 参数错误 | 检查文档 ID 格式 |
| `forbidden.accessDenied` | 无权限 | 确认用户对文档有权限 |
| `docNotExist` | 文档不存在 | 检查文档 ID 是否正确 |
| `blockNotExist` | 块元素不存在 | 检查块 ID 是否正确 |
| `operatorId invalid` | 操作人无效 | 检查用户身份获取 |

## 限制

- 单次内容最大长度：50000 字符
- 频率限制：100 次/分钟/文档
- 块元素更新：目前仅支持段落类型

## 安全说明

### 权限隔离

- 每个用户只能访问自己有权限的文档
- 不会越权访问其他用户的私有文档
- 企业公开文档对所有用户可见

### 审计日志

所有操作都会记录：
- 操作人（operatorId）
- 操作时间
- 操作类型
- 目标文档

### 凭证管理

- ClientSecret 妥善保管，不要泄露
- Access Token 自动获取和刷新
- 建议使用 HTTPS 传输

---

_最后更新：2026-04-09_
