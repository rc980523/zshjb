# 掌上黄金埠 AI 开发指南

> 用途：给 AI 编程助手或新开发人员快速理解项目、接手开发、扩展功能。
>
> 当前仓库暂未包含 `README.md`、`PROJECT_REBUILD_CONTEXT.md`、`DEPLOYMENT_GUIDE.md` 等配套文档时，以本文为主要开发依据。后续如果补充了这些文档，业务口径和接口契约应以 `PROJECT_REBUILD_CONTEXT.md` 为最高优先级。

## 1. 项目概述

“掌上黄金埠”是面向江西省上饶市余干县黄金埠镇及周边片区的本地生活服务聚合平台。

首期重点是本地建材家居类商家和商品展示，包括装修、家具、门窗、脚手架等类型。项目不是完整电商交易系统，首版不做在线支付、订单履约和复杂售后。

首版核心目标：

- 前台展示和搜索本地商家、商品。
- 用户提交咨询或预约意向。
- 后台管理分类、轮播、门店、商品、线索、用户和商户。
- 支持 PC、移动端 H5、小程序 WebView、App WebView。

## 2. 技术栈

前端：

- Vue 2.7
- Vue Router 3，使用 hash 模式
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

部署建议：

- 前端执行 `npm run build`，由 Nginx 托管 `dist/`。
- 后端使用 PM2 启动 `backend/src/index.js`。
- 数据库使用 MySQL，生产环境可迁移到云数据库。

线上环境信息：

- 计划线上域名：`zshjb.com`。
- 当前公网 IP：`8.145.52.103`。
- 备案状态：备案待审核。
- 备案通过前，不要在文档或代码中假设 `https://zshjb.com` 已经可稳定访问。
- 生产环境上线前，应确认域名解析、备案状态、HTTPS 证书、Nginx 反向代理和 `/api` 转发均已完成。

## 3. 推荐项目结构

