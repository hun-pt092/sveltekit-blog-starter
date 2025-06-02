---
title: "Ôn tập cuối kì hệ phân tán"
slug: "he-thong-phan-tan"
date: "2025-05-24"
coverImage: "/images/distribu-system.png"
categories: ["công nghệ", "hệ thống phân tán"]
---
**Tác giả:** Nguyễn Duy Hưng
## 1,So sánh Hệ thống Tập trung, Phân tán và Phi tập trung

## Hệ thống Tập trung (Centralized System)

###  Đặc điểm:
- Tất cả dữ liệu và quyết định được xử lý tại **một nút trung tâm duy nhất**.
- Các client chỉ gửi yêu cầu và nhận phản hồi.
- Dễ kiểm soát, bảo mật tập trung, nhưng có **điểm yếu duy nhất (single point of failure)**.

### Ví dụ:
- **Google Search**: Người dùng gửi truy vấn đến server trung tâm của Google.
- **Hệ thống ngân hàng truyền thống**: Dữ liệu và xử lý được quản lý tại trung tâm dữ liệu của ngân hàng.

###  Cách nhận biết:
- Có một điểm duy nhất điều khiển toàn bộ hệ thống.
- Nếu mất nút trung tâm → toàn hệ thống ngừng hoạt động.

---

## Hệ thống Phân tán (Distributed System)

###  Đặc điểm:
- Nhiều nút (nodes) **hợp tác xử lý và lưu trữ dữ liệu**.
- Dữ liệu có thể được sao chép và đồng bộ giữa các nút.
- Không có một điểm trung tâm duy nhất điều khiển tất cả.

###  Ví dụ:
- **Google File System (GFS)**, **Hadoop HDFS**: Dữ liệu được chia nhỏ và phân phối trên nhiều máy chủ.
- **CDN (Content Delivery Network)**: Nội dung web được phân phối qua nhiều máy chủ toàn cầu.

###  Cách nhận biết:
- Tác vụ và dữ liệu được phân chia và xử lý đồng thời tại nhiều nơi.
- Tăng khả năng chịu lỗi và mở rộng tốt.

---

##  Hệ thống Phi tập trung (Decentralized System)

###  Đặc điểm:
- Không có trung tâm điều khiển rõ ràng.
- Các nút hoạt động **độc lập** và **có quyền ngang nhau**, nhưng vẫn tương tác và chia sẻ thông tin.
- Là dạng đặc biệt của hệ thống phân tán, thiên về **quản trị ngang hàng (peer-to-peer)**.

###  Ví dụ:
- **Blockchain (Bitcoin, Ethereum)**: Các nút đều tham gia xác thực và lưu trữ giao dịch.
- **BitTorrent**: Chia sẻ tệp giữa người dùng không cần máy chủ trung tâm.

###  Cách nhận biết:
- Không có máy chủ trung tâm.
- Dữ liệu và quyết định được phân phối, xác nhận bởi nhiều nút ngang hàng.

---
## 2,Các đặc tính của hệ phân tán là gì?
Hệ phân tán có 4 đặc điểm đặc trưng sau:Chia sẻ tài nguyên,Tính trong suốt,Tính mở,Tính co giãn

### Chia sẻ tài nguyên

- **Kết nối tài nguyên**: Các tài nguyên được chia sẻ qua mạng.
- **Giảm chi phí**: Tận dụng tài nguyên sẵn có, giảm đầu tư hạ tầng mới.
- **Tăng tính sẵn sàng**: Nếu một tài nguyên bị lỗi, có thể dùng tài nguyên khác thay thế.
- **Hỗ trợ làm việc nhóm**: Nhiều người dùng có thể cùng truy cập và làm việc trên tài nguyên chung.
- **Tăng rủi ro về an toàn thông tin**: Do nhiều người truy cập, nguy cơ rò rỉ hoặc bị tấn công tăng lên.

---

### Tính trong suốt (Transparency)

- **Hệ thống là duy nhất với người sử dụng (NSD)**:
  - Giao diện giống nhau
  - Cách thức truy cập giống nhau
