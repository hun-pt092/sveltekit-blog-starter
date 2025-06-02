 ---
title: "Deliverable 3: Tiến độ dự án:"
slug: "he-thong"
date: "2025-05-27"
categories: ["web blog"]
---
 # Báo cáo tiến độ dự án: Hệ thống blog phân tán với CockroachDB và SvelteKit

## Tóm tắt tiến độ

Dự án hệ thống blog phân tán với tính năng bình luận thời gian thực đã đạt được các tiêu chí chính cho một hệ thống phân tán. Hệ thống hiện tại hoạt động đầy đủ với kiến trúc phân tán bao gồm:

- **Cơ sở dữ liệu phân tán**: Cluster 3 nút CockroachDB với khả năng sao chép và chịu lỗi
- **Backend phân tán**: 2 server SvelteKit chạy song song phía sau load balancer
- **Đồng bộ hóa thời gian thực**: Hệ thống WebSocket với Redis adapter cho phép bình luận thời gian thực
- **Giám sát hệ thống**: Prometheus thu thập metrics từ tất cả các thành phần

Hệ thống đáp ứng đầy đủ các yêu cầu:
- Fault Tolerance: Hệ thống vẫn hoạt động khi một nút gặp sự cố
- Distributed Communication: Các nút giao tiếp qua HTTP và WebSocket
- Replication: Dữ liệu được tự động sao chép giữa các nút CockroachDB
- Monitoring: Prometheus cung cấp giám sát toàn bộ hệ thống
- Stress Testing: Script kiểm tra khả năng chịu tải đã được triển khai

### Phần đã hoàn thành
- Cơ sở dữ liệu phân tán (CockroachDB)
- Backend servers với SvelteKit
- Tính năng bình luận thời gian thực
- Load balancing với Nginx
- Đồng bộ hóa giữa các backend bằng Redis
- Giám sát hệ thống với Prometheus
- Container hóa với Docker Compose
- Kiểm tra tải với stress-test.js

### Phần đang phát triển
- UI admin để quản lý nội dung
- Tính năng phân quyền nâng cao
- Bảo mật SSL/TLS cho kết nối database

### Thách thức và giải pháp
1. **Đồng bộ hóa WebSocket giữa các backend server**
   - **Vấn đề**: Khi người dùng kết nối đến các backend khác nhau, bình luận không đồng bộ
   - **Giải pháp**: Triển khai Redis adapter cho Socket.IO để đồng bộ hóa các sự kiện

2. **Kết nối database khi một node bị lỗi**
   - **Vấn đề**: Ứng dụng mất kết nối khi node database chính bị lỗi
   - **Giải pháp**: Cài đặt cơ chế failover trong db.js để tự động kết nối đến node khác

3. **Session persistence giữa các backend**
   - **Vấn đề**: Người dùng phải đăng nhập lại khi được chuyển hướng đến backend khác
   - **Giải pháp**: Sử dụng Redis để lưu trữ session

## Demo hoạt động của hệ thống

### Các demo được triển khai
- Demo bình luận thời gian thực (demo-youtube-comments.js)
- Kiểm tra hệ thống cuối cùng (final-system-test.js)
- Kiểm tra tải (stress-test.js)

### Kết quả kiểm thử
Theo báo cáo PROJECT_COMPLETION_REPORT.md:
```
Test Results Summary:
- Comment API: PASS
- WebSocket Real-time: PASS
- Load Balancing: PASS
All tests passed! Distributed blog system is working correctly.
```

Trạng thái container:
```
NAME            STATUS              PORTS
blog-backend1   Up (healthy)        0.0.0.0:3000-3001->3000-3001/tcp
blog-backend2   Up (healthy)        0.0.0.0:3002->3000/tcp, 0.0.0.0:3003->3001/tcp
cockroach1      Up                  0.0.0.0:26257->26257/tcp, 0.0.0.0:8085->8080/tcp
cockroach2      Up                  0.0.0.0:8086->8080/tcp, 0.0.0.0:26258->26257/tcp
cockroach3      Up                  0.0.0.0:8087->8080/tcp, 0.0.0.0:26259->26257/tcp
nginx-lb        Up                  0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
prometheus      Up                  0.0.0.0:9090->9090/tcp
redis           Up (healthy)        0.0.0.0:6379->6379/tcp
```

## Mã nguồn quan trọng

### 1. Cơ chế failover database (src/lib/db.js)
```javascript
// Thử kết nối tới tất cả các URL trong mảng cho đến khi một kết nối thành công
async function connectWithFailover() {
  const databaseUrls = getDatabaseUrls();
  let lastError = null;
  
  for (const url of databaseUrls) {
    try {
      const tempPool = new Pool({ connectionString: url });
      const client = await tempPool.connect();
      
      console.log('Connected to CockroachDB successfully');
      client.release();
      
      // Trả về URL kết nối thành công
      return { pool: tempPool, url };
    } catch (err) {
      console.error(`Failed to connect to CockroachDB at ${url.replace(/\/\/.*@/, '//***@')}:`, err.message);
      lastError = err;
    }
  }
  
  // Nếu không có kết nối nào thành công, ném lỗi cuối cùng
  throw lastError || new Error('Could not connect to any CockroachDB instances');
}
```

### 2. Xử lý bình luận thời gian thực (websocket-handler.js)
```javascript
// Xử lý khi nhận bình luận mới
socket.on('new_comment', async (data) => {
  // Đo thời gian xử lý bình luận
  const commentTimer = commentLatency.startTimer({ node_id: NODE_ID });
  
  try {
    // Lưu bình luận vào database
    const { post_id, author, content, postSlug } = data;
    
    // Validate dữ liệu
    if (!postSlug || !author || !content) {
      socket.emit('comment_error', { error: 'Invalid comment data' });
      commentsCounter.inc({ node_id: NODE_ID, status: 'invalid' });
      commentTimer();
      return;
    }

    // Insert vào database
    try {
      const result = await db.query(
        `INSERT INTO comments (post_id, author, content) 
         VALUES ($1, $2, $3) 
         RETURNING id, post_id, author, content, created_at`,
        [post_id, author, content]
      );

      const newComment = result.rows[0];
      
      // Phát lại bình luận đến tất cả các client đang theo dõi bài viết
      io.to(`post:${postSlug}`).emit('comment_added', {
        id: newComment.id,
        post_id: newComment.post_id,
        author: newComment.author,
        content: newComment.content,
        created_at: newComment.created_at
      });
      
      // Ghi nhận metrics cho bình luận thành công
      commentsCounter.inc({ node_id: NODE_ID, status: 'success' });
    } catch (dbError) {
      // Trường hợp không kết nối được database, vẫn phát sự kiện bình luận tạm thời
      // (nhưng không lưu vào database)
      const mockComment = {
        id: Date.now().toString(),
        post_id: post_id,
        author: author,
        content: content,
        created_at: new Date().toISOString()
      };
      
      io.to(`post:${postSlug}`).emit('comment_added', mockComment);
      commentsCounter.inc({ node_id: NODE_ID, status: 'db_error' });
    }
  } catch (error) {
    socket.emit('comment_error', { error: 'Failed to add comment' });
  } finally {
    commentTimer();
  }
});
```

### 3. Cấu hình Nginx Load Balancer (nginx/conf.d/default.conf)
```nginx
upstream backend_servers {
    server blog-backend1:3000;
    server blog-backend2:3000;
    ip_hash;  # Đảm bảo người dùng luôn được route đến cùng một server
}

upstream websocket_servers {
    server blog-backend1:3001;
    server blog-backend2:3001;
    ip_hash;  # Đảm bảo kết nối websocket luôn đến cùng server
}

server {
    listen 80;
    server_name localhost;
    
    location / {
        proxy_pass http://backend_servers;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # Route WebSocket to port 3001
    location /socket.io/ {
        proxy_pass http://websocket_servers;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

### 4. Kiểm tra tải (stress-test.js)
```javascript
function runSocketTest() {
  console.log('Bắt đầu kiểm tra WebSocket...');
  
  return new Promise((resolve) => {
    // Tạo các kết nối socket
    for (let i = 0; i < NUM_CONNECTIONS; i++) {
      const socket = io(WS_URL, {
        reconnection: true,
        reconnectionAttempts: 3,
        transports: ['websocket']
      });
      
      socket.on('connect', () => {
        results.socketConnections++;
        
        // Tham gia phòng kiểm tra
        socket.emit('join_post', POST_SLUG);
        
        // Gửi bình luận mỗi 2 giây
        if (i % 5 === 0) { // Chỉ 20% client gửi bình luận để không quá tải
          const interval = setInterval(() => {
            socket.emit('new_comment', {
              post_id: '1',
              author: `Test User ${i}`,
              content: `This is a stress test comment from connection ${i} at ${new Date().toISOString()}`,
              postSlug: POST_SLUG
            });
            results.socketMessages++;
          }, 2000);
          
          socket._testInterval = interval;
        }
      });
    }
  });
}
```

## Danh sách tính năng đã hoàn thành

### 1. Fault-tolerant Architecture
- **Mô tả**: Hệ thống có thể tiếp tục hoạt động khi một nút hoặc quá trình bị lỗi
- **Triển khai**: CockroachDB với 3 nút, cơ chế failover trong db.js
- **Demo**: Khi tắt một nút CockroachDB, hệ thống tự động chuyển đến nút khác và tiếp tục hoạt động

### 2. Distributed Communication
- **Mô tả**: Các nút giao tiếp qua các giao thức mạng thực tế
- **Triển khai**: HTTP và WebSocket cho giao tiếp frontend-backend, Raft protocol trong CockroachDB
- **Demo**: Người dùng trên các máy khác nhau có thể nhìn thấy bình luận của nhau trong thời gian thực

### 3. Data Replication
- **Mô tả**: Dữ liệu được sao chép giữa các nút để đảm bảo tính sẵn sàng
- **Triển khai**: CockroachDB tự động sao chép dữ liệu với thuật toán Raft
- **Demo**: Khi một nút database offline, dữ liệu vẫn có thể truy cập từ nút khác

### 4. Simple Monitoring / Logging
- **Mô tả**: Hệ thống cung cấp khả năng giám sát và ghi nhật ký
- **Triển khai**: Prometheus thu thập metrics, log chi tiết trong websocket-handler.js và server.js
- **Demo**: Dashboard Prometheus tại http://localhost:9090 hiển thị metrics của hệ thống

### 5. Basic Stress Test
- **Mô tả**: Kiểm tra khả năng chịu tải của hệ thống
- **Triển khai**: Script stress-test.js mô phỏng nhiều kết nối đồng thời
- **Demo**: Script tạo 100 kết nối đồng thời và 20% trong số đó gửi bình luận mỗi 2 giây

### 6. Load Balancing
- **Mô tả**: Phân phối yêu cầu người dùng giữa các backend
- **Triển khai**: Nginx với cấu hình ip_hash để đảm bảo sticky sessions
- **Demo**: Khi nhiều người dùng truy cập, các yêu cầu được phân phối giữa các backend

### 7. System Recovery
- **Mô tả**: Nút có thể tham gia lại hệ thống sau khi gặp lỗi
- **Triển khai**: CockroachDB tự động tham gia lại cluster, cơ chế reconnect trong Socket.IO
- **Demo**: Sau khi khởi động lại một node, nó tự động tham gia lại cluster và đồng bộ hóa dữ liệu

## Kế hoạch tiếp theo

### Các tính năng cần hoàn thiện
1. **Giao diện quản trị (Admin UI)**
   - Dashboard quản lý nội dung
   - Quản lý người dùng và phân quyền
   - Theo dõi metrics hệ thống

2. **Nâng cao bảo mật**
   - Triển khai SSL/TLS cho CockroachDB
   - Xác thực hai yếu tố cho người dùng
   - Mã hóa dữ liệu nhạy cảm

3. **Tối ưu hóa hiệu suất**
   - Caching với Redis cho các truy vấn phổ biến
   - Lazy loading cho nội dung media
   - Tối ưu hóa GraphQL API thay thế cho RESTful API

### Các vấn đề cần giải quyết
1. **Đồng bộ hóa dữ liệu hiệu quả hơn**
   - Giảm lưu lượng mạng giữa các nút
   - Kiểm soát xung đột dữ liệu tốt hơn

2. **Khả năng mở rộng toàn cầu**
   - Triển khai multi-region cho database
   - CDN cho nội dung tĩnh
   - Edge computing cho một số tính năng

3. **Giám sát và cảnh báo nâng cao**
   - Alerting khi phát hiện vấn đề
   - Phân tích log tự động
   - Theo dõi hiệu suất người dùng cuối

### Lộ trình triển khai
1. **Hoàn thiện Admin UI** - 2 tuần
2. **Nâng cấp bảo mật** - 1 tuần
3. **Tối ưu hiệu suất** - 2 tuần
4. **Kiểm thử tổng hợp** - 1 tuần
5. **Triển khai production** - 1 tuần

### Thử thách dự kiến
- Đảm bảo hiệu suất ổn định khi mở rộng quy mô
- Duy trì tính đồng bộ giữa các vùng địa lý khác nhau
- Cân bằng giữa tính nhất quán dữ liệu và độ trễ người dùng

Hệ thống đã đạt được các mục tiêu chính của một ứng dụng phân tán và sẵn sàng cho việc mở rộng và triển khai trong môi trường production.
