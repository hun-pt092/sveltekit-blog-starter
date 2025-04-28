---
title: "Tìm hiểu về Hệ thống phân tán"
slug: "he-thong-phan-tan"
date: "2025-04-28"
excerpt: "Bài viết giới thiệu hệ thống phân tán, các khái niệm quan trọng, thuật ngữ và ví dụ ứng dụng trong thực tế."
coverImage: "/images/distribu-system.png"
coverWidth: 1200
coverHeight: 630
categories: ["công nghệ", "hệ thống phân tán"]
---
**Tác giả:** Nguyễn Duy Hưng
# Hệ thống phân tán là gì?

Hệ thống phân tán (Distributed System) là một tập hợp các máy tính độc lập kết nối với nhau thông qua mạng, phối hợp hoạt động như một hệ thống thống nhất. Dù các thành phần nằm rải rác về mặt địa lý và hoạt động độc lập, nhưng chúng cùng chia sẻ tài nguyên, dữ liệu và công việc để thực hiện các tác vụ lớn một cách hiệu quả.

Trong một hệ thống phân tán, người dùng không cần biết hoặc quan tâm đến việc dữ liệu đến từ đâu, xử lý ở đâu – mọi thứ diễn ra "trong suốt", tự động và hiệu quả.

---

# Các ứng dụng của hệ thống phân tán

Hệ thống phân tán là nền tảng của hầu hết các ứng dụng hiện đại:

- **Dịch vụ lưu trữ đám mây**: Google Drive, Dropbox, OneDrive
- **Nền tảng microservices**: Các hệ thống nội bộ tại các tập đoàn lớn như Amazon, Tiki, Grab
- **Dịch vụ truyền phát trực tuyến**: Netflix, YouTube, Spotify
- **Mạng xã hội**: Facebook, Instagram, TikTok
- **Hệ thống tài chính**: Internet Banking, ví điện tử như Momo, ZaloPay
- **Blockchain và tiền mã hóa**: Ethereum, Bitcoin
- **Công cụ tìm kiếm**: Google Search, Bing

---

# Các khái niệm chính của hệ thống phân tán

## Scalability (Khả năng mở rộng)

Khả năng hệ thống thích nghi với lưu lượng tăng cao bằng cách mở rộng tài nguyên. Có hai dạng:
- **Vertical Scaling**: Tăng cường phần cứng cho máy chủ hiện tại.
- **Horizontal Scaling**: Thêm nhiều máy chủ mới để chia tải.

## Fault Tolerance (Khả năng chịu lỗi)

Khi một hoặc nhiều thành phần bị lỗi, hệ thống vẫn hoạt động ổn định bằng cách chuyển hướng hoặc phục hồi dữ liệu/tác vụ.

## Availability (Tính sẵn sàng)

Khả năng hệ thống luôn sẵn sàng phục vụ người dùng. Một hệ thống phân tán tốt có thời gian hoạt động gần như tuyệt đối (99.999%).

## Transparency (Tính trong suốt)

Người dùng không nhận thấy sự phân tán – họ tương tác như thể hệ thống là một khối thống nhất. Transparency bao gồm:
- Location transparency
- Access transparency
- Replication transparency
- Concurrency transparency

## Concurrency (Tính đồng thời)

Hệ thống có thể xử lý nhiều yêu cầu cùng lúc từ hàng ngàn người dùng, ở nhiều vị trí khác nhau.

## Parallelism (Tính song song)

Hệ thống có thể chia nhỏ công việc để xử lý song song nhằm tăng tốc độ. Ví dụ: render video, xử lý dữ liệu lớn.

## Openness (Tính mở)

Hệ thống dễ tích hợp, phát triển thêm, nhờ tuân thủ các chuẩn mở như HTTP, RESTful, gRPC...

## Vertical Scaling

Mở rộng bằng cách nâng cấp phần cứng máy chủ – nhanh nhưng giới hạn và đắt đỏ.

## Horizontal Scaling

Thêm nhiều node vào hệ thống – hiệu quả và bền vững hơn.

## Load Balancer (Bộ cân bằng tải)

Phân phối lưu lượng truy cập đến các máy chủ khác nhau, giúp tối ưu hiệu suất và tránh tình trạng quá tải.

## Replication (Sao chép dữ liệu)