- **Trong suốt về quy mô và vị trí**: Người dùng không cần biết tài nguyên nằm ở đâu, quy mô bao nhiêu.
- **Che giấu tính phân tán** của hệ thống: Làm cho hệ thống trông như một hệ thống đơn nhất.
- **Mức độ trong suốt**:
  - Cần **cân bằng giữa hiệu năng và độ trong suốt**: quá trong suốt có thể làm giảm hiệu suất.

---

### Tính mở (Openness)

- **Cho phép các thành phần đến từ nhiều nhà sản xuất khác nhau**.
- **Cung cấp các dịch vụ theo đặc tả** về:
  - **Cú pháp**
  - **Ngữ nghĩa**
  → Gọi là **giao diện**.
- **Đặc tả giao diện** thường được mô tả bằng **ngôn ngữ tương tác dữ liệu (IDL)**.
- **Tính đầy đủ của đặc tả**:
  - Quá chi tiết: Gây **phụ thuộc vào cài đặt cụ thể**.
  - Không đủ chi tiết: Khi cài đặt phải bổ sung thêm thông tin.
- **Khả năng phối hợp (Interoperability)**: Hệ thống có thể hoạt động với nhiều nền tảng, giao thức.
- **Tính khả chuyển (Portability)**: Dễ dàng chuyển sang môi trường khác mà không cần sửa đổi lớn.
- **Tính mềm dẻo và mở rộng được** (*Flexibility, Extensibility*): Dễ nâng cấp, mở rộng, thay đổi thành phần.
- **Tách biệt chính sách và cơ chế**: Chính sách có thể thay đổi mà không cần thay đổi cơ chế thực thi.

---

### Tính co giãn (Scalability)

- **Về quy mô**: Số lượng người dùng và tài nguyên có thể thay đổi linh hoạt.
- **Không gian địa lý**: Có thể mở rộng ra nhiều khu vực địa lý khác nhau.
- **Tổ chức**:
  - Quy mô tổ chức thay đổi → Cần tổ chức hệ thống thành **các miền (domains)** để quản lý hiệu quả.

#### Co giãn theo số lượng:

- **Mô hình tập trung**:
  - **Dịch vụ**: Có thể gây nghẽn (*cổ chai*).
  - **Dữ liệu**: Vấn đề lưu trữ và xử lý dữ liệu lớn.
  - **Giải thuật**: Vấn đề xử lý đầu vào/ra tăng nhanh.

- **Mô hình không tập trung**:
  - Phức tạp hơn.
  - Gặp vấn đề về **bảo mật** và **riêng tư**.
  - Các **quyết định mang tính cục bộ**, khó phối hợp tổng thể.
  - **Khó phát hiện lỗi** trong hệ thống.

---

## 3, Nút Chủ trong Hệ Thống Phân Tán

###  Mục đích của nút chủ (Master Node)

**Điều phối tài nguyên**
   - Phân công công việc hoặc dữ liệu cho các nút con (worker/slave nodes).
   - Quản lý quá trình phân chia, sao lưu hoặc đồng bộ dữ liệu.

**Quản lý trạng thái hệ thống**
   - Theo dõi trạng thái các nút khác (alive, fail, idle...).
   - Duy trì thông tin về cấu trúc hệ thống (ai đang xử lý gì, dữ liệu ở đâu...).

**Xử lý truy vấn hoặc yêu cầu của người dùng**
   - Nhận yêu cầu từ client và định tuyến đến các nút phù hợp.

**Duy trì nhất quán và đồng bộ dữ liệu** (trong một số hệ thống)
   - Ví dụ: Trong HDFS (Hadoop Distributed File System), `NameNode` là nút chủ – giữ metadata của toàn hệ thống.

---

###  Điều gì xảy ra nếu nút chủ gặp sự cố?

####  Trường hợp 1: **Không có cơ chế dự phòng (Single Point of Failure)**

- **Toàn bộ hệ thống có thể ngừng hoạt động** do không có ai điều phối.
- **Dữ liệu có thể bị mất hoặc không truy cập được** nếu metadata chỉ nằm ở nút chủ.
- Hệ thống trở nên **không ổn định hoặc mất kiểm soát**.

