---
title: "Deliverable 2: Thiết kế hệ thống"
slug: "he-thong"
date: "2025-05-17"
categories: ["web blog"]
---
# Hệ thống blog phân tán với tính năng bình luận thời gian thực, sử dụng CockroachDB và SvelteKit

## 1. Bản vẽ kiến trúc hệ thống

```
                             ┌──────────────┐
                             │    Client    │
                             │   (Browser)  │
                             └───────┬──────┘
                                     │
                                     │ HTTP/WebSocket
                                     ▼
                             ┌───────────────┐
                             │  NGINX LB     │
                             │(Load Balancer)│
                             └─┬────────────┬┘
                               │            │
           ┌───────────────────┘            └───────────────────┐
           │                                                    │
┌──────────▼─────────┐                              ┌───────────▼─────────┐
│   Blog Backend 1   │◄────Socket.IO Redis PubSub──►│   Blog Backend 2    │
│ (SvelteKit+Node.js)│                              │ (SvelteKit+Node.js) │
└──────────┬─────────┘                              └──────────┬──────────┘
           │                                                   │
           │                    ┌──────────┐                   │
           └────────────────────►  Redis   ◄───────────────────┘
                                │ (PubSub) │
                                └──────────┘
           │                                                   │
           │                                                   │
┌──────────▼─────────┐      ┌──────────────────┐      ┌───────▼──────────┐
│   CockroachDB 1    │◄────►│   CockroachDB 2   ◄────►│   CockroachDB 3  │
│     (Node 1)       │      │     (Node 2)     │      │     (Node 3)     │
└────────────────────┘      └──────────────────┘      └──────────────────┘
                                     ▲
                                     │
                                     │
                              ┌─────────────┐
                              │ Prometheus  │
                              │(Monitoring) │
                              └─────────────┘
```

## 2. Mô tả chi tiết các thành phần trong hệ thống

### Client (Trình duyệt người dùng)
- **Vai trò**: Giao diện cho người dùng tương tác với hệ thống blog
- **Chức năng**:
  - Hiển thị nội dung blog được render từ SvelteKit
  - Tương tác với API để đọc và gửi bình luận
  - Duy trì kết nối WebSocket để nhận bình luận thời gian thực
  - Xử lý tương tác người dùng (đăng nhập, bình luận, vote)

### NGINX Load Balancer
- **Vai trò**: Phân phối lưu lượng truy cập giữa các backend server
- **Chức năng**:
  - Cân bằng tải HTTP request giữa các backend servers
  - Định tuyến WebSocket connections đến đúng backend
  - Đảm bảo "sticky sessions" với thuật toán ip_hash để người dùng luôn kết nối đến cùng một backend server
  - Xử lý health checks để loại bỏ backend không hoạt động

### Blog Backend (SvelteKit + Node.js)
- **Vai trò**: Xử lý logic ứng dụng, render trang web và quản lý WebSocket
- **Chức năng**:
  - Render trang web bằng SvelteKit với SSR (Server-Side Rendering)
  - Xử lý API requests (RESTful) để quản lý bài viết và bình luận
  - Quản lý kết nối WebSocket cho bình luận thời gian thực
  - Tương tác với CockroachDB để lưu trữ và truy xuất dữ liệu
  - Xác thực và phân quyền người dùng

### Redis
- **Vai trò**: Đồng bộ hóa WebSocket giữa các backend servers
- **Chức năng**:
  - Cung cấp kênh PubSub cho các backend servers
  - Đảm bảo bình luận thời gian thực được gửi đến tất cả người dùng bất kể họ kết nối đến backend nào
  - Hỗ trợ sự nhất quán và đồng bộ giữa các backend servers

### CockroachDB Cluster
- **Vai trò**: Cơ sở dữ liệu phân tán SQL với khả năng chịu lỗi cao
- **Chức năng**:
  - Lưu trữ dữ liệu bài viết, bình luận, người dùng
  - Đảm bảo tính nhất quán và sẵn sàng của dữ liệu
  - Tự động sao chép dữ liệu giữa các nút
  - Phục hồi sau sự cố mà không mất dữ liệu

**Cơ chế Replication trong CockroachDB:**
- Sử dụng thuật toán đồng thuận Raft để sao chép dữ liệu
- Mỗi phân đoạn dữ liệu (range) có ít nhất 3 bản sao (replicas)
- Một nút được chọn làm leader cho mỗi phân đoạn
- Leader xử lý tất cả các yêu cầu ghi, sau đó sao chép đến các follower
- Khi leader gặp sự cố, một follower mới sẽ được bầu làm leader

