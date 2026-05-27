# 掌上黄金埠 AI 开发指导文档

> 用途：给 AI 编程助手或新开发人员快速理解项目、接手开发、扩展功能。  
> 优先级：如本文档与 `PROJECT_REBUILD_CONTEXT.md`、`README.md`、`DEPLOYMENT_GUIDE.md` 有冲突，以 `PROJECT_REBUILD_CONTEXT.md` 的业务、接口、数据模型为最高优先级。

## 1. AI 阅读顺序

AI 开始开发前，必须先按顺序阅读：

1. `README.md`：快速理解项目结构、启动方式和核心约定。
2. `PROJECT_REBUILD_CONTEXT.md`：理解业务定位、页面、接口、数据模型、权限、验收标准。
3. `DEPLOYMENT_GUIDE.md`：理解本地开发、Docker、生产部署和上线注意事项。
4. 本文档：理解 AI 开发时的执行边界、禁止事项和检查清单。

不要只读本文档后直接开发。

## 2. 项目一句话概述

“掌上黄金埠”是面向江西省上饶市余干县黄金埠镇及周边片区的本地生活服务聚合平台，首期聚焦装修、家具、门窗、脚手架等建材家居类商家和商品信息展示。

项目不是电商交易系统，首版重点是：

- 前台展示和检索本地商家、商品。
- 用户提交咨询或预约意向。
- 后台管理分类、轮播、门店、商品、线索、用户/商户。
- 支持 PC、移动端 H5、小程序 WebView、App WebView。

## 3. 当前技术栈

前端：

- Vue 2.7
- Vue Router 3，hash 模式
- Vuex 3
- Ant Design Vue 1.7
- Axios
- Less
- Vue CLI 5

后端：

- Node.js
- Express 4
- Sequelize 6
- MySQL 8
- JWT
- bcryptjs
- Joi
- Winston

部署推荐：

- 前端：`npm run build` 后由 Nginx 托管 `dist`。
- 后端：PM2 启动 `backend/src/index.js`。
- 数据库：MySQL，生产可迁移到云数据库。

## 4. 项目结构

当前仓库是 monorepo 结构：根目录是前端，`backend/` 是后端。

```text
.
├── src/                         # Vue 前端源码
│   ├── api/                     # Axios 封装和业务 API
│   ├── assets/styles/           # 全局样式
│   ├── components/              # 通用组件
│   ├── directives/              # 自定义指令
│   ├── layouts/                 # 前台/后台布局
│   ├── router/                  # 路由配置
│   ├── store/                   # Vuex
│   ├── utils/                   # 登录、存储、环境工具
│   ├── views/frontend/          # 前台页面
│   └── views/admin/             # 后台页面
├── public/                      # 静态资源和 logo
├── backend/                     # Express 后端
│   ├── src/config/              # 数据库、认证配置
│   ├── src/controllers/         # 控制器
│   ├── src/middleware/          # 中间件
│   ├── src/models/              # Sequelize 模型
│   ├── src/routes/              # 路由聚合
│   ├── src/utils/               # 响应、日志工具
│   ├── database.sql             # 建表和初始化脚本
│   └── Dockerfile
├── docker-compose.yml
├── Dockerfile.frontend
├── vue.config.js
├── package.json
├── README.md
├── PROJECT_REBUILD_CONTEXT.md
└── DEPLOYMENT_GUIDE.md
```

## 5. 功能边界

### 5.1 前台功能

必须覆盖：

- 首页：轮播图、分类、推荐门店、推荐商品、最新门店。
- 分类门店列表：支持关键词、片区、排序、分页。
- 门店详情：门店资料、联系方式、店内商品。
- 商品详情：商品图片、描述、规格、价格、所属门店、相关推荐。
- 搜索：混合搜索门店和商品。
- 我的、收藏：需要前台登录。
- 意向表单：用户提交咨询或预约意向。

注意：

- 前台允许匿名浏览。
- 收藏、我的、需要身份的操作要统一走全局登录唤醒。
- 前端页面不要硬编码正式业务数据，展示内容应来自后端 API 或初始化数据。
- 文档中不要再泛写“二级、三级、四级详情页”，当前明确路由以分类、门店、商品详情为准。

### 5.2 后台功能

必须覆盖：

- 登录与仪表盘。
- 分类管理。
- 轮播图管理。
- 门店管理。
- 商品管理。
- 线索管理。
- 用户/商户管理。

角色：

- `super_admin`：平台超管，拥有全站后台权限。
- `merchant`：商家，只能访问和操作授权范围内的数据。
- `user`：普通前台用户。

权限要求：

- 后台页面菜单隐藏只是体验优化，不是安全控制。
- 商户数据隔离必须由后端强制实现。
- 不允许只靠前端判断角色来保护接口。

## 6. API 和数据约定

所有后端接口统一挂载在 `/api` 下。

统一响应格式：

```json
{
  "code": 0,
  "message": "ok",
  "data": {}
}
```

前端成功判断：

```text
code === 0 或 code === 200
```

Axios baseURL：

```js
process.env.VUE_APP_API_BASE_URL || "/api";
```

鉴权：

- 管理端接口使用 `admin-token`。
- 前台接口优先使用 `frontend-token`。
- Token 通过 `Authorization: Bearer <token>` 传递。

核心接口以 `PROJECT_REBUILD_CONTEXT.md` 第 6 节为准，不能随意改路径。

## 7. 数据模型

核心表以当前项目为准：

- `users`：用户、管理员、商家。
- `categories`：分类。
- `banners`：轮播图。
- `stores`：门店。
- `products`：商品。
- `leads`：意向表单 / 销售线索。
- `favorites`：用户收藏。

