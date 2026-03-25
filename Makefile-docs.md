## 📍 Vị trí file:

File này là **`Makefile`** nằm ở **thư mục gốc của project**:

```
your-repo/
├── Makefile  ← File này
├── backend/
│   └── services/
│       └── inference/
│           ├── Dockerfile
│           ├── docker-compose.yml
│           ├── docker-compose.prod.yml
│           └── docker-compose.monitoring.yml
└── ...
```

---

## 🎯 Makefile là gì?

**Makefile** là file cấu hình giúp bạn **chạy các lệnh phức tạp bằng một lệnh đơn giản**. Thay vì nhập dài dòng, bạn chỉ cần `make dev` hoặc `make push`.

---

## 📖 Giải thích chi tiết từng phần:

### 1️⃣ **Phần khai báo biến (dòng 1-6)**

```makefile
SERVICE_DIR  := backend/services/inference
COMPOSE      := docker compose -f $(SERVICE_DIR)/docker-compose.yml
COMPOSE_PROD := docker compose -f $(SERVICE_DIR)/docker-compose.yml \
                               -f $(SERVICE_DIR)/docker-compose.prod.yml
COMPOSE_MON  := docker compose -f $(SERVICE_DIR)/docker-compose.yml \
                               -f $(SERVICE_DIR)/docker-compose.monitoring.yml
```

| Biến | Ý nghĩa |
|-----|---------|
| `SERVICE_DIR` | Đường dẫn đến thư mục inference service |
| `COMPOSE` | Lệnh docker compose cơ bản |
| `COMPOSE_PROD` | Lệnh docker compose + overlay production |
| `COMPOSE_MON` | Lệnh docker compose + overlay monitoring |

**Overlay** = kết hợp nhiều file compose lại

---

### 2️⃣ **Phần khai báo targets (dòng 8)**

```makefile
.PHONY: help dev stop logs shell test test-local lint format typecheck \
        build push prod-up prod-down monitor monitor-stop clean
```

`.PHONY` = khai báo các **target không phải file thực**. Nếu không có, Make sẽ tìm file tên `dev`, `stop`, etc.

---

### 3️⃣ **Phần Development (dòng 13-24)**

```makefile
dev: ## Start the inference service in development mode (hot reload)
	$(COMPOSE) up --build
```

**Lệnh:** `make dev`

**Thực thi:**
```bash
docker compose -f backend/services/inference/docker-compose.yml up --build
```

**Tác dụng:** Khởi động service với hot reload (tự reload khi code thay đổi)

---

```makefile
stop: ## Stop all containers
	$(COMPOSE) down
```

**Lệnh:** `make stop`

**Thực thi:**
```bash
docker compose -f backend/services/inference/docker-compose.yml down
```

**Tác dụng:** Dừng tất cả containers

---

```makefile
logs: ## Tail container logs
	$(COMPOSE) logs -f inference
```

**Lệnh:** `make logs`

**Thực thi:** Xem logs real-time của container `inference`

---

```makefile
shell: ## Open a shell inside the running container
	$(COMPOSE) exec inference bash
```

**Lệnh:** `make shell`

**Thực thi:** Vào shell của container đang chạy

---

### 4️⃣ **Phần Testing & Quality (dòng 27-40)**

```makefile
test: ## Run tests inside Docker (no local GPU/AWS needed)
	$(COMPOSE) run --rm inference \
		pytest --tb=short -v
```

**Lệnh:** `make test`

**Tác dụng:** Chạy tests bên trong Docker (không cần GPU local)

---

```makefile
test-local: ## Run tests locally (requires venv with dev deps)
	cd $(SERVICE_DIR) && pytest --tb=short -v
```

**Lệnh:** `make test-local`

**Tác dụng:** Chạy tests trên máy local (cần Python venv)

---

```makefile
lint: ## Run ruff lint + format check
	cd $(SERVICE_DIR) && ruff check app/ tests/ && ruff format --check app/ tests/
```

**Lệnh:** `make lint`

**Tác dụng:** Kiểm tra code style (không sửa)

---

```makefile
format: ## Auto-fix formatting with ruff
	cd $(SERVICE_DIR) && ruff format app/ tests/ && ruff check --fix app/ tests/
```

**Lệnh:** `make format`

**Tác dụng:** Tự động sửa code style

