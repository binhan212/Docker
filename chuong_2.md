# Chương 2: Cài đặt Docker và Cấu hình Môi trường

Chương này hướng dẫn chi tiết cách cài đặt Docker trên **Windows**, **macOS**, **Linux**, đồng thời cấu hình môi trường để Docker hoạt động ổn định. Đây là chương quan trọng trước khi bước vào thực hành.

---

# 1. Các thành phần Docker cần cài

Docker có 2 nhóm phần mềm:

## 1.1. Docker Desktop (Windows & macOS)

Bao gồm:

* Docker Engine
* Docker CLI
* Docker Compose
* Dashboard GUI
* Kubernetes tùy chọn

## 1.2. Docker Engine (Linux)

Linux không sử dụng Docker Desktop (trừ bản thương mại), mà cài trực tiếp Docker Engine.

---

# 2. Cài đặt Docker trên Windows

Docker yêu cầu:

* Windows 10/11 64-bit
* Bật WSL2
* RAM khuyến nghị: 8GB trở lên

## 2.1. Bước 1: Bật WSL2

Mở PowerShell (Administrator):

```powershell
wsl --install
```

Khởi động lại máy.

## 2.2. Bước 2: Cài Ubuntu từ Microsoft Store

Tìm "Ubuntu" → Install → Chạy lần đầu.

## 2.3. Bước 3: Tải Docker Desktop

Trang chủ Docker Desktop.

Cài đặt như phần mềm bình thường.

## 2.4. Bước 4: Kiểm tra

```bash
docker --version
docker run hello-world
```

Nếu hiện thông báo success → Docker hoạt động.

---

# 3. Cài đặt Docker trên macOS

Hỗ trợ:

* macOS 11+ (Apple Silicon & Intel)

## 3.1. Tải Docker Desktop

Cài đặt bằng file `.dmg`.

## 3.2. Kiểm tra

```bash
docker version
docker run hello-world
```

---

# 4. Cài đặt Docker Engine trên Linux (Ubuntu)

Ubuntu là OS phổ biến nhất để chạy Docker production.

## 4.1. Gỡ Docker cũ (nếu có)

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

## 4.2. Cài đặt Docker Engine

### Bước 1: Cập nhật & cài pre-requisites

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
```

### Bước 2: Thêm Docker GPG Key

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

### Bước 3: Thêm Docker Repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Bước 4: Cài Docker Engine

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 4.3. Kiểm tra Docker

```bash
docker run hello-world
```

---

# 5. Cấu hình quyền chạy Docker không cần sudo (Linux)

Mặc định phải dùng `sudo docker ...`, để chạy bình thường:

```bash
sudo usermod -aG docker $USER
```

Đăng xuất → đăng nhập lại.

Kiểm tra:

```bash
docker ps
```

---

# 6. Cài Docker Compose

Trong Docker Desktop đã có compose.

Linux cài plugin Compose:

```bash
sudo apt-get install docker-compose-plugin
```

Sử dụng:

```bash
docker compose version
```

---

# 7. Kiểm tra Docker hoạt động

## 7.1. Kiểm tra version

```bash
docker version
```

## 7.2. Kiểm tra info hệ thống

```bash
docker info
```

## 7.3. Chạy container test

```bash
docker run -d -p 8080:80 nginx
```

Truy cập: [http://localhost:8080](http://localhost:8080)

---

# 8. Cấu hình Docker cơ bản

## 8.1. Đường dẫn file cấu hình

* Windows/macOS: Docker Desktop GUI → Settings
* Linux: `/etc/docker/daemon.json`

## 8.2. Ví dụ cấu hình daemon.json

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Sau khi chỉnh cấu hình:

```bash
sudo systemctl restart docker
```

---

# 9. Các lỗi thường gặp khi cài Docker

## 9.1. Không bật WSL2 (Windows)

Lỗi:

```
Docker requires WSL2
```

→ Bật WSL2.

## 9.2. Không đủ RAM

Docker Desktop cần 4–8GB RAM.

## 9.3. Port 80/443 bị chiếm

Fix:

```bash
net stop http
```

## 9.4. Lỗi permission trên Linux

Fix:

```bash
sudo usermod -aG docker $USER
```

---
