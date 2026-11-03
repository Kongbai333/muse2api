# muse2api 镜像仓（GitHub Actions 自动构建）

本仓库是 [`czg86389-hub/muse2api`](https://github.com/czg86389-hub/muse2api) 在 `v1.6.6` 上的镜像副本，**唯一改动**：

- 新增 `.github/workflows/docker-publish.yml`：GitHub 上自动构建镜像并推送到 GHCR，打 tag 时同时产出离线 `tar.gz`（发到 Release）。
- `Dockerfile` 的 apt 那行加了 `Acquire::Retries=8` 与 `Acquire::http::Timeout=60`，只提高网络重试韧性，安装的包与版本完全一致（上游直连环境可自行还原）。

## 用法

```bash
# 拉镜像（仓库 public 时，若包仍是 private，需要 docker login ghcr.io 后拉）
docker pull ghcr.io/kongbai333/muse2api:latest
docker pull ghcr.io/kongbai333/muse2api:v1.6.6

# 不想用 registry：到 Releases 下载 muse2api-vX.Y.Z-linux-amd64.tar.gz
docker load -i muse2api-v1.6.6-linux-amd64.tar.gz
```

## 部署

```yaml
services:
  muse2api:
    image: ghcr.io/kongbai333/muse2api:v1.6.6
    container_name: muse2api
    restart: always
    ports: ["18610:18610"]
    environment:
      - MUSE2API_KEY=${MUSE2API_KEY:?}
      - MUSE2API_HOST=0.0.0.0
      - MUSE2API_PORT=18610
    volumes:
      - ./data:/app/data
    shm_size: '2gb'      # Chromium 必需
```

反代记得 `proxy_read_timeout 600s` 与 `proxy_buffering off`（生图/视频慢、对话是 SSE 流）。
账号数据不入库，用浏览器扩展或油猴脚本从 muse.ai 登录态导入到 `./data`。

## 手动触发

仓库 → Actions → `build-and-publish-image` → Run workflow。