Nhân bản dữ liệu đến nhiều node để tăng độ tin cậy, giảm độ trễ, và hỗ trợ phục hồi khi gặp lỗi.

---

# Ví dụ thực tế: Shopee

Giả sử bạn đang sử dụng **Shopee** – một nền tảng thương mại điện tử phổ biến:

- Khi bạn truy cập trang web, yêu cầu của bạn được gửi đến một **Load Balancer**, rồi phân phối đến máy chủ gần nhất.
- Mỗi đơn hàng, thông tin tài khoản, hình ảnh sản phẩm đều được **replicate** nhiều nơi.
- Khi có Flash Sale, hệ thống sẽ tự động **horizontal scale** để mở rộng và chịu tải tốt hơn.
- Nếu một máy chủ bị lỗi, hệ thống chuyển hướng sang node khác (**fault tolerance**).
- Trong quá trình thanh toán, nhiều dịch vụ cùng phối hợp đồng thời (**concurrency**) và song song (**parallelism**).

→ Đây là một minh họa rõ ràng cho các tính chất: **scalability, fault tolerance, availability, replication, concurrency**, v.v.

---

# Kiến trúc của hệ thống phân tán

## 1. Client-Server

Cấu trúc cơ bản nhất: máy khách (client) gửi yêu cầu, máy chủ (server) xử lý và phản hồi.

## 2. Peer-to-Peer (P2P)

Mỗi node đều có thể là client và server, chia sẻ tài nguyên ngang hàng. Ví dụ: BitTorrent, IPFS.

## 3. Microservices Architecture

Ứng dụng được chia thành các dịch vụ nhỏ, triển khai và mở rộng độc lập. Rất phổ biến trong các hệ thống cloud-native.

## 4. Service-Oriented Architecture (SOA)

Các dịch vụ giao tiếp với nhau qua các chuẩn như SOAP, WSDL. Tiền thân của Microservices.

## 5. Event-Driven Architecture (EDA)

Các thành phần trong hệ thống tương tác thông qua các sự kiện (event) – giúp hệ thống phản ứng nhanh, mềm dẻo.

---

# Ví dụ: Netflix – Kiến trúc phân tán thực tế

**Netflix** là một ví dụ tiêu biểu về hệ thống phân tán quy mô toàn cầu:

- Sử dụng **Microservices**: Hàng trăm dịch vụ nhỏ phối hợp để cung cấp giao diện người dùng, thanh toán, cá nhân hóa...
- Dựa trên **AWS Cloud**: Khả năng **autoscale** linh hoạt theo lưu lượng người dùng toàn cầu.
- Dữ liệu được **replicate** tại nhiều khu vực địa lý để đảm bảo tốc độ truy cập và tính sẵn sàng.
- Sử dụng **Load Balancer**, **CDN**, và các công cụ như Chaos Monkey để kiểm tra độ bền (**fault tolerance**).
- Toàn bộ hoạt động xử lý từ người dùng, phân phối nội dung, đề xuất, ghi nhận hành vi... được thực hiện một cách **song song** và **đồng thời** ở quy mô hàng triệu người dùng mỗi ngày.

---
# Mô phỏng kiến trúc hệ thống phân tán bằng mã nguồn

Dưới đây là ví dụ đơn giản minh họa cách xây dựng một dịch vụ đơn giản trong kiến trúc microservices sử dụng Node.js, Redis, và thể hiện cách load balancer hoạt động.

## Ví dụ 1: Microservice API đơn giản với Node.js

```js
// service.js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/status', (req, res) => {
  res.send({ service: `Service on port ${PORT}`, status: 'OK' });
});

app.listen(PORT, () => {
  console.log(`Service is running on port ${PORT}`);
});
```

# Kết luận

Hệ thống phân tán là nền tảng của hầu hết các ứng dụng hiện đại, từ mạng xã hội đến thương mại điện tử và dịch vụ video. Việc hiểu rõ các khái niệm như scalability, availability, fault tolerance hay kiến trúc microservices sẽ giúp bạn xây dựng những hệ thống mạnh mẽ, linh hoạt và có khả năng phát triển lâu dài.

---

> 📌 *Tham khảo: Slide bài giảng môn Hệ thống phân tán, Wikipedia, AWS Architecture Blog, Netflix Tech Blog, Martin Fowler on Microservices.*
