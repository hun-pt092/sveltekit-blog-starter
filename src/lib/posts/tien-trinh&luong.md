---
title: "Tiến trình & Luồng"
slug: "he-thong-phan-tan"
date: "2025-04-28"
excerpt: "Nói về hiệu năng máy tính của tôi, các bài toán sử dụng đa luồng đa tiến trình."
coverImage: "/images/distribu-system.png"
coverWidth: 1200
coverHeight: 630
categories: ["công nghệ", "hệ thống phân tán"]
---
# Tiến Trình & Luồng

## 1. Hiệu năng máy tính hiện tại

**CPU:** Intel Core i3-7020U @ 2.30GHz  
- **Cores:** 2 | **Logical Processors:** 4 (nhờ Hyper-Threading)  
- **Threads hiện tại:** ~2890 | **Processes:** 293  
- **Utilization:** ~27% | **Speed:** 1.19 GHz

**RAM:** 19.9 GB (Sử dụng 11.0 GB - 55%)  
**GPU:** Intel HD Graphics 620 (Sử dụng 10%)  
**Disk:** SSD

**Nhận định:**  
- Hệ thống tốt cho lập trình, học hỏi, đa nhiệm.  
- Số luồng/tiến trình lớn nhờ context switching.  
- GPU và CPU đủ dùng cho các tác vụ nhẹ.

---

## 2. 12 Bài toán phổ biến trong CNTT có sử dụng đa luồng/đa tiến trình

| STT | Bài toán                  | Dùng Thread hay Process | Giải thích                              |
|-----|---------------------------|--------------------------|------------------------------------------|
| 1   | Web server                | Thread-per-request       | Mỗi request sinh ra luồng                |
| 2   | App chat                  | Thread-per-connection    | Mỗi user sinh ra luồng                   |
| 3   | Game Engine               | Đa luồng                 | Luồng AI, luồng physics                  |
| 4   | Video Rendering           | Process                  | Mỗi tình huống render độc lập            |
| 5   | Download Manager          | Đa tiến trình            | Mỗi file tải về là 1 process             |
| 6   | Compiler                  | Đa tiến trình            | Biên dịch song song các file             |
| 7   | Web scraping              | Đa luồng                 | Nhiều thread crawling song song          |
| 8   | Machine learning training | Multiprocess             | Phân chia data cho các process           |
| 9   | Video streaming server    | Đa luồng                 | Stream, handle comment, cache...         |
| 10  | IDE (VS Code, IntelliJ)   | Đa luồng                 | UI, biên dịch, syntax check              |
| 11  | OS Scheduler              | Kernel threads           | Quản lý tiến trình, task                 |
| 12  | CI/CD Pipeline            | Multiprocess             | Mỗi bước build/test là process           |

---

## 3. Khi nào dùng Thread, khi nào dùng Process?

![Ảnh phân tán](/static/images/distribu-system.png)


| Trường hợp                 | Thread                        | Process                         | Cả hai                                      |
|---------------------------|-------------------------------|----------------------------------|----------------------------------------------|
| Chia sẻ bộ nhớ chung      | ✔️                             | ❌                               | -                                            |
| Tác vụ song song nhẹ      | ✔️                             | ❌                               | -                                            |
| Tách biệt, an toàn        | ❌                             | ✔️                               | -                                            |
| Tính toán nặng (CPU bound)| ❌ (Python GIL giới hạn)       | ✔️                               | -                                            |
| Web server                | ✔️ (Thread-per-request)        | -                                | ✔️ Thread pool + Process nếu cần mở rộng     |
| AI Training (PyTorch)     | -                             | ✔️ (Data/Model Parallelism)      | ✔️ (Hybrid strategy dùng cả hai)             |

---

## 4. ChatGPT được đào tạo bằng Distributed System như thế nào?

- **Mô hình GPT (như ChatGPT)** được huấn luyện bằng hàng ngàn GPU kết nối qua mạng tốc độ cao.
- Áp dụng kỹ thuật **Model Parallelism** và **Data Parallelism**.
- Sử dụng các framework và công cụ như:
  - `DeepSpeed`, `Megatron-LM`
  - `PyTorch Distributed`, `NCCL`, `RDMA`
  - `Tensor/Model/Optimizer Parallelism`

### Tài liệu tham khảo:

- [HuggingFace Blog - LLM Training](https://huggingface.co/blog/llm-training)  
- [NVIDIA Developer - Scaling GPT-3](https://developer.nvidia.com/blog/scaling-gpt-3-training-on-ams/)  
- [Megatron-LM Paper (PDF)](https://arxiv.org/pdf/2104.04473.pdf)

---
