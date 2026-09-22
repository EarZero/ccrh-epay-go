# CCRH EPay Gateway 线上部署包

本包采用“独立 Caddy 统一网关 + EPay 独立业务 Compose”的部署方式。

- Caddy 是唯一对外暴露宿主机端口的服务，使用 `80/443`。
- EPay 前端和后端同时加入共享网络 `ccrh-edge` 与 EPay 私有网络。
- PostgreSQL 和 Redis 只加入 EPay 私有网络，不暴露宿主机端口。
- 本包使用已构建的 `linux/amd64` 镜像，不包含项目源码。

## 目录结构

```text
edge-gateway/
├── Caddyfile
├── .env.example
├── docker-compose.yml
└── data/

epay/
├── .env.example
├── docker-compose.yml
└── data/
```

## 首次部署

### 1. 创建共享网关网络

只需要在服务器上执行一次：

```bash
docker network create ccrh-edge
```

如果网络已经存在，提示已存在即可，不要重复创建。

### 2. 启动 EPay 业务服务

```bash
cd epay
cp .env.example .env
vi .env

docker login ai-city-cn-shanghai.cr.volces.com
docker compose pull
docker compose up -d
docker compose ps
```

至少修改以下配置：

- `BACKEND_IMAGE`、`FRONTEND_IMAGE`：需要部署其他镜像版本时修改。
- `DB_PASSWORD`：PostgreSQL 密码。
- `REDIS_PASSWORD`：Redis 密码。
- `JWT_SECRET`：生产环境随机密钥。
- `DEFAULT_ADMIN_PASSWORD`：首次初始化管理员密码。
- `MERCHANT_REGISTRATION_ENABLED`：生产环境保持 `false`；商户由管理端「商户管理 → 新增商户」创建。

公开注册关闭后，管理员登录 `/admin/login`，进入「商户管理」点击「新增商户」即可创建商户。新增商户默认立即启用，管理员负责安全交付初始密码。

### 3. 配置并启动 Caddy

先将 `edge-gateway/.env.example` 复制为 `.env`，确认域名配置：

```bash
cd ../edge-gateway
cp .env.example .env
vi .env

docker compose up -d
docker compose ps
```

启动前需要确保：

- `epay.ccrh.glgtjt.com` 已解析到本服务器。
- 服务器防火墙允许入站 `80`、`443`。
- `80`、`443` 没有被其他宿主机服务占用。

Caddy 会根据 `epay.ccrh.glgtjt.com` 自动申请和续期 HTTPS 证书。

预留域名如下，当前不会转发到任何服务：

```text
router.ccrh.glgtjt.com
xiaogu.ccrh.glgtjt.com
```

## 后续增加服务

新服务的 Compose 文件加入已有的外部网络：

```yaml
networks:
  ccrh-edge:
    external: true
    name: ccrh-edge
```

需要被公网访问的前端或后端服务加入 `ccrh-edge`，数据库和 Redis 只加入该服务自己的私有网络，不配置 `ports`。

然后在 `edge-gateway/Caddyfile` 中取消对应预留域名的注释，并填写实际服务名和端口，例如：

```caddyfile
router.ccrh.glgtjt.com {
    reverse_proxy ccrh-router-gateway:8080
}
```

修改 Caddy 配置后执行：

```bash
docker compose up -d
```

## 数据目录

数据均挂载在当前部署目录下：

```text
epay/data/postgres
epay/data/redis
edge-gateway/data/caddy
edge-gateway/data/caddy-config
```

请定期备份这些目录和 PostgreSQL 数据。不要随意执行 `docker compose down -v`。

## 常用命令

```bash
docker compose -f epay/docker-compose.yml logs -f ccrh-epay-gateway
docker compose -f epay/docker-compose.yml ps
docker compose -f edge-gateway/docker-compose.yml logs -f ccrh-edge-caddy
docker compose -f edge-gateway/docker-compose.yml ps
docker network inspect ccrh-edge
```
