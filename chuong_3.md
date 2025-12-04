# Chương 3: Docker Image — Quản lý, Tối ưu và Kỹ thuật Nâng cao

Chương này đi sâu vào **Image** — thành phần cốt lõi trong Docker. Bạn sẽ hiểu rõ image là gì, cấu trúc layer, cách build, tối ưu kích thước, cache, đa stage build, gắn thẻ (tag), đẩy (push) lên registry, bảo mật image và tích hợp vào CI.

---

## 1. Image là gì? Cấu trúc cơ bản

* **Image** là một tập hợp các layer read-only xếp chồng lên nhau. Mỗi lệnh trong Dockerfile (RUN, COPY, ADD,...) tạo ra một layer mới.
* Khi chạy `docker run`, Docker tạo một **container** bằng cách gắn một layer read-write lên cùng các layer image.
* Layer giúp tái sử dụng: nhiều image có thể chia sẻ cùng layer, tiết kiệm dung lượng.

---

## 2. Một số lệnh cơ bản làm việc với image

```bash
# Tải image từ registry
docker pull nginx:latest

# Xem danh sách image
docker images --format "{{.Repository}}:{{.Tag}} {{.ID}} {{.Size}}"

# Xóa image
docker rmi <image-id-or-name>

# Xem lịch sử layer của image
docker history <image>

# Inspect image metadata
docker image inspect <image>
```

---

## 3. Dockerfile: Các lệnh cơ bản và ý nghĩa

* `FROM`: image gốc (base)
* `LABEL`: metadata cho image (maintainer, version,...)
* `ARG`: biến build-time
* `ENV`: biến runtime (được lưu vào image)
* `WORKDIR`: đặt thư mục làm việc
* `COPY` / `ADD`: sao chép file vào image (`ADD` có thêm tính năng giải nén URL)
* `RUN`: thực hiện các lệnh để cài đặt phần mềm
* `EXPOSE`: khai báo port (chỉ mang tính tài liệu)
* `VOLUME`: khai báo điểm mount
* `USER`: chuyển user
* `ENTRYPOINT` / `CMD`: chỉ định tiến trình chính khi container khởi động

Ví dụ Dockerfile đơn giản:

```Dockerfile
FROM node:18-alpine
LABEL maintainer="anbinh@example.com"
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

---

## 4. Build image — các tuỳ chọn quan trọng

```bash
# Build image
docker build -t myapp:1.0 .

# Sử dụng build-arg
docker build --build-arg NODE_ENV=production -t myapp:prod .

# Sử dụng BuildKit để tăng tốc và cache tốt hơn
DOCKER_BUILDKIT=1 docker build -t myapp:bk .

# Chỉ định file Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .
```

---

## 5. .dockerignore — tương tự .gitignore

* Nên dùng `.dockerignore` để loại trừ node_modules, file .git, logs,... tránh copy thừa vào image.

Ví dụ `.dockerignore`:

```
node_modules
.git
Dockerfile.dev
*.log
.env
```

---

## 6. Layer caching và cách tận dụng

* Docker cache dùng layer cũ nếu lệnh Dockerfile không thay đổi.
* Sắp xếp Dockerfile hợp lý: các lệnh thay đổi ít (ví dụ `RUN apt-get update && apt-get install ...`) nên đặt trước; các file nguồn thay đổi nhiều (COPY . .) nên đặt sau.

Ví dụ tối ưu cho NodeJS:

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
CMD ["node","index.js"]
```

Ở trên, nếu chỉ thay code mà không thay package.json thì bước `npm ci` được cache.

---

## 7. Multi-stage build (Multi-stage Dockerfile)

* Giảm kích thước image bằng cách tách giai đoạn build và runtime.
* Dùng phổ biến trong build ứng dụng compiled (Go, Java) hoặc front-end build.

Ví dụ Node build + runtime:

```Dockerfile
# Stage 1: build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: runtime
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --production
CMD ["node","dist/server.js"]
```

---

## 8. Giảm kích thước image — mẹo thực tế

* Dùng base image nhẹ (alpine, scratch nếu có thể)
* Dùng multi-stage build
* Loại bỏ file tạm trong cùng một `RUN` (sử dụng `&&` và `rm -rf /var/lib/apt/lists/*`)
* Dùng `--no-install-recommends` với apt
* Dọn cache package manager
* Sử dụng `.dockerignore`
* Xóa source files không cần thiết

Ví dụ RUN tối ưu (Debian/Ubuntu):

```Dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
  && rm -rf /var/lib/apt/lists/*
```

---

## 9. Tagging & Naming image (best practice)

* Cấu trúc tên: `<registry>/<username-or-org>/<repo>:<tag>`
* Tag thường dùng: `latest`, semantic version (`1.2.3`), `sha-commit`, `ci-<build-number>`

Ví dụ:

```
myregistry.example.com/anbinh/myapp:1.0.0
anbinh/myapp:latest
anbinh/myapp:sha-abcdef123
```

---

## 10. Push / Pull image lên Registry

```bash
# Đăng nhập Docker Hub
docker login

# Push image
docker push anbinh/myapp:1.0

# Pull image
docker pull anbinh/myapp:1.0
```

### 10.1. Private Registry (self-hosted)

* Có thể dùng Docker Registry image hoặc Nexus/Harbor.
* Cấu hình HTTPS & authentication.

Ví dụ chạy registry local:

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
# Push tới local
docker tag myapp localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0
```

---

## 11. Quản lý image cũ & dọn dẹp

* Xem image không dùng: `docker image ls -a`
* Xóa dangling images (untagged): `docker image prune`
* Xóa unused images, containers, volumes: `docker system prune -a --volumes` (cẩn thận)

---

## 12. Bảo mật image

### 12.1. Quét lỗ hổng (vulnerability scanning)

* Dùng công cụ: `trivy`, `clair`, `anchore`.

Ví dụ dùng trivy:

```bash
trivy image anbinh/myapp:1.0
```

### 12.2. Sử dụng non-root user

* Trong Dockerfile, set `USER` thấp privilege.

### 12.3. Minimize surface attack

* Dùng minimal base image
* Loại bỏ công cụ build ra image runtime

### 12.4. Image signing & provenance

* Dùng Docker Content Trust (Notary) hoặc cosign để ký image.

---

## 13. Metadata: Labels, SBOM và reproducibility

* `LABEL` giúp lưu metadata: maintainer, version, description, vcs-ref.
* Tạo SBOM (Software Bill of Materials) để audit thành phần image.
* Dùng `--label` khi build hoặc `LABEL` trong Dockerfile.

Ví dụ:

```Dockerfile
LABEL org.opencontainers.image.revision="${VCS_REF}" \
      org.opencontainers.image.source="https://github.com/your/repo"
```

---

## 14. BuildKit và cache dist/remote cache

* BuildKit cải thiện tốc độ build, parallelisme, cache sharing.
* Kích hoạt BuildKit: `DOCKER_BUILDKIT=1 docker build ...`
* Sử dụng cache export/import để chia cache giữa CI job:

```bash
# Export cache
DOCKER_BUILDKIT=1 docker build --cache-to=type=local,dest=.buildcache -t myapp:ci .
# Import cache
DOCKER_BUILDKIT=1 docker build --cache-from=type=local,src=.buildcache -t myapp:ci .
```

---

## 15. Inspect, history và debug image

* `docker image inspect <image>`: metadata JSON
* `docker history <image>`: hiển thị size & layer per instruction
* Chạy container tạm để debug:

```bash
docker run -it --rm --entrypoint sh myapp:latest
```

---

## 16. Tích hợp image build vào CI/CD (ví dụ GitHub Actions)

* Steps chính: checkout → setup docker/login → build → scan → push

Sample GitHub Actions job:

```yaml
jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Login to registry
        uses: docker/login-action@v2
        with:
          registry: docker.io
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: anbinh/myapp:${{ github.sha }}
```

---

## 17. Những lỗi thường gặp liên quan đến image

* Build fails do missing files → kiểm tra `.dockerignore` và đường dẫn COPY
* Quá lớn → dùng multi-stage và alpine
* Cache không được dùng → kiểm tra thứ tự lệnh
* Lỗi permission file → dùng `chown` hoặc `USER` phù hợp

---

## 18. Thực hành đề xuất

1. Viết Dockerfile cho một ứng dụng NodeJS nhỏ, build image, chạy container.
2. Thử multi-stage build với dự án có bước build (React / Vue / Go).
3. Sử dụng `trivy` để quét image.
4. Thiết lập registry local và push/pull image.

---

## 19. Tài liệu & công cụ tham khảo

* Docker official docs: [https://docs.docker.com](https://docs.docker.com)
* Trivy: [https://github.com/aquasecurity/trivy](https://github.com/aquasecurity/trivy)
* BuildKit docs
* Docker best practices guide

---