当前项目按 monorepo 组织：根目录为 Vue 前端，`backend/` 为 Express 后端。

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
├── DEPLOYMENT_GUIDE.md
└── AI_DEVELOPMENT_GUIDE.md
```

如果实际目录与上述结构不同，AI 开发时应先检查现有代码，再决定是否补齐目录，不要盲目重建整套结构。

## 4. 前台功能边界

必须覆盖：

- 首页：轮播图、分类入口、推荐门店、推荐商品、最新门店。
- 分类门店列表：支持关键词、片区、排序、分页。
- 门店详情：门店资料、联系方式、店内商品。
- 商品详情：商品图片、描述、规格、价格、所属门店、相关推荐。
- 搜索：混合搜索门店和商品。
- 我的、收藏：需要前台登录。
- 意向表单：用户提交咨询或预约意向。

前台约束：

- 前台允许匿名浏览。
- 收藏、我的、提交意向等需要身份的操作，应统一触发登录提醒。
- 页面展示数据应来自后端 API 或数据库初始化数据，不要在 Vue 页面中长期硬编码正式业务数据。
- 当前明确路由以分类、门店、商品详情为主，不要泛化成“二级/三级/四级详情页”。

## 5. 后台功能边界

必须覆盖：

- 登录与仪表盘。
- 分类管理。
- 轮播图管理。
- 门店管理。
- 商品管理。
- 线索管理。
- 用户/商户管理。

角色：

- `super_admin`：平台超级管理员，拥有全站后台权限。
- `merchant`：商户，只能访问和操作授权范围内的数据。
- `user`：普通前台用户。

权限要求：

- 后台菜单隐藏只是体验优化，不是安全控制。
- 商户数据隔离必须由后端强制实现。
- 不允许只依赖前端判断角色来保护接口。

## 6. API 约定

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

```js
response.code === 0 || response.code === 200
```

Axios baseURL：

```js
process.env.VUE_APP_API_BASE_URL || "/api";
```

鉴权：

- 管理端接口使用 `admin-token`。
- 前台接口优先使用 `frontend-token`。
- Token 通过 `Authorization: Bearer <token>` 传递。

分页列表建议统一返回：

```json
{
  "list": [],
  "total": 0,
  "page": 1,
  "pageSize": 10
}
```

基础接口建议：

- `GET /api/health`：健康检查。
- `POST /api/auth/login`：后台或统一登录。
- `POST /api/frontend/auth/login`：前台登录，如后续独立前台登录体系。
- `GET /api/categories`：分类列表。
- `GET /api/banners`：轮播列表。
- `GET /api/stores`：门店列表。
- `GET /api/stores/:id`：门店详情。
- `GET /api/products`：商品列表。
- `GET /api/products/:id`：商品详情。
- `GET /api/search`：混合搜索。
- `POST /api/leads`：提交线索。
- `GET /api/favorites`：收藏列表。
- `POST /api/favorites`：新增收藏。
- `DELETE /api/favorites/:id`：取消收藏。

如后续 `PROJECT_REBUILD_CONTEXT.md` 给出了更精确的接口路径，应以该文档为准，并同步更新前端调用和部署配置。

## 7. 数据模型约定

核心表：

- `users`：用户、管理员、商户。
- `categories`：分类。
- `banners`：轮播图。
- `stores`：门店。
- `products`：商品。
- `leads`：意向表单/销售线索。
- `favorites`：用户收藏。

重要约束：

- 不要新增独立 `merchants` 表，除非明确完成迁移设计。当前商户身份通过 `users.role = 'merchant'` 表达。
- 门店名称字段使用 `stores.name`。
- 商品名称字段使用 `products.title`。
- 收藏建议添加唯一约束：`userId + type + targetId`。
- 商户和门店关系可能存在 `stores.merchantId` 与 `users.storeIds` 两种表达。重构时应统一模型，并同步权限判断。

## 8. 开发硬性规则

AI 修改项目时必须遵守：

- 保持根目录前端、`backend/` 后端的 monorepo 结构。
- 优先复用已有组件、API 封装、Vuex、路由和后端工具函数。
- 后端正式接口必须从 `backend/src/index.js` 和 `backend/src/routes/index.js` 进入。
- 生产环境不要使用 `server/preview-api.js`。
- 不要重新引入 `src/mock/*` 作为正式数据层。
- 不要把业务数据长期硬编码在 Vue 页面里。
- 不要改变 `/api` 接口挂载前缀，除非同步修改前端、文档和部署配置。
- 不要提交 `.env`、数据库密码、JWT 密钥、`node_modules/`、`dist/`、日志文件。
- 中文文档和源码统一使用 UTF-8。
- 如果文档和代码不一致，先检查 `PROJECT_REBUILD_CONTEXT.md`，再检查实际代码；修复时说明取舍。

## 9. 推荐开发顺序

从零重建或大改时，建议按以下顺序：

1. 确认数据模型和 `backend/database.sql`。
2. 确认统一响应、错误处理、日志、数据库连接。
3. 实现 JWT 登录和角色权限中间件。
4. 实现前台接口：首页、分类列表、门店详情、商品详情、搜索、基础数据。
5. 实现收藏和线索接口。
6. 实现后台 CRUD：分类、轮播、门店、商品、线索、用户。
7. 确认前端 `src/api/request.js` 和 `src/api/service.js`。
8. 实现前台页面和登录提醒。
9. 实现后台页面和角色菜单。
10. 做 PC、移动端 H5、WebView 适配检查。
11. 做本地启动、构建、上线验收。

小改动不需要机械执行完整顺序，但不能跳过受影响范围内的验证。

## 10. 验收清单

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
- [ ] 默认管理员密码上线前已修改。

部署：

- [ ] `npm run build` 成功。
- [ ] 生产 API 地址写入 `.env.production`。
- [ ] `zshjb.com` 备案已通过。
- [ ] `zshjb.com` 已正确解析到 `8.145.52.103`。
- [ ] PM2 能启动后端。
- [ ] Nginx 指向 `dist/` 并反代 `/api`。
- [ ] MySQL 未暴露公网。
- [ ] HTTPS 证书有效。

## 11. 首版不建议优先做的功能

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

## 12. 后续可扩展方向

首版稳定后可考虑：

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
4. 前台允许匿名浏览，收藏、我的等需要身份的操作统一走登录提醒。
5. 后台角色为 super_admin、merchant、user，商户数据隔离必须由后端强制实现。
6. 不要使用 server/preview-api.js 作为生产接口，不要重新引入 mock 作为正式数据层。
7. 修改接口、字段、路由、权限时，同步检查前端调用、后端实现和文档。
8. 完成后说明改动文件、验证方式、未验证风险。
```

## 14. 维护建议

- 新增接口时，同步更新 API 约定和验收清单。
- 新增环境变量时，同步更新 `.env.example`。
- 新增运行产物、缓存目录或上传目录时，同步更新 `.gitignore` 和 `.dockerignore`。
- 新增上线流程时，同步补充 `DEPLOYMENT_GUIDE.md`。
- 大规模重构前，先补充或更新 `PROJECT_REBUILD_CONTEXT.md`，避免业务口径散落在代码里。
