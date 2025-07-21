# CADASU Docker Compose

這是 CADASU 專案的 Docker Compose 配置，用於統一管理所有服務。

> **注意**: 此專案使用 Docker Compose V2 的最新慣例，檔案命名為 `compose.yaml` 而非傳統的 `docker-compose.yml`。

## 專案結構

此 repo 與其他服務 repo 平行存在：

```
cadasu/
├── docker-compose/        # 本 repo
├── OSLO-backend/          # Django 後端
├── OSLO-frontend/         # React 前端
└── image-processing-server/  # 圖片處理服務
```

## 設置步驟

### 1. 配置環境變數

Docker Compose 的環境變數管理採用分層設計：
- `.env` - Docker Compose 基礎設定（路徑、端口等）
- `backend.env` - Backend 服務專用環境變數
- `frontend.env` - Frontend 服務專用環境變數
- `image-processor.env` - Image Processor 服務專用環境變數

複製所有範例檔案：

```bash
# 複製 Docker Compose 基礎設定
cp .env.example .env

# 複製各服務的環境變數
cp backend.env.example backend.env
cp frontend.env.example frontend.env
cp image-processor.env.example image-processor.env
```

### 2. 編輯環境變數檔案

#### 基礎設定 (.env)
主要包含路徑和端口設定，通常不需要修改：

```bash
# 如果使用預設的平行結構，保持這些相對路徑即可
BACKEND_PATH=../OSLO-backend
FRONTEND_PATH=../OSLO-frontend
IMAGE_PROCESSOR_PATH=../image-processing-server
```

#### Backend 設定 (backend.env)
編輯 `backend.env`，設定必要的環境變數：
- `SECRET_KEY` - Django 密鑰
- `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` - AWS 憑證
- 其他服務相關設定

#### Frontend 設定 (frontend.env)
Frontend 設定相對簡單，主要是 API 連接設定。

#### Image Processor 設定 (image-processor.env)
設定圖片處理相關的環境變數。

### 3. 啟動服務

```bash
# 在 docker-compose 目錄下
# 第一次啟動時構建映像
docker compose up --build -d

# 之後的啟動
docker compose up -d

# 如果需要重新構建（例如依賴變更後）
docker compose build
```

## 常用指令

```bash
# 查看服務狀態
docker compose ps

# 查看日誌
docker compose logs -f [service-name]

# 重啟特定服務
docker compose restart backend

# 停止所有服務
docker compose down

# 停止並清除資料
docker compose down -v

# 進入容器
docker compose exec backend bash
```

## 服務說明

- **backend**: Django API 服務 (port 8000)
- **celery-broadcast**: Celery worker for broadcast queue
- **celery-io**: Celery worker for IO queue
- **frontend**: React 前端應用 (port 3000)
- **image-processor**: 圖片處理服務
- **postgres**: PostgreSQL 資料庫 with pgvector (port 5432)
- **redis**: Redis 快取服務 (port 6379)

## 環境變數管理

### 優先順序
1. Docker Compose 在 `docker-compose/` 目錄下的環境變數檔案優先
2. 各專案內的 `.env` 檔案不會被使用（除非您修改 compose.yaml）
3. 服務間的連接使用容器名稱（如 `postgres`、`redis`）

### 檔案結構
```
docker-compose/
├── .env                    # Docker Compose 基礎設定
├── backend.env            # Backend 服務環境變數
├── frontend.env           # Frontend 服務環境變數
├── image-processor.env    # Image Processor 服務環境變數
└── compose.yaml           # Docker Compose 配置
```

## 疑難排解

### 權限問題
如果遇到檔案權限問題，確保 Docker 有權限訪問本地檔案：
- macOS: 在 Docker Desktop 設定中添加檔案共享路徑
- Linux: 確保用戶在 docker 群組中

### 網路連接
服務間通訊使用容器名稱，例如：
- Django 連接 PostgreSQL: `postgres:5432`
- Django 連接 Redis: `redis:6379`

### 效能優化
在 macOS 上掛載大量檔案可能影響效能，可以考慮：
1. 只掛載必要的目錄
2. 使用 Docker volumes 替代本地掛載
3. 排除 node_modules 和 venv 目錄

## 進階配置

### 生產環境
如需部署到生產環境，建議：
1. 使用專用的 Dockerfile 構建映像
2. 移除本地檔案掛載
3. 使用環境特定的配置檔案
4. 添加健康檢查和重啟策略

### 擴展服務
要添加新服務，在 `compose.yaml` 中添加新的服務定義，並在 `.env.example` 中添加相應的路徑配置。