不要新增独立 `merchants` 表，除非明确完成迁移设计。当前商户身份通过 `users.role = merchant` 表达。

需要特别注意：

- 门店字段是 `name`。
- 商品字段是 `title`。
- 收藏建议唯一约束为 `userId + type + targetId`。
- 商家和门店关系当前可能同时存在 `stores.merchantId` 和 `users.storeIds`，如重构需统一模型并同步权限判断。

## 8. AI 开发硬性规则

AI 修改项目时必须遵守：

- 保持根目录前端、`backend/` 后端的 monorepo 结构。
- 优先复用现有组件、API 封装、Vuex、路由和后端工具函数。
- 后端正式接口必须走 `backend/src/index.js` 和 `backend/src/routes/index.js`。
- 生产环境不要使用 `server/preview-api.js`。
- 不要重新引入 `src/mock/*` 作为正式数据层。
- 不要把业务数据长期硬编码在 Vue 页面里。
- 不要改变 `/api` 接口挂载前缀，除非同步修改前端、文档和部署配置。
- 不要把 `.env`、数据库密码、JWT 密钥、`node_modules/`、`dist/`、日志提交到仓库。
- 中文文档和源码统一使用 UTF-8。
- 若发现文档和代码不一致，先检查 `PROJECT_REBUILD_CONTEXT.md`，再检查实际代码；修复时说明取舍。

## 9. 推荐开发顺序

AI 从零重建或大改时，建议按这个顺序：

1. 确认数据模型和 `backend/database.sql`。
2. 确认统一响应、错误处理、日志、数据库连接。
3. 实现 JWT 登录和角色权限中间件。
4. 实现前台接口：首页、分类列表、门店详情、商品详情、搜索、参考数据。
5. 实现收藏和线索接口。
6. 实现后台 CRUD：分类、轮播、门店、商品、线索、用户。
7. 确认前端 `src/api/request.js` 和 `src/api/service.js`。
8. 实现前台页面和登录唤醒。
9. 实现后台页面和角色菜单。
10. 做 PC、移动端 H5 和 WebView 适配检查。
11. 做本地启动、构建、上线验收。

小改动时不必机械执行完整顺序，但不能跳过受影响范围的验证。

## 10. 任务检查清单

前端：

- [ ] 页面数据来自 API 或后端初始化数据。
- [ ] 首页、分类、门店详情、商品详情、搜索能正常加载。
- [ ] 收藏、我的能触发登录。
- [ ] 意向表单能提交。
- [ ] 网络错误和鉴权失败有友好提示。
- [ ] PC 和移动端布局不破版。

后端：

- [ ] `/api/health` 正常。
- [ ] 统一返回 `{ code, message, data }`。
- [ ] 后台接口需要鉴权。
- [ ] 商户接口有后端数据隔离。
- [ ] 分页列表返回 `list`、`total`、`page`、`pageSize`。
- [ ] 参数校验和错误处理明确。

数据库：

- [ ] 表结构和模型一致。
- [ ] 初始化数据能支撑首页、分类、后台登录。
- [ ] 默认管理员密码上线前可修改。

部署：

- [ ] `npm run build` 成功。
- [ ] 生产 API 地址写入 `.env.production`。
- [ ] PM2 启动后端。
- [ ] Nginx 指向 `dist` 并反代 `/api`。
- [ ] MySQL 未暴露公网。
- [ ] HTTPS 证书有效。

## 11. 当前不建议优先做的功能

首版不要把重点放在这些功能上：

- 在线支付。
- 即时聊天。
- 复杂会员积分体系。
- 用户评价体系。
- 商家自助入驻审核流。
- 精准地图定位。
- 复杂多管理员权限体系。
- 推荐算法。

这些功能可以作为后续迭代，但不能影响首版“展示 + 咨询线索 + 后台管理”的稳定性。

## 12. 可扩展方向

首版稳定后可以考虑：

- 图片上传模块，接入 OSS、COS、七牛云或本地上传。
- 短信验证码和线索通知。
- 微信授权登录、小程序 WebView、公众号 H5。
- App WebView JSBridge。
- 地图导航和距离展示。
- 埋点统计：页面曝光、搜索、点击电话、收藏、提交预约。
- 线索转化统计和后台运营看板。

## 13. 给 AI 的直接任务提示词

需要让 AI 接手开发时，可以直接使用：

```text
请先阅读 README.md、PROJECT_REBUILD_CONTEXT.md、DEPLOYMENT_GUIDE.md 和 AI_DEVELOPMENT_GUIDE.md，然后基于当前代码完成任务。

必须遵守：
1. 当前项目是根目录 Vue 2.7 前端 + backend/ Express 后端的 monorepo。
2. 后端接口统一挂载 /api，响应格式为 { code, message, data }。
3. 数据库为 MySQL，核心表为 users、categories、banners、stores、products、leads、favorites。
4. 前台允许匿名浏览，收藏/我的等需要身份的操作统一走登录唤醒。
5. 后台角色为 super_admin、merchant、user，商户数据隔离必须由后端强制实现。
6. 不要使用 server/preview-api.js 作为生产接口，不要重新引入 mock 作为正式数据层。
7. 修改接口、字段、路由、权限时，同步检查前端调用、后端实现和文档。
8. 完成后说明改动文件、验证方式、未验证风险。
```

## 14. 相关文档

- `README.md`：快速启动、基本结构和核心功能概览。
- `PROJECT_REBUILD_CONTEXT.md`：详细业务定位、路由、API、数据模型、权限和 AI 重建上下文。
- `DEPLOYMENT_GUIDE.md`：本地开发、Docker、生产部署、阿里云上线和维护命令。
