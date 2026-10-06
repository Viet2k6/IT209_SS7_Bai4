# Bài 4: Cấu hình Reverse Proxy Nginx cho ứng dụng Spring Boot

## 1. Mục tiêu

* Cấu hình Nginx Server Block làm Reverse Proxy.
* Phục vụ trang tĩnh tại `/var/www/html/`.
* Chuyển tiếp các request `/api/` đến backend tại `127.0.0.1:8082`.

## 2. Trang web tĩnh

File:

```text
/var/www/html/index.html
```

Nội dung trang hiển thị thông tin học viên:

```text
Họ tên: Đăng Việt
Mã lớp: K24
```

Kiểm tra:

```bash
curl -I http://localhost/
```

Kết quả:

```text
HTTP/1.1 200 OK
Server: nginx/1.28.3 (Ubuntu)
Content-Type: text/html
```

## 3. Cấu hình Reverse Proxy

File:

```text
/etc/nginx/sites-available/spring-proxy.conf
```

Cấu hình:

```nginx
server {
    listen 80;
    server_name localhost _;

    location / {
        root /var/www/html;
        index index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8082/;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Cấu hình được kích hoạt bằng symlink:

```text
/etc/nginx/sites-enabled/spring-proxy.conf
```

Server Block cũ `my-app` được tắt để tránh xung đột với `localhost`.

## 4. Kiểm tra cú pháp Nginx

Lệnh:

```bash
sudo nginx -t
```

Kết quả:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Sau đó reload Nginx:

```bash
sudo systemctl reload nginx
```

## 5. Kiểm tra Reverse Proxy

Backend giả lập chạy trên:

```text
127.0.0.1:8082
```

Kiểm tra:

```bash
curl -i http://localhost/api/health
```

Kết quả:

```text
HTTP/1.1 200 OK
Server: nginx/1.28.3 (Ubuntu)
Content-Type: text/plain
```

Nội dung backend trả về:

```text
Spring Boot backend is healthy
```

## 6. Kết luận

Nginx đã được cấu hình thành công với hai chức năng:

* Request `/` được phục vụ trực tiếp từ `/var/www/html/`.
* Request `/api/` được Reverse Proxy đến backend tại `127.0.0.1:8082`.

Kết quả kiểm tra HTTP đều đạt yêu cầu.
