---
title: "Deliverable 4: Hệ thống xây dựng blog phân tán bằng cockroach đến đâu"
slug: "he-thong"
date: "2025-06-02"
categories: ["web blog"]
---
# Phân tích tiến độ dự án theo các tiêu chí yêu cầu

Dựa trên phân tích code và cấu trúc dự án, tôi sẽ đánh giá chi tiết việc đáp ứng các tiêu chí yêu cầu của hệ thống blog phân tán:

## Tiêu chí bắt buộc

### 1. Fault Tolerance (Hoàn thành)
- **Đã triển khai**:
  - Cụm CockroachDB 3 nút đảm bảo dữ liệu vẫn có sẵn khi một nút bị lỗi
  - Cơ chế failover trong db.js tự động chuyển đổi kết nối khi một database node bị lỗi
  - Xử lý lỗi trong websocket-handler.js cho phép bình luận tạm thời ngay cả khi database gặp sự cố
  - Cấu hình `restart: always` trong Docker Compose để tự động khởi động lại các container bị lỗi

- **Ví dụ code**:
```javascript
async function connectWithFailover() {
  const databaseUrls = getDatabaseUrls();
  
  for (const url of databaseUrls) {
    try {
      const tempPool = new Pool({ connectionString: url });
      const client = await tempPool.connect();
      return { pool: tempPool, url };
    } catch (err) {
      console.error(`Failed to connect to CockroachDB at ${url}`);
    }
  }
  throw new Error('Could not connect to any CockroachDB instances');
}
```

### 2. Distributed Communication (Hoàn thành)
- **Đã triển khai**:
  - HTTP và WebSocket cho giao tiếp client-server
  - Redis PubSub cho giao tiếp giữa các backend server
  - Raft protocol trong CockroachDB cho giao tiếp giữa các database node
  - NGINX load balancer điều phối giao tiếp

- **Ví dụ code** (Redis adapter cho Socket.IO):
```javascript
// Thiết lập Redis adapter cho Socket.IO
const pubClient = createClient({ url: process.env.REDIS_URL || 'redis://redis:6379' });
const subClient = pubClient.duplicate();
io.adapter(createAdapter(pubClient, subClient));
```

### 3. Replication (Hoàn thành)
- **Đã triển khai**:
  - CockroachDB tự động sao chép dữ liệu giữa 3 nút theo thuật toán Raft
  - Mỗi phân đoạn dữ liệu (range) có ít nhất 3 bản sao trên các nút khác nhau
  - Cấu hình `cockroach1`, `cockroach2`, `cockroach3` trong Docker Compose với join command

- **Cấu hình**:
```yaml
cockroach1:
  command: start --insecure --join=cockroach1,cockroach2,cockroach3
```

### 4. Simple Monitoring / Logging (Hoàn thành)
- **Đã triển khai**:
  - Prometheus thu thập metrics từ tất cả các dịch vụ
  - Custom metrics trong websocket-handler.js (kết nối, bình luận, độ trễ)
  - Logging chi tiết với NODE_ID để phân biệt logs từ các nút khác nhau
  - Health checks cho các container

- **Cấu hình**:
```yaml
prometheus:
  image: prom/prometheus:latest
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml
```

- **Metrics code**:
```javascript
const wsConnectionsCounter = new Counter({
  name: 'websocket_connections_total',
  help: 'Total count of WebSocket connections',
  labelNames: ['node_id']
});
```

### 5. Basic Stress Test (Hoàn thành)
- **Đã triển khai**:
  - Script stress-test.js mô phỏng nhiều kết nối HTTP và WebSocket đồng thời
  - Tạo bình luận tự động từ nhiều client
  - Theo dõi và báo cáo hiệu suất

- **Ví dụ code**:
```javascript
for (let i = 0; i < NUM_CONNECTIONS; i++) {
  const socket = io(WS_URL, {
    reconnection: true,
    reconnectionAttempts: 3,
    transports: ['websocket']
  });
  
  socket.on('connect', () => {
    // Gửi bình luận mỗi 2 giây từ 20% client
    if (i % 5 === 0) {
      const interval = setInterval(() => {
        socket.emit('new_comment', { ... });
      }, 2000);
    }
  });
}
```

## Tiêu chí tùy chọn

### 1. System Recovery (Hoàn thành)
- **Đã triển khai**:
  - CockroachDB tự động tham gia lại cluster và đồng bộ hóa dữ liệu sau khi khởi động lại
  - Cơ chế reconnect trong Socket.IO và Redis
  - Retry logic trong db.js

### 2. Load Balancing (Hoàn thành)
- **Đã triển khai**:
  - NGINX load balancer phân phối yêu cầu giữa các backend servers
  - Thuật toán ip_hash để đảm bảo sticky sessions cho WebSocket
  - Health checks để loại bỏ backend không khỏe mạnh

- **Cấu hình NGINX**:
```nginx
upstream backend_servers {
    server blog-backend1:3000;
    server blog-backend2:3000;
    ip_hash;
}
```

### 3. Consistency Guarantees (Hoàn thành)
- **Đã triển khai**:
  - CockroachDB với thuật toán Raft đảm bảo tính nhất quán mạnh mẽ (strong consistency)
  - Giao dịch ACID trong CockroachDB
  - Redis đồng bộ hóa WebSocket giữa các backend

### 4. Deployment Automation (Hoàn thành)
- **Đã triển khai**:
  - Docker Compose cho triển khai toàn bộ hệ thống
  - Scripts PowerShell (start-dev-distributed.ps1, deploy.ps1) cho automation
  - Health checks để xác nhận dịch vụ đã sẵn sàng

- **Cấu hình Docker Compose**:
```yaml
services:
  cockroach1:
    image: cockroachdb/cockroach:v23.1.11
    # ...
  blog-backend1:
    build:
      context: .
      dockerfile: Dockerfile
    # ...
```

### 5. Security Features (Một phần)
- **Đã triển khai một phần**:
  - Có đề cập đến auth.js và các script SQL liên quan (add-auth-system.sql)
  - System authentication đã được cấu hình
  
- **Chưa triển khai**:
  - Kết nối bảo mật TLS/SSL giữa các node (hiện đang sử dụng `--insecure`)
  - Mã hóa dữ liệu nhạy cảm

### Một số hình ảnh về dự án

![Ảnh phân tán](/images/cpux.png)
![](/images/vdthread-process.jpg)

## Tóm tắt tiến độ dự án

Dự án đã hoàn thành tất cả 5 tiêu chí bắt buộc và 4/5 tiêu chí tùy chọn đã chọn (Load Balancing, System Recovery, Consistency Guarantees và Deployment Automation). Tiêu chí Security Features đã được triển khai một phần.

### Các thành phần đã hoàn thành:
- Cụm CockroachDB 3 nút với sao chép dữ liệu
- 2 backend SvelteKit với load balancing
- Redis để đồng bộ hóa WebSocket
- Prometheus monitoring
- Stress testing
- Docker Compose deployment

### Phần cần cải thiện:
- Bổ sung bảo mật TLS/SSL cho kết nối database
- Hoàn thiện hệ thống xác thực
- Cải thiện monitoring dashboard
- Cần triển khai trên production.
- thời gian chưa đủ để hoàn thành đẩy đủ
- app còn vấn đề chưa đầy đủ hoàn thiện
- 1 số chức năng chưa hoàn thành

Hệ thống blog phân tán với tính năng bình luận thời gian thực sẵn sàng cho việc mở rộng thêm.