**Cơ chế Sharding (phân mảnh):**
- Dữ liệu được chia thành các phân đoạn (ranges) theo khóa
- Phân đoạn được tạo và chia tự động khi dữ liệu tăng lên
- Cơ chế cân bằng tải tự động di chuyển phân đoạn giữa các nút
- Mỗi nút CockroachDB chịu trách nhiệm cho nhiều phân đoạn
- Hỗ trợ truy vấn phân tán: truy vấn có thể cần dữ liệu từ nhiều phân đoạn trên nhiều nút khác nhau

### Prometheus
- **Vai trò**: Giám sát và thu thập metrics từ hệ thống
- **Chức năng**:
  - Thu thập metrics từ tất cả các thành phần hệ thống
  - Lưu trữ dữ liệu theo thời gian
  - Cung cấp API để truy vấn metrics
  - Giám sát hiệu suất và tình trạng hệ thống

## 3. Công nghệ và thư viện sử dụng

### CockroachDB
- **Lý do lựa chọn**: Là cơ sở dữ liệu SQL phân tán với khả năng mở rộng theo chiều ngang
- **Ưu điểm**:
  - Tính nhất quán mạnh mẽ (tuân thủ ACID) trong môi trường phân tán
  - Tự động sao chép dữ liệu và phục hồi sau lỗi
  - Khả năng mở rộng tuyến tính bằng cách thêm nút
  - Tương thích với giao thức PostgreSQL
  - Phân tán địa lý (geo-partitioning) nếu cần

### SvelteKit
- **Lý do lựa chọn**: Framework hiện đại cho cả front-end và back-end
- **Ưu điểm**:
  - Server-side rendering (SSR) giúp tăng SEO
  - Hydration giúp giao diện người dùng phản hồi nhanh
  - File-based routing đơn giản hóa cấu trúc ứng dụng
  - JavaScript ít hơn gửi đến client nhờ compile-time
  - API endpoints tích hợp trong cùng một codebase

### Socket.IO
- **Lý do lựa chọn**: Thư viện WebSocket với nhiều tính năng nâng cao
- **Ưu điểm**:
  - Khả năng tự động chuyển đổi giữa WebSocket, long polling, và các kỹ thuật khác
  - Hỗ trợ phân chia theo phòng (rooms) cho quản lý kết nối
  - Redis adapter cho phép đồng bộ hóa giữa nhiều server
  - Xử lý tự động kết nối lại và phục hồi

### Redis
- **Lý do lựa chọn**: In-memory database tốc độ cao với tính năng PubSub
- **Ưu điểm**:
  - Hiệu suất cao với độ trễ thấp
  - Hỗ trợ Publish/Subscribe pattern
  - Socket.IO Redis adapter được tích hợp sẵn
  - Dễ dàng triển khai và quản lý

### NGINX
- **Lý do lựa chọn**: Web server và reverse proxy hiệu suất cao
- **Ưu điểm**:
  - Khả năng cân bằng tải tin cậy
  - Xử lý đồng thời kết nối HTTP và WebSocket
  - Cấu hình linh hoạt để định tuyến yêu cầu
  - Hiệu suất cao với tài nguyên hệ thống thấp

### Docker & Docker Compose
- **Lý do lựa chọn**: Công cụ container hóa và điều phối
- **Ưu điểm**:
  - Đóng gói ứng dụng và dependencies trong container nhất quán
  - Docker Compose cho phép định nghĩa và chạy multi-container Docker applications
  - Cung cấp môi trường nhất quán giữa development và production
  - Dễ dàng triển khai và mở rộng

### Prometheus
- **Lý do lựa chọn**: Hệ thống monitoring và alerting mã nguồn mở
- **Ưu điểm**:
  - Thu thập metrics dựa trên pull model
  - Lưu trữ time-series data hiệu quả
  - Ngôn ngữ truy vấn mạnh mẽ (PromQL)
  - Tích hợp dễ dàng với nhiều exporter

## 4. Mô hình dữ liệu (Database Model)

### Bảng `posts`
```sql
CREATE TABLE IF NOT EXISTS posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  slug TEXT NOT NULL UNIQUE,
  content TEXT NOT NULL,
  excerpt TEXT,
  coverImage TEXT,
  coverWidth INT,
  coverHeight INT,
  date TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP,
  categories TEXT[],
  created_at TIMESTAMP NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_posts_slug ON posts(slug);
CREATE INDEX IF NOT EXISTS idx_posts_date ON posts(date DESC);
```

