# Chương 1: Giới thiệu về Docker

## 1. Docker là gì?

Docker là một nền tảng giúp đóng gói (package), phân phối (ship) và chạy (run) ứng dụng trong môi trường container. Container là một môi trường độc lập, nhẹ và chia sẻ kernel của hệ điều hành host, nhưng vẫn đảm bảo tính cô lập.

### 1.1. Khái niệm Container

* Container là một đơn vị phần mềm bao gồm mã nguồn, thư viện, cấu hình cần thiết để ứng dụng chạy.
* Giúp ứng dụng chạy giống nhau trên mọi môi trường: Dev → Test → Staging → Production.
* Khởi động rất nhanh (chỉ vài mili giây).

### 1.2. Docker giải quyết vấn đề gì?

* Giáo trình khác nhau giữa các máy ("máy anh chạy được mà máy em không chạy").
* Cài đặt môi trường mất thời gian.
* Khó triển khai ứng dụng có nhiều thành phần (DB, backend, frontend...).

Docker giải quyết tất cả bằng cách chuẩn hóa môi trường và đóng gói ứng dụng.

---

## 2. Ưu điểm của Docker

### 2.1. Nhẹ và nhanh

* Container chia sẻ kernel → nhẹ hơn máy ảo (VM).
* Khởi động nhanh → tăng tốc độ phát triển.

### 2.2. Dễ triển khai

* Chỉ cần 1 lệnh để chạy: `docker run ...`
* Triển khai giống nhau trên mọi OS.

### 2.3. Khả năng mở rộng (scaling)

* Tăng số lượng container dễ dàng.

### 2.4. Tính cô lập

* Mỗi container có file system, network, process riêng → tránh xung đột.

### 2.5. Tiết kiệm tài nguyên

* Không cần toàn bộ hệ điều hành như VM.

---

## 3. Docker hoạt động như thế nào?

Docker hoạt động dựa trên kiến trúc Client → Server.

### 3.1. Docker Client

Công cụ dòng lệnh (CLI) như `docker run`, `docker pull`.

### 3.2. Docker Engine (Daemon)

* Quản lý images
* Quản lý containers
* Quản lý network, volumes
* Nhận yêu cầu từ client

### 3.3. Docker Registry

Kho chứa images. Có 2 loại:

* Docker Hub (public)
* Private Registry (tự thiết lập)

---

## 4. So sánh Docker và Virtual Machine

| Tiêu chí         | Docker                 | Máy Ảo (VM) |
| ---------------- | ---------------------- | ----------- |
| Tài nguyên       | Nhẹ                    | Nặng        |
| Tốc độ khởi động | Mili giây              | Vài phút    |
| Cô lập ứng dụng  | Tốt                    | Rất tốt     |
| Dung lượng       | Nhỏ (MB – vài trăm MB) | Lớn (GB)    |
| Chia sẻ Kernel   | Có                     | Không       |
| Tính linh hoạt   | Cao                    | Trung bình  |

Docker phù hợp cho microservices, CI/CD, DevOps.
VM phù hợp cho chạy nhiều hệ điều hành khác nhau.

---

## 5. Một số thuật ngữ quan trọng cần nắm

### 5.1. Image

* Mẫu (template) chứa ứng dụng và môi trường chạy.
* Dùng để tạo container.

### 5.2. Container

* Phiên bản đang chạy của image.

### 5.3. Registry

* Nơi lưu trữ images.

### 5.4. Dockerfile

* File mô tả cách build image.

### 5.5. Volume

* Lưu dữ liệu bền vững.

### 5.6. Network

* Giúp các container giao tiếp với nhau.

---

## 6. Docker giải quyết vấn đề gì trong thực tế?

### 6.1. Backend nhiều phiên bản

Ví dụ: Máy dev dùng Node 18, server dùng Node 16 → chạy khác hành vi.

Docker giúp chuẩn hóa.

### 6.2. Triển khai Microservices

hàng chục service → Docker Compose quản lý.

### 6.3. Tối ưu CI/CD

Tạo môi trường build nhanh, tái sử dụng được.

### 6.4. Dễ dàng mở rộng

Thêm container để tăng tải.

---

## 7. Một ví dụ đơn giản với Docker

Chạy NGINX bằng 1 lệnh:

```bash
docker run -d -p 8080:80 nginx
```

Truy cập: [http://localhost:8080](http://localhost:8080)

---

## 8. Tổng kết chương 1

Trong chương 1, bạn đã nắm được:

* Docker là gì
* Tại sao Docker quan trọng
* Kiến trúc Docker
* So sánh Docker với VM
* Các khái niệm quan trọng

Bạn đã sẵn sàng để bắt đầu Chương 2: **Cài đặt Docker và cấu hình môi trường**.
