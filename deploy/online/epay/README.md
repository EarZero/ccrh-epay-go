# CCRH EPay

这是 EPay 业务服务部署包，使用已经构建好的 `linux/amd64` 镜像，不需要在服务器编译源码。

## 部署前提

先创建 Caddy 使用的共享网络：

```bash
docker network create ccrh-edge
```

如果网络已经存在，提示已存在即可。

## 配置并启动

```bash
cp .env.example .env
vi .env

docker login ai-city-cn-shanghai.cr.volces.com
docker compose pull
docker compose up -d
docker compose ps
```

必须修改 `.env` 中的生产环境配置：

- `DB_PASSWORD`
- `REDIS_PASSWORD`
- `JWT_SECRET`
- `DEFAULT_ADMIN_PASSWORD`
- `MERCHANT_REGISTRATION_ENABLED`（生产环境保持 `false`）

公开注册默认关闭。管理员登录管理后台后，进入「商户管理」点击「新增商户」创建账号；新商户默认立即启用。后端仍会对公开注册接口返回 `403`，即使绕过前端也不能注册。

只有前端和后端加入共享网络 `ccrh-edge`，PostgreSQL 和 Redis 只加入 EPay 私有网络，不暴露宿主机端口。

## 数据目录

```text
./data/postgres
./data/redis
```

## 常用命令

```bash
docker compose logs -f ccrh-epay-gateway
docker compose ps
docker compose restart
docker compose down
```
