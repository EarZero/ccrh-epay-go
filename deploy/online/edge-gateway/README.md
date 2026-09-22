# CCRH Edge Gateway

这是独立的 Caddy 统一网关部署包。

## 部署前提

- 服务器已安装 Docker 和 Docker Compose Plugin。
- 服务器的 `80/443` 端口可用。
- `epay.ccrh.glgtjt.com` 的 DNS A/AAAA 记录已指向服务器。
- 已创建共享 Docker 网络：

```bash
docker network create ccrh-edge
```

## 启动

```bash
cp .env.example .env
vi .env
docker compose up -d
docker compose ps
```

只有 Caddy 暴露宿主机的 `80/443` 端口。业务服务通过外部网络 `ccrh-edge` 被 Caddy 访问。

当前只启用 EPay：

```text
epay.ccrh.glgtjt.com → ccrh-epay-frontend / ccrh-epay-gateway
```

Router 和 Xiaogu 域名暂时只是 Caddyfile 中的注释占位，服务上线后再取消注释并填写正确的 Docker 服务名和端口。

## 数据目录

```text
./data/caddy
./data/caddy-config
```

## 常用命令

```bash
docker compose logs -f ccrh-edge-caddy
docker compose restart
docker compose down
```
