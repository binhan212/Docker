# Chương 4: Docker Container — Quản lý, Vận hành và Thực hành thực tế

Trong chương này, bạn sẽ hiểu rõ **container là gì**, cách **tạo – chạy – quản lý – gỡ lỗi** container, và cách vận hành chúng trong môi trường thực tế.

---

# 1. Docker Container là gì?

Container là môi trường chạy ứng dụng được đóng gói sẵn từ Docker Image.

Một container có:

* File system riêng (từ image)
* Mạng riêng
* Process riêng
* Cổng xuất/nhập

Container **không phải** máy ảo vì:

* Không cần Hypervisor
* Không tạo OS mới
* Chạy trực tiếp trên kernel của host → rất nhẹ

---

# 2. Kiểm tra container trên máy

```bash
docker ps              # danh sách container đang chạy
docker ps -a           # toàn bộ container kể cả đã stop
```

---

# 3. Tạo và chạy container

## 3.1. Chạy container đơn giản

```bash
docker run nginx
```

Lệnh này chạy Nginx foreground (chiếm terminal). Nếu tắt terminal → container tắt.

## 3.2. Chạy container dưới dạng nền (background)

```bash
docker run -d nginx
```

`-d` = detached mode.

## 3.3. Map port container ↔ host

```bash
docker run -d -p 8080:80 nginx
```

Truy cập ứng dụng tại:

```
http://localhost:8080
```

## 3.4. Đặt tên container

```bash
docker run -d --name web1 nginx
```

## 3.5. Chạy container với volume

```bash
docker run -d -p 8080:80 -v /mydata:/usr/share/nginx/html nginx
```

Volume giúp bạn cập nhật dữ liệu mà không phải rebuild image.

---

# 4. Quản lý container

## 4.1. Stop container

```bash
docker stop web1
```

## 4.2. Start container đã dừng

```bash
docker start web1
```

## 4.3. Restart container

```bash
docker restart web1
```

## 4.4. Xóa container

```bash
docker rm web1
```

Nếu container đang chạy, dùng:

```bash
docker rm -f web1
```

## 4.5. Xóa tất cả container stop

```bash
docker container prune
```

---

# 5. Truy cập vào bên trong container

Có 3 cách chính:

## 5.1. Dùng bash/sh

```bash
docker exec -it web1 bash
```

Hoặc:

```bash
docker exec -it web1 sh
```

## 5.2. Xem log container

```bash
docker logs web1
```

## 5.3. Xem log realtime

```bash
docker logs -f web1
```

---

# 6. Copy file giữa container và host

## Copy từ host → container

```bash
docker cp index.html web1:/usr/share/nginx/html/
```

## Copy từ container → host

```bash
docker cp web1:/etc/nginx/nginx.conf ./
```

---

# 7. Xem tài nguyên container

## 7.1. Xem process

```bash
docker top web1
```

## 7.2. Xem chi tiết container

```bash
docker inspect web1
```

---

# 8. Cấu hình resource container (CPU/RAM)

Hạn chế tài nguyên giúp server ổn định.

## 8.1. Giới hạn RAM

```bash
docker run -d --memory="256m" nginx
```

## 8.2. Giới hạn CPU

```bash
docker run -d --cpus="1.5" nginx
```

## 8.3. Tự động restart nếu container crash

```bash
docker run -d --restart=always nginx
```

Các mode restart:

* no
* always
* unless-stopped
* on-failure

---

# 9. Mạng (Network) trong Docker

Mặc định, Docker có 3 loại network:

## 9.1. bridge (mặc định)

Container có IP riêng và có thể map port ra ngoài.

Tạo network riêng:

```bash
docker network create mynet
```

Chạy container trong network:

```bash
docker run -d --name web1 --network=mynet nginx
```

## 9.2. host

Container dùng trực tiếp network của host.

```bash
docker run -d --network=host nginx
```

## 9.3. none

Container cô lập hoàn toàn.

---

# 10. Dừng toàn bộ container, xóa toàn bộ container & images

## Dừng tất cả container

```bash
docker stop $(docker ps -q)
```

## Xóa tất cả container

```bash
docker rm $(docker ps -aq)
```

---

# 11. Thực hành thực tế: Deploy website trong container

## Bước 1: Tạo file `index.html`

```html
<h1>Hello Docker</h1>
```

## Bước 2: Chạy container với volume

```bash
docker run -d \
  --name mysite \
  -p 8080:80 \
  -v $(pwd)/index.html:/usr/share/nginx/html/index.html \
  nginx
```

## Bước 3: Truy cập

```
http://localhost:8080
```

Nếu file `index.html` thay đổi → website cập nhật ngay.

---

# 12. Thực hành nâng cao: Deploy Node.js trong container

## 12.1. Tạo file `server.js`

```javascript
const http = require('http');
const port = 3000;
http.createServer((req, res) => {
  res.write("Docker Node.js running!");
  res.end();
}).listen(port, () => console.log("Server running on port 3000"));
```

## 12.2. Tạo Dockerfile

```Dockerfile
FROM node:18
WORKDIR /app
COPY . .
CMD ["node", "server.js"]
```

## 12.3. Build image

```bash
docker build -t nodeapp .
```

## 12.4. Chạy container

```bash
docker run -d -p 3000:3000 --name node1 nodeapp
```

---

# 13. Các lỗi thường gặp khi quản lý container

## 13.1. Port bị chiếm

```
bind: address already in use
```

Fix:

```bash
netstat -ano | findstr :8080
kill -f <PID>
```

## 13.2. Permission denied khi gắn volume Linux

Chạy container với quyền root hoặc fix quyền thư mục:

```bash
sudo chmod -R 777 /myfolder
```

## 13.3. Container khởi động xong tắt ngay

Do lệnh CMD kết thúc.
Giải pháp: chạy foreground process, ví dụ:

```bash
tail -f /dev/null
```

---
