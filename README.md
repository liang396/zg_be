# zg_be（ZhiGuang Backend）

`zg_be` 是 **ZhiGuang（知光）** 的后端服务仓库，基于 Spring Boot 构建，提供认证与用户体系、内容发布（KnowPost）、用户关系、计数与搜索、对象存储以及 AI/RAG 能力等一组 REST API。

## 功能概览

- **认证与账号**：验证码发送/校验、注册、登录、Access/Refresh Token、刷新/登出、密码重置、登录审计日志
- **用户资料**：个人信息编辑、头像上传
- **KnowPost（图文内容）**：草稿创建、内容上传确认、元数据更新、发布、详情与 Feed
- **用户关系**：关注/取关、关系状态、关注/粉丝列表、关系计数
- **计数系统**：点赞/取消点赞、收藏/取消收藏、计数查询（事件驱动 + 缓存）
- **搜索与检索**：Elasticsearch 检索；向量检索用于 RAG
- **对象存储**：基于阿里云 OSS 的上传（含预签名/直传流程）
- **AI 能力**：基于 Spring AI 的内容摘要/描述生成与 RAG 问答（依赖向量库）

更多接口细节见 `docs/` 目录下的接口与设计文档。

## 技术栈

- Java 21 + Spring Boot 3.2.x
- Spring Security（OAuth2 Resource Server + JWT）
- MyBatis + MySQL
- Redis（含 Redisson）+ Caffeine（本地 L2 缓存）
- Kafka（异步事件、聚合消费）
- Elasticsearch（检索 + 向量库后端）
- Spring AI（模型接入 + VectorStore）
- 阿里云 OSS（对象存储）

## 快速开始（本地开发）

### 1) 环境要求

- JDK **21**
- Maven 3.9+
- MySQL 8.x、Redis、Kafka、Elasticsearch（按需启用对应功能）

### 2) 初始化数据库

在 MySQL 中创建数据库后执行：

```sql
-- 见 db/schema.sql
```

### 3) 配置 `application.yml`

仓库未内置本地配置文件（如 `application.yml`）。请在 `src/main/resources/application.yml` 中按需配置（示例仅供参考）：

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

# JWT / 验证码等认证配置
auth:
  jwt:
    issuer: zhiguang
    key-id: zhiguang-key
    private-key: classpath:keys/private.pem
    public-key: classpath:keys/public.pem

# 对象存储（阿里云 OSS）
oss:
  endpoint: oss-cn-xxx.aliyuncs.com
  access-key-id: ${OSS_ACCESS_KEY_ID:}
  access-key-secret: ${OSS_ACCESS_KEY_SECRET:}
  bucket: ${OSS_BUCKET:}
  public-domain: ${OSS_PUBLIC_DOMAIN:} # 可选：自定义域名/CDN

```

注意：`src/main/resources/keys/` 下包含开发用的 RSA Key。**生产环境务必替换并使用安全的密钥管理方式**，避免私钥随代码分发。

### 4) 运行

```bash
mvn -DskipTests spring-boot:run
```

## 文档

- 认证/账号：`docs/API接口文档.md`（概览：`docs/API接口.md`）
- KnowPost：`docs/API接口文档_knowpost.md`
- 用户关系：`docs/API接口文档_用户关系.md`
- 计数：`docs/API接口文档_计数.md`
- 设计文档：`docs/用户关系设计方案.md`、`docs/计数系统设计方案.md`

## 目录结构

- `src/main/java`：业务代码（Controller/Service/Config 等）
- `src/main/resources/mapper`：MyBatis Mapper XML
- `db/schema.sql`：MySQL 表结构
- `docs/`：接口与设计文档
