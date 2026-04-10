# 钉钉文档企业版技能 (dingtalk-doc-enterprise)

通过钉钉开放平台企业 API 管理钉钉文档，支持多用户场景，自动从钉钉连接器获取当前用户身份。

> 背景：我在使用openclaw和钉钉的官方连接器，但是他们不支持这些功能（最新版本0.8.13）。
>
> 这是我的issue： https://github.com/DingTalk-Real-AI/dingtalk-openclaw-connector/issues/456
>
> 网上找遍了资源，也加入了钉钉开发群，都说不支持、还说没有操作文档的API。
>
> 连接器的文档说["钉钉文档能力依赖 MCP（Model Context Protocol）提供底层 tool"](https://github.com/DingTalk-Real-AI/dingtalk-openclaw-connector?tab=readme-ov-file#%E9%92%89%E9%92%89%E6%96%87%E6%A1%A3docs%E4%B8%8E-mcpdocs) ，这种方案全是权限问题。
>
> 于是有了这个玩具。

---

## 🚀 快速开始

### 1. 环境变量配置

在 OpenClaw Gateway 环境中添加以下环境变量：

```powershell
# 钉钉企业内部应用凭证
$env:DINGTALK_CLIENTID="your_client_id"
$env:DINGTALK_CLIENTSECRET="your_client_secret"
```

**获取方式：**
1. 登录 [钉钉开放平台](https://open.dingtalk.com/)
2. 进入「应用开发」→「企业内部开发」
3. 创建或选择已有应用
4. 在「应用详情」页面获取 ClientId 和 ClientSecret

---

### 2. 申请应用权限

在钉钉开放平台「权限管理」中申请以下权限：

| 权限码 | 用途 | 必需 |
|--------|------|------|
| `qyapi_get_member` | 获取用户信息 | ✅ |
| `Storage.File.Read` | 读取文档内容 | ✅ |
| `Storage.File.Write` | 编辑/创建文档 | ✅ |

**申请步骤：**
1. 进入应用管理页面
2. 点击「权限管理」
3. 搜索上述权限码并申请
4. 等待管理员审批通过

---

## ✨ 核心特性

- ✅ **自动身份识别** - 从钉钉连接器消息自动获取发送者身份
- ✅ **多用户支持** - 企业内所有用户均可使用
- ✅ **权限隔离** - 每个用户只能操作自己有权限的文档
- ✅ **完整 CRUD** - 支持读取、创建(假的，不能创建)、编辑、删除文档

---

## 📖 使用方式

### 自动触发（推荐）

**发送钉钉文档链接即可自动读取：**
```
https://alidocs.dingtalk.com/i/nodes/xxx
```

**文档链接 + 操作指令：**
```
帮我总结这篇文档：https://alidocs.dingtalk.com/i/nodes/xxx
在这篇文档里添加一段测试数据：https://alidocs.dingtalk.com/i/nodes/xxx
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

---

## 🔧 工作原理

```
用户发送消息 
    ↓
钉钉连接器接收
    ↓
OpenClaw 提取 sender_id
    ↓
查询用户 unionId
    ↓
作为 operatorId 调用钉钉 API
    ↓
返回结果给用户
```

---

## ⚠️ 注意事项

### 权限说明
- 每个用户只能操作**自己有权限访问**的文档
- 企业公开文档对所有用户可见
- 私有文档需要单独授权

### 限制
- 单次内容最大长度：50000 字符
- 频率限制：100 次/分钟/文档
- 块元素更新：目前仅支持段落类型

---

## 🐛 常见问题

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| `forbidden.accessDenied` | 无文档权限 | 确认用户对文档有访问权限 |
| `docNotExist` | 文档不存在 | 检查文档 ID 是否正确 |
| `operatorId invalid` | 用户身份无效 | 检查钉钉连接器配置 |
| `paramError` | 参数错误 | 检查文档链接格式 |

---

## 📚 API 参考

支持的钉钉文档 API 端点：

| 操作 | 端点 | 方法 |
|------|------|------|
| 覆写文档 | `/v1.0/doc/suites/documents/{docKey}/overwriteContent` | POST |
| 查询块元素 | `/v1.0/doc/suites/documents/{dentryUuid}/blocks` | GET |
| 插入块元素 | `/v1.0/doc/suites/documents/{dentryUuid}/blocks` | POST |
| 更新块元素 | `/v1.0/doc/suites/documents/{dentryUuid}/blocks/{blockId}` | PUT |
| 删除块元素 | `/v1.0/doc/suites/documents/{dentryUuid}/blocks/{blockId}` | DELETE |

---

**最后更新：** 2026-04-10  
**版本：** 1.0  
**作者：** Ash · ShyZhen