### Bảng `comments`
```sql
CREATE TABLE IF NOT EXISTS comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  author TEXT NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  updated_at TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_comments_post_id ON comments(post_id);
```

### Bảng `users`
```sql
CREATE TABLE IF NOT EXISTS users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(20) DEFAULT 'user',
  display_name VARCHAR(100),
  avatar_url TEXT,
  created_at TIMESTAMP DEFAULT current_timestamp(),
  updated_at TIMESTAMP DEFAULT current_timestamp()
);
```

### Bảng `votes`
```sql
CREATE TABLE IF NOT EXISTS votes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  comment_id UUID NOT NULL REFERENCES comments(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id),
  vote_type VARCHAR(10) NOT NULL CHECK (vote_type IN ('like', 'dislike')),
  created_at TIMESTAMP DEFAULT current_timestamp(),
  UNIQUE(comment_id, user_id)
);

CREATE INDEX IF NOT EXISTS idx_votes_comment ON votes(comment_id);
CREATE INDEX IF NOT EXISTS idx_votes_user ON votes(user_id);
```

### Mối quan hệ giữa các bảng
- Một bài viết (`posts`) có nhiều bình luận (`comments`)
- Một bình luận thuộc về một bài viết
- Một người dùng (`users`) có thể tạo nhiều bài viết và bình luận
- Một bình luận có thể nhận được nhiều votes từ các người dùng khác nhau
- Một người dùng chỉ có thể vote một lần cho mỗi bình luận

## 5. Chiến lược triển khai và cấu hình hệ thống

### Containerization với Docker
- Mỗi thành phần hệ thống được đóng gói trong container riêng biệt
- Sử dụng Dockerfile để định nghĩa môi trường cho Blog Backend
- Sử dụng các image chính thức cho CockroachDB, Redis, NGINX và Prometheus

### Orchestration với Docker Compose
- File docker-compose.yml định nghĩa toàn bộ stack ứng dụng
- Cấu hình networks để các container có thể giao tiếp với nhau
- Định nghĩa volumes để dữ liệu được lưu trữ bền vững
- Cấu hình restart policies để tự động khởi động lại các container gặp sự cố
- Sử dụng healthchecks để đảm bảo các dịch vụ hoạt động đúng

### Quy trình deployment
1. **Khởi tạo CockroachDB cluster**:
   - 3 nodes CockroachDB được khởi động
   - Container init khởi tạo cluster
   - Database và schema được tạo thông qua script

2. **Khởi động Redis**:
   - Cấu hình với appendonly để đảm bảo dữ liệu bền vững
   - Kiểm tra health với Redis client

3. **Triển khai Backend Services**:
   - Build và khởi động 2 instances SvelteKit
   - Mỗi instance kết nối đến một node CockroachDB chính
   - Cấu hình WebSocket server trên mỗi instance
   - Redis adapter được thiết lập cho đồng bộ hóa

4. **Cấu hình NGINX Load Balancer**:
   - Cấu hình upstream cho HTTP và WebSocket
   - Thiết lập ip_hash để đảm bảo sticky sessions
   - Cấu hình proxy cho SvelteKit và WebSocket

5. **Thiết lập Monitoring**:
   - Prometheus thu thập metrics từ tất cả các thành phần
   - Custom metrics được định nghĩa trong ứng dụng

### Khả năng mở rộng
- **Thêm Backend Nodes**: 
  - Thêm mục mới trong docker-compose.yml
  - Cập nhật cấu hình NGINX cho upstream mới
  - Không cần thay đổi code ứng dụng

- **Thêm Database Nodes**:
  - Thêm mục CockroachDB mới trong docker-compose
  - Cập nhật chuỗi join trong cấu hình CockroachDB
  - CockroachDB tự động thêm nút mới vào cluster và tái cân bằng dữ liệu

- **Theo dõi và Bảo trì**:
  - Scripts giám sát sức khỏe hệ thống
  - Tự động backup dữ liệu
  - Log rotation và aggregation

Với kiến trúc được thiết kế này, hệ thống blog phân tán có khả năng chịu lỗi cao, mở rộng dễ dàng và hỗ trợ tính năng bình luận thời gian thực mượt mà, đáp ứng đầy đủ yêu cầu cho một hệ thống phân tán hiện đại.