---

```makefile
typecheck: ## Run mypy type checker
	cd $(SERVICE_DIR) && mypy app/
```

**Lệnh:** `make typecheck`

**Tác dụng:** Kiểm tra type hints

---

### 5️⃣ **Phần Docker (dòng 43-56)**

```makefile
build: ## Build the production Docker image
	docker build \
		-t evelynn-inference:latest \
		-f $(SERVICE_DIR)/Dockerfile \
		$(SERVICE_DIR)
```

**Lệnh:** `make build`

**Thực thi:**
```bash
docker build -t evelynn-inference:latest \
  -f backend/services/inference/Dockerfile \
  backend/services/inference
```

**Tác dụng:** Build Docker image với tag `evelynn-inference:latest`

---

```makefile
push: build ## Build and push to GHCR (requires IMAGE_TAG env var)
	docker tag evelynn-inference:latest \
		ghcr.io/$(shell git config --get remote.origin.url | sed 's/.*github.com[:/]//' | sed 's/\.git//')/evelynn-inference:$(IMAGE_TAG)
	docker push \
		ghcr.io/$(shell git config --get remote.origin.url | sed 's/.*github.com[:/]//' | sed 's/\.git//')/evelynn-inference:$(IMAGE_TAG)
```

**Lệnh:** `make push IMAGE_TAG=v1.0.0`

**Thực thi:**
1. Build image (gọi `build` target)
2. Tag lại với GHCR registry
3. Push lên GHCR

**Ví dụ:**
```bash
# Lấy repo từ git
# your-username/your-repo

# Kết quả:
# ghcr.io/your-username/your-repo/evelynn-inference:v1.0.0
```

---

### 6️⃣ **Phần Production (dòng 59-66)**

```makefile
prod-up: ## Start with production compose overlay
	$(COMPOSE_PROD) up -d
```

**Lệnh:** `make prod-up`

**Thực thi:** Khởi động với cấu hình production (overlay)

---

```makefile
prod-down: ## Stop production stack
	$(COMPOSE_PROD) down
```

**Lệnh:** `make prod-down`

**Thực thi:** Dừng production stack

---

### 7️⃣ **Phần Monitoring (dòng 69-82)**

```makefile
monitor: ## Start app + Prometheus + Grafana
	$(COMPOSE_MON) up --build
	@echo ""
	@echo "  Prometheus : http://localhost:9090"
	@echo "  Grafana    : http://localhost:3001  (admin / admin)"
	@echo "  App        : http://localhost:8000/docs"
	@echo ""
```

**Lệnh:** `make monitor`

**Thực thi:**
- Khởi động app + Prometheus + Grafana
- In ra các URL để truy cập

---

### 8️⃣ **Phần Cleanup (dòng 85-88)**

```makefile
clean: ## Remove containers, volumes, and dangling images
	$(COMPOSE) down -v --remove-orphans
	docker image prune -f
```

**Lệnh:** `make clean`

**Thực thi:**
- Xóa containers + volumes
- Xóa dangling images

---

## 📋 Bảng tóm tắt các lệnh:

| Lệnh | Ý nghĩa |
|-----|---------|
| `make help` | Hiển thị tất cả lệnh |
| `make dev` | Chạy development (hot reload) |
| `make stop` | Dừng containers |
| `make logs` | Xem logs real-time |
| `make shell` | Vào shell container |
| `make test` | Chạy tests trong Docker |
| `make test-local` | Chạy tests local |
| `make lint` | Kiểm tra code style |
| `make format` | Tự động sửa code style |
| `make typecheck` | Kiểm tra type hints |
| `make build` | Build Docker image |
| `make push IMAGE_TAG=v1.0.0` | Build + push lên GHCR |
| `make prod-up` | Khởi động production |
| `make prod-down` | Dừng production |
| `make monitor` | Chạy với Prometheus + Grafana |
| `make monitor-stop` | Dừng monitoring |
| `make clean` | Xóa containers + volumes |

---

## 💡 Cách sử dụng:

```bash
# Xem tất cả lệnh
make help

# Chạy development
make dev

# Trong terminal khác, xem logs
make logs

# Chạy tests
make test

# Push lên GHCR
make push IMAGE_TAG=v1.0.0

# Dừng tất cả
make stop

# Dọn dẹp
make clean
```