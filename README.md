# 知光平台后端 `zg_be`

一个面向内容社区场景的后端服务项目，基于 **Spring Boot 3 + Java 21** 构建，覆盖了认证体系、内容发布、用户关系、计数系统、搜索检索、对象存储以及 AI/RAG 能力。

这个项目更偏向“可用于面试展示的完整后端工程”而不只是接口集合：既有典型业务模块，也包含缓存、异步事件、搜索、向量检索、最终一致性等中高级后端设计点。

## 项目定位

我希望通过这个项目展示的，不只是“会写 CRUD”，而是：

- 能独立搭建一个结构清晰的 Spring Boot 后端
- 能围绕真实业务拆分模块并设计接口
- 能处理高频读写场景下的缓存、一致性和异步化问题
- 能把 Elasticsearch、Kafka、Redis、OSS、Spring AI 等组件整合到业务中
- 能产出配套的接口文档和专题设计文档，而不是只停留在代码层

## 核心功能

- **认证与账号体系**
  - 验证码发送与校验
  - 注册、登录、登出
  - Access Token / Refresh Token
  - 密码重置
  - 登录审计日志
- **用户资料**
  - 个人信息维护
  - 头像上传
- **内容系统 KnowPost**
  - 草稿创建
  - 内容上传确认
  - 元数据更新
  - 内容发布、详情查询、Feed 流
- **用户关系**
  - 关注 / 取消关注
  - 关系状态查询
  - 关注列表 / 粉丝列表
  - 关系计数维护
- **计数系统**
  - 点赞 / 取消点赞
  - 收藏 / 取消收藏
  - 多维度计数查询
- **搜索与检索**
  - Elasticsearch 关键词搜索
  - 搜索建议
  - 面向 RAG 的向量检索
- **对象存储**
  - 基于阿里云 OSS 的上传能力
  - 预签名上传流程
- **AI 能力**
  - 内容描述生成
  - 内容摘要
  - 基于向量库的问答能力

## 技术栈

- **基础框架**：Java 21、Spring Boot 3.2.4
- **安全认证**：Spring Security、OAuth2 Resource Server、JWT
- **数据存储**：MySQL、Redis、Elasticsearch
- **ORM / 持久层**：MyBatis
- **消息队列**：Kafka
- **缓存方案**：Redis + Caffeine + Redisson
- **对象存储**：Alibaba Cloud OSS
- **AI 集成**：Spring AI、OpenAI / DeepSeek、Elasticsearch Vector Store
- **其他工程能力**：Actuator、参数校验、全局异常处理、线程池配置

## 面试可重点展开的亮点

### 1. 认证体系不是“单接口登录”

项目实现了完整的认证链路，包括验证码校验、JWT 签发、Refresh Token 刷新、登出失效、密码重置和登录审计。这一部分适合重点讲：

- 为什么后端 API 采用无状态 JWT
- Refresh Token 如何借助 Redis 管理
- 为什么把登录日志单独沉淀为审计能力

### 2. 用户关系与计数采用异步化设计

关注、点赞、收藏这类动作不是简单地“写一张表然后 count(*)”，而是引入了：

- Redis 位图 / 聚合计数
- Kafka 异步事件
- 最终一致性的计数落盘思路
- 异常情况下的重建与补偿机制

这部分很适合体现对高并发读写和性能优化的理解。

### 3. 搜索与内容系统不是割裂的

内容发布之后，可以进入搜索链路；同时也为 AI/RAG 场景预留了向量检索基础设施。相比只做普通搜索，这一部分能体现：

- 业务数据如何进入检索系统
- 关键词搜索与向量搜索的差异
- AI 能力如何嵌入到内容产品中

### 4. 工程化意识比较完整

仓库中除了业务代码，还包含：

- 数据库初始化脚本
- 模块级接口文档
- 用户关系与计数系统设计文档
- 统一异常处理与配置化能力

这意味着项目并非“只跑起来”，而是考虑了可维护性和可沟通性。

## 项目结构

```text
zg_be
├─ src/main/java/com/tongji
│  ├─ auth        # 认证与令牌体系
│  ├─ profile     # 用户资料
│  ├─ knowpost    # 内容发布与 Feed
│  ├─ relation    # 用户关系
│  ├─ counter     # 计数系统
│  ├─ search      # 搜索服务
│  ├─ storage     # OSS 存储
│  ├─ llm         # AI / RAG 能力
│  ├─ cache       # 缓存相关配置与热点处理
│  └─ common      # 通用异常、工具与 Web 层封装
├─ src/main/resources/mapper   # MyBatis Mapper XML
├─ db/schema.sql               # 数据库初始化脚本
└─ docs                        # 接口文档与专题设计文档
```

