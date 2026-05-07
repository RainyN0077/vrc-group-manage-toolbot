# Changelog

## v0.2.0 — 2026-05-07

### 🚀 新增功能

#### 插件系统
- **新增 `plugins/group_admin.py`** — 12 个 VRChat 群管理命令
  - `#gmembers` 分页群成员列表
  - `#ginvite` / `#gkick` / `#gban` / `#gunban` 成员管理
  - `#grole` 角色设置（模糊匹配）
  - `#grequests` / `#gaccept` / `#greject` 入群审核
  - `#gannounce` / `#gdelannounce` 公告管理
  - `#gaudit` 审核日志
- **新增 `plugins/user_bind.py`** — QQ ↔ VRChat 用户绑定系统
  - `#bind` BIO 验证码绑定（3 分钟过期）
  - `#bind force` 管理员强制绑定
  - `#confirm` / `#unbind` 确认与解绑
  - `#bindinfo` / `#whois` 绑定信息与状态查询
- **新增 `#2fa` 命令** — 交互式两步验证流程
- **新增 `#vrclLogin cookie=xxx`** — 浏览器 Cookie 直登，跳过用户名密码 + 2FA

#### 服务层
- **新增 `services/`** — 插件与后端之间的胶水层
  - `api_guard.py` — 速率控制（≥1.5s）+ 指数退避重试 + TTL 缓存
  - `permission.py` — 三级权限（USER/GROUP_ADMIN/SUPERUSER）+ VRChat 角色检查
  - `message_utils.py` — 消息格式化/分段（>3800 字符自动拆分）
  - `user_binding.py` — 绑定数据 CRUD（JSON 持久化）
  - `group_config.py` — 每 QQ 群独立配置

#### VRChat API 客户端
- 新增 14 个群管理 API 端点（成员/角色/公告/审核/封禁）
- 新增 `_request()` 统一请求层，HTTP 错误上抛给 api_guard
- 新增 `verify_2fa()` 独立两步验证方法
- User 模型增加 `bio`/`bioLinks` 字段
- Instance 模型增加 `instanceId` 别名

### 🔧 改进

- **命令触发**：移除所有 `rule=to_me()`，直接 `#` 前缀触发
- **命令前缀**：`/` 改为 `#`（避免 QQ 内 `/` 被转为表情）
- **API 调用**：旧方法（`get_user`/`get_instance`/`get_group` 等）改用 `_request()`，401/429 等错误正确传播到 api_guard 处理
- **登录**：POST + form-data 改为 GET + Basic Auth
- **启动**：修复 `nonebot.run(app="bot:app")` 导致的双重初始化
- **2FA**：分离 `verify_2fa()` 方法，不再每次重新 GET `/auth/user`

### 🐛 修复

- `Instance` 模型重复 `class Config` 导致 `populate_by_name` 被覆盖
- `FinishedException` 被 `try/except Exception` 捕获导致错误信息异常
- `#whereis` / `#instances` 参数类型错误（`MessageEvent` → `Message`）导致 handler 被跳过
- 三个插件各自独立 `get_vrc_client()` 单例 → 统一为 `utils/VRC` 内的共享实例
- `api_guard` 将 `None` 返回值当成功 → 配合 `_request()` 异常传播修复

### 📁 目录重构

```
utils/vrc_client.py  →  utils/VRC/vrc_client.py
utils/vrc_config.py  →  utils/VRC/vrc_config.py
utils/vrc_models.py  →  utils/VRC/vrc_models.py
utils/__init__.py    →  重新从 VRC 子包导出
+(新增) services/    →  5 个新文件
+(新增) plugins/group_admin.py, user_bind.py
```

---

## v0.1.0 — 2026-05-06

### 初始版本

- NoneBot2 + OneBot V11 框架搭建
- VRChat API 基础客户端（登录、用户查询、实例查询）
- `#instances` / `#whereis` / `#vrclLogin` 三个基础命令
