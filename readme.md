# 多 Hasura 项目启动说明

本项目支持多个 Hasura 实例共用一个 PostgreSQL 容器，每个实例独立数据库，端口互不冲突。

---

## 1. 创建 Docker 网络

所有服务需要在同一个 Docker 网络下才能互相访问。

```bash
docker network create hasura-network
```

> 只需创建一次，已存在会自动跳过。

---

## 2. 启动 PostgreSQL 容器

```bash
docker-compose -f postgres.docker-compose.yml up -d
```

---

## 3. 创建数据库

为每个 Hasura 项目创建独立数据库（如 db_8080、db_8081）：

```bash
docker exec -it hasura-postgres-1 psql -U postgres -c "CREATE DATABASE db_8080;"
docker exec -it hasura-postgres-1 psql -U postgres -c "CREATE DATABASE db_8081;"
```

如需更多项目，按需创建更多数据库。

---

## 4. 启动 Hasura 服务

分别启动每个 Hasura 实例（端口和数据库独立）：

```bash
docker-compose -f graphql-engine-8080.docker-compose.yml up -d
docker-compose -f graphql-engine-8081.docker-compose.yml up -d
```

如需更多实例，复制 compose 文件并修改端口和数据库名。

---

## 5. 访问控制台

- 项目1：http://localhost:8080/console
- 项目2：http://localhost:8081/console

---

## 6. 停止服务

分别停止各服务：

```bash
docker-compose -f graphql-engine-8080.docker-compose.yml down
docker-compose -f graphql-engine-8081.docker-compose.yml down
docker-compose -f postgres.docker-compose.yml down -v
```

---

## 7. 常见问题

- **端口被占用**：请先释放端口或修改 compose 文件端口。
- **数据库连接失败**：请确保所有服务都在 `hasura-network` 网络下，数据库 host 填写 `hasura-postgres-1`。
- **添加新项目**：复制一份 Hasura compose 文件，修改端口和数据库名，创建新数据库即可。

---

## 8. 重要环境变量说明

- **HASURA_GRAPHQL_ADMIN_SECRET**
  - 作用：Hasura 控制台和 API 的管理员密钥，访问 console 和所有 API 时都需要带上。
  - 配置示例：
    ```yaml
    HASURA_GRAPHQL_ADMIN_SECRET: admin_secret
    ```
  - 访问控制台时需在登录界面输入该密钥。

- **HASURA_GRAPHQL_JWT_SECRET**
  - 作用：配置 JWT 认证，保护 API 安全。
  - 配置示例：
    ```yaml
    HASURA_GRAPHQL_JWT_SECRET: '{"type":"HS256", "key":"your_jwt_secret_key"}'
    ```
  - 说明：如不需要 JWT，可不配置；如需自定义 JWT 规则，请参考 Hasura 官方文档。

- **HASURA_GRAPHQL_DATABASE_URL**
  - 作用：指定连接的 PostgreSQL 数据库。
  - 示例：
    ```yaml
    HASURA_GRAPHQL_DATABASE_URL: postgres://postgres:postgrespassword@hasura-postgres-1:5432/db_8080
    ```

- 其他常用环境变量：
  - `HASURA_GRAPHQL_ENABLE_CONSOLE`: 是否开启 web 控制台（一般设为 "true"）
  - `HASURA_GRAPHQL_DEV_MODE`: 开发模式（一般设为 "true"）
  - `HASURA_GRAPHQL_ENABLED_LOG_TYPES`: 日志类型
  - `HASURA_GRAPHQL_CORS_DOMAIN`: CORS 域名设置

---

## 9. 使用外部数据库工具连接 Postgres

你可以用 DBeaver、DataGrip、Navicat、psql 等工具直接连接 Docker 容器中的 Postgres 数据库。

### 连接信息
- **主机（Host）**：localhost
- **端口（Port）**：5432
- **用户名（User）**：postgres
- **密码（Password）**：postgrespassword
- **数据库名（Database）**：db_8080、db_8081 或 postgres（根据需要选择）

### 连接字符串示例
```
postgres://postgres:postgrespassword@localhost:5432/db_8080
```

### 工具填写示例
- **DBeaver/DataGrip/Navicat**：
  - Host: localhost
  - Port: 5432
  - User: postgres
  - Password: postgrespassword
  - Database: db_8080
- **psql 命令行**：
  ```bash
  psql -h localhost -p 5432 -U postgres -d db_8080
  ```
  然后输入密码 `postgrespassword`

### 注意事项
- Postgres 容器已将 5432 端口映射到本地，可直接用 localhost:5432 连接。
- 如果在服务器上，需开放 5432 端口或用 SSH 隧道转发。
- 如有自定义端口或密码，请用实际配置。

---

如有疑问请联系维护者。