## 重点模块说明

### 认证模块 `auth`

- 提供注册、登录、验证码、刷新令牌、退出登录等能力
- 使用 JWT 进行接口鉴权
- 使用 Redis 管理 Refresh Token 和验证码

### 内容模块 `knowpost`

- 支持图文内容草稿、编辑、发布、详情和 Feed
- 对接搜索与 AI 能力，形成“内容生产 + 内容消费”闭环

### 用户关系模块 `relation`

- 支持关注、取关、关系状态查询、关注/粉丝列表
- 配套设计文档说明了主从表、异步同步和幂等处理思路

### 计数模块 `counter`

- 把点赞/收藏等行为抽象为统一计数能力
- 结合 Redis、Kafka 做异步聚合与最终一致性处理
- 是整个项目里最适合面试深入展开的技术模块之一

### 搜索与 AI 模块 `search` / `llm`

- Elasticsearch 提供关键词搜索和建议能力
- Spring AI + Vector Store 支撑内容增强与 RAG 场景

## 文档索引

- [接口总览](./docs/API接口.md)
- [认证与账号接口文档](./docs/API接口文档.md)
- [KnowPost 接口文档](./docs/API接口文档_knowpost.md)
- [用户关系接口文档](./docs/API接口文档_用户关系.md)
- [计数系统接口文档](./docs/API接口文档_计数.md)
- [用户关系设计方案](./docs/用户关系设计方案.md)
- [计数系统设计方案](./docs/计数系统设计方案.md)

## 本地运行

### 1. 环境要求

- JDK 21
- Maven 3.9+
- MySQL 8.x
- Redis
- Kafka
- Elasticsearch

其中 Kafka、Elasticsearch、AI 相关配置按需启用；如果只调试基础账号与资料能力，可以先最小化配置运行。

### 2. 初始化数据库

先创建数据库，再执行：

```sql
-- 见 db/schema.sql
```

### 3. 配置 `application.yml`

仓库默认未提交完整本地配置，请在 `src/main/resources/application.yml` 中按需补充。下面给出一个简化示例：

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/zhiguang?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: root
  data:
    redis:
      host: localhost
      port: 6379
  kafka:
    bootstrap-servers: localhost:9092
  elasticsearch:
    uris:
      - http://localhost:9200
  ai:
    vectorstore:
      elasticsearch:
        index-name: zhiguang-ai-index

auth:
  jwt:
    issuer: zhiguang
    key-id: zhiguang-key
    private-key: classpath:keys/private.pem
    public-key: classpath:keys/public.pem

oss:
  endpoint: oss-cn-xxx.aliyuncs.com
  access-key-id: ${OSS_ACCESS_KEY_ID:}
  access-key-secret: ${OSS_ACCESS_KEY_SECRET:}
  bucket: ${OSS_BUCKET:}
  public-domain: ${OSS_PUBLIC_DOMAIN:}
```

注意事项：

- `src/main/resources/keys/` 下包含开发用密钥
- 正式环境应替换为安全的密钥管理方案
- OSS、AI 模型、向量库等配置建议全部走环境变量

### 4. 启动项目

```bash
mvn -DskipTests spring-boot:run
```

## 如果面试官只看 1 分钟，我希望他看到什么

- 这是一个完整的内容社区后端，不是单点功能 demo
- 技术栈覆盖认证、缓存、消息队列、搜索、对象存储、AI/RAG
- 不只写了接口，还写了关系系统和计数系统的设计文档
- 项目里能看到对性能、扩展性和工程化的思考

## 后续仍可继续加强的方向

- 增加 Docker Compose，一键拉起 MySQL / Redis / Kafka / Elasticsearch
- 增加测试覆盖率，特别是认证、关系、计数模块
- 补充系统架构图、时序图和压测结果
- 增加接口示例请求与响应，进一步提升可读性

---

如果这份 README 是专门给面试官看的，我的优化思路是：先让对方快速知道“这项目做了什么”，再让对方看到“这里面有哪些值得追问的技术点”。这样比单纯罗列接口更容易建立技术印象。