> *Ví dụ*: Trong Hadoop cũ (trước Hadoop 2), nếu `NameNode` chết, toàn bộ hệ thống HDFS ngừng hoạt động.

###  Trường hợp 2: **Có cơ chế dự phòng (Redundancy / High Availability)**

- Một nút chủ khác (*standby master / backup*) sẽ **tự động thay thế** nút chủ bị lỗi.
- Hệ thống tiếp tục hoạt động bình thường hoặc chỉ gián đoạn trong thời gian ngắn.
- *Ví dụ*: Hadoop mới hỗ trợ `Active NameNode + Standby NameNode` để tránh downtime.

---
## 4,Giao tiếp qua Gossip Protocol trong Mạng Không Gian

## Vì sao các máy sử dụng Gossip Protocol?

Trong một mạng phân tán quy mô lớn (đặc biệt như mạng không gian, mạng cảm biến, mạng ngang hàng...), các máy thường giao tiếp với nhau qua **Gossip Protocol** thay vì:

- Gửi thông tin đến **tất cả các máy khác** (broadcast/flooding), hoặc
- Gửi thông tin về **một nút trung tâm**

Lý do:

### Khả năng mở rộng cao (Scalability)

- Gossip hoạt động hiệu quả trong mạng có nhiều nút.
- Mỗi nút chỉ gửi thông tin đến một vài nút lân cận → giảm tải mạng.
- Broadcast toàn mạng sẽ gây tốn băng thông và tài nguyên khi mạng lớn.

### Chịu lỗi tốt (Fault-tolerance)

- Không phụ thuộc vào một nút trung tâm → tránh điểm lỗi duy nhất.
- Nếu một số nút mất kết nối, thông tin vẫn lan truyền được qua các nút khác.

### Lan truyền thông tin hiệu quả

- Giao thức hoạt động như "lời đồn": mỗi nút truyền tin cho vài nút khác, lặp lại nhiều vòng.
- Đảm bảo thông tin được lan truyền tới hầu hết các nút với chi phí thấp.

### Giảm tiêu tốn tài nguyên

- Đặc biệt phù hợp với mạng có hạn chế về năng lượng hoặc năng lực xử lý.
- Không cần mỗi nút phải biết toàn bộ cấu trúc mạng.

### Không cần quản lý tập trung

- Không cần bảng định tuyến phức tạp hoặc máy chủ trung tâm.
- Phù hợp với các mạng phân tán hoặc mạng có các nút di động (như vệ tinh).

---

## Vì sao không dùng broadcast toàn mạng?

- Rất tốn băng thông.
- Gây trùng lặp dữ liệu.
- Không hiệu quả trong mạng lớn.
- Có thể gây tắc nghẽn hoặc sập hệ thống.

## Vì sao không gửi về nút trung tâm?

- Gây ra điểm yếu duy nhất: nếu nút trung tâm bị lỗi, toàn hệ thống có thể tê liệt.
- Dễ gây tắc nghẽn tại trung tâm.
- Không phù hợp với mạng có quy mô rộng hoặc các nút thường xuyên thay đổi.

---


## 9,Sharding là 
[Sharding](https://viblo.asia/p/database-sharding-la-gi-Az45boQVKxY)  
Sharding là một mẫu kiến trúc cơ sở dữ liệu liên quan đến phân vùng ngang - thực tế tách một hàng bảng Bảng thành nhiều bảng khác nhau, được gọi là partitions. Mỗi partitions có cùng schema và cột, nhưng cũng có các hàng hoàn toàn khác nhau. Tương tự, dữ liệu được giữ trong mỗi partitions là duy nhất và độc lập với dữ liệu được giữ trong các partitions khác.

Có thể hữu ích khi so sánh suy nghĩ về phân vùng ngang theo cách nó liên quan đến phân vùng dọc. Trong một bảng được phân vùng theo chiều dọc, toàn bộ các cột được tách ra và đưa vào các bảng mới, riêng biệt. Dữ liệu được giữ trong một phân vùng dọc độc lập với dữ liệu trong tất cả các phân vùng khác và mỗi dữ liệu chứa cả các hàng và cột riêng biệt.
