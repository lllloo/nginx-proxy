# nginx-proxy (nginx-proxy + acme companion)

本專案用於在單機 Docker 環境中運行 nginx-proxy 與 acme-companion，自動管理 TLS 憑證並反向代理多個虛擬主機。

**主要目的**

- 在單一 Docker Host 上快速架設反向代理（nginx）與自動憑證管理（Let's Encrypt / ACME）。

## 快速開始

1. 複製並編輯環境變數

```powershell
cp .env.example .env
# 編輯 .env，設定 DEFAULT_EMAIL 等必要參數
```

2. 啟動服務

```powershell
docker-compose up -d
```

3. 新增被代理的服務（範例）

- 啟動其他容器時，於該容器的環境變數中加入：
  - `VIRTUAL_HOST=your.domain.com`
  - `LETSENCRYPT_HOST=your.domain.com`
  - `VIRTUAL_PORT=3000`（視服務實際埠號而定）
- acme-companion（或類似的 ACME 工具）會自動為有設定的 host 申請並續期憑證。

## GitHub Actions 自動部署

- Workflow 位於 ` .github/workflows/deploy.yml`。
- 預設行為：當 push 到 `main` 分支時，透過 SSH 在目標主機上拉取或更新 repo，並執行 `docker-compose down && docker-compose up -d`。
- 請在 GitHub Secrets 中設定必要的變數，例如 `AWS_EC2_HOST`、`AWS_EC2_USERNAME`、`AWS_EC2_SSH_KEY`、`DISCORD_WEBHOOK`（視 workflow 內容而定）。

## 注意事項

- 憑證與 ACME 資料（`acme/`）以及 `.env` 等敏感檔案請勿提交至版本控制。請確認 `.gitignore` 已妥善設定。
- Docker Host 必須對外開放 `80` 與 `443`（或依驗證方式所需之埠），以利 ACME 的驗證。
- 若在雲端（例如 EC2）上使用自動部署，請確認 GitHub Action 使用的 SSH 金鑰與遠端用途權限安全妥善設定。

## 進階與自訂

- 自訂 vhost 檔可放置於 `vhost/`（若 compose 有 mount 至容器的對應路徑，例如 `/etc/nginx/vhost.d`）。
- 靜態檔放在 `html/`，會被 mount 至容器內的靜態根目錄（視 compose 設定）。
