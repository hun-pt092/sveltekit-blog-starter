---
title: "Deliverable 1: Đề xuất đề tài và mô tả vấn đề"
slug: "he-thong-phan-tan"
date: "2025-05-10"
categories: ["web blog"]
---

## 📄 Deliverable 1: Đề xuất đề tài và mô tả vấn đề

---

##  1. Đề tài đề xuất

**Xây dựng hệ thống blog phân tán với tính năng bình luận thời gian thực, sử dụng CockroachDB và SvelteKit.**

---

##  2. Mô tả vấn đề cần giải quyết

Các hệ thống blog hiện nay thường được xây dựng theo mô hình tập trung, dễ gặp phải các vấn đề như:

- Quá tải khi có nhiều người dùng đồng thời.
- Gián đoạn dịch vụ khi server chính gặp sự cố.
- Khó mở rộng sang các khu vực địa lý khác.

Đề tài này hướng tới xây dựng một **nền tảng blog phân tán** với khả năng:

- Cho phép nhiều người dùng đăng bài, bình luận thời gian thực mà vẫn đảm bảo hiệu năng.
- Duy trì hoạt động ổn định ngay cả khi một hoặc vài node gặp lỗi.
- Lưu trữ và truy xuất dữ liệu một cách hiệu quả, nhất quán giữa các node nhờ CockroachDB.

---

## 🔍 3. Thư viện / Framework được giao: CockroachDB

### 🎯 Mục đích sử dụng

CockroachDB là hệ quản trị cơ sở dữ liệu phân tán mã nguồn mở, hỗ trợ SQL (PostgreSQL-compatible). Nó giúp xây dựng các hệ thống:

- Có khả năng **mở rộng theo chiều ngang**.
- **Tự động phục hồi** khi có lỗi phần cứng.
- **Đảm bảo nhất quán dữ liệu mạnh** trong môi trường phân tán.

### 🧠 CockroachDB giải quyết được gì?

- **Nhất quán mạnh**: Đồng bộ dữ liệu giữa các node một cách tự động.
- **Khả năng phục hồi lỗi cao**: Hệ thống vẫn hoạt động khi một hoặc nhiều node mất kết nối.
- **Không downtime khi mở rộng**: Có thể thêm node mới dễ dàng trong quá trình hoạt động.
- **Thích hợp cho ứng dụng quy mô lớn**, phân tán: mạng xã hội, hệ thống tài chính, e-commerce đa vùng.

### ⚖️ So sánh CockroachDB với các CSDL/Framework khác

| Tiêu chí              | CockroachDB             | PostgreSQL           | MongoDB              | Redis              |
|-----------------------|--------------------------|-----------------------|-----------------------|--------------------|
| Phân tán dữ liệu      | ✅ Có                    | ❌ Không              | ✅ Có                 | ✅ Có              |
| Hỗ trợ SQL            | ✅ PostgreSQL-compatible | ✅ Chuẩn SQL          | ❌ NoSQL             | ❌ (Key-Value)     |
| Nhất quán mạnh        | ✅ Có                    | ✅ Có                 | ❌ (eventual)        | ⚠️ Tùy cấu hình    |
| Khả năng mở rộng      | ✅ Tốt                   | ❌ Hạn chế            | ✅ Tốt                | ✅ Tốt             |
| Độ thân thiện         | ⚠️ Vừa phải              | ✅ Dễ                 | ✅ Dễ                | ✅ Dễ              |

### ➕ Ưu điểm

- Phân tán thực sự (true distributed system).
- Hỗ trợ SQL, dễ tiếp cận với lập trình viên quen dùng RDBMS.
- Tự động cân bằng tải, phục hồi lỗi.
- Có dịch vụ Cloud và Docker dễ triển khai.

### ➖ Nhược điểm

- Triển khai cụm nhiều node thủ công khá phức tạp.
- Tốn tài nguyên (RAM, CPU) hơn DB truyền thống.
- Không phù hợp cho bài toán nhỏ hoặc hệ thống ít người dùng.

### 🚀 Ứng dụng thực tế

- Ngân hàng, tài chính, giao dịch thời gian thực.
- Mạng xã hội, nền tảng blog đông người dùng.
- Hệ thống e-commerce toàn cầu cần truy cập dữ liệu nhanh ở nhiều khu vực.

---

## 🧩 4. Ứng dụng CockroachDB trong đề tài blog

- **Lưu trữ bài viết và bình luận phân tán** trên nhiều node → đảm bảo không mất dữ liệu nếu có node gặp sự cố.
- **Tính năng bình luận thời gian thực** qua WebSocket sẽ tương tác với backend để cập nhật bình luận mới lên DB ngay khi người dùng gửi.
- **Dễ dàng mở rộng** khi hệ thống có nhiều người dùng hoặc nhiều bài viết hơn.

---

## 📆 5. Kế hoạch dự kiến giữa kỳ

| Tuần | Công việc                                                                 |
|------|---------------------------------------------------------------------------|
| 1    | Cài đặt và cấu hình cụm CockroachDB nhiều node bằng Docker               |
| 2    | Thiết kế CSDL: bảng User, Post, Comment                                  |
| 3    | Xây dựng giao diện frontend với SvelteKit                                |
| 4    | Kết nối frontend ↔ backend ↔ CockroachDB, viết API cơ bản                |
| 5    | Triển khai WebSocket cho bình luận thời gian thực                        |
| 6    | Mô phỏng lỗi node, kiểm thử tính nhất quán và khả năng phục hồi          |
| 7    | Viết báo cáo giữa kỳ, hoàn thiện phần chức năng cần demo                 |

### 🎯 Chức năng hoàn thành trong giữa kỳ:

- CRUD bài viết.
- Giao diện viết và xem bài.
- Kết nối hệ thống với CockroachDB phân tán.
- Xử lý thêm/lấy bình luận (có thể kết nối sẵn WebSocket nếu kịp).
