---
name: job-hunter
description: Kỹ năng tìm kiếm và tổng hợp các tin tuyển dụng IT tại Việt Nam. Sử dụng tool search_web để fetch dữ liệu từ ITviec, TopDev, LinkedIn VN, v.v.
allowed-tools: search_web, write_to_file
version: 1.0
priority: MEDIUM
---

# Job Hunter (Thợ săn việc làm VN)

Kỹ năng này giúp tự động hóa việc tìm kiếm các cơ hội việc làm mới nhất dựa trên Tech Stack của bạn.

## 1. Công thức tìm kiếm (Search Queries)

Để có kết quả tốt nhất, Agent nên sử dụng các query sau:
- `site:itviec.com [tech_stack] [location]`
- `site:topdev.vn [tech_stack] [location]`
- `jobs in Vietnam [tech_stack] "remote"`
- `[tech_stack] tuyển dụng tháng 4 2026`

## 2. Quy trình trích xuất thông tin

Khi nhận được kết quả tìm kiếm, hãy trích xuất các trường thông tin sau:
- **Công ty**: Tên công ty tuyển dụng.
- **Vị trí**: Senior/Middle/Junior + Tech.
- **Địa điểm**: Hà Nội, HCM, Đà Nẵng, Remote.
- **Mức lương**: (Nếu có).
- **Link**: Link gốc của tin tuyển dụng.

## 3. Quản lý Pipeline (kết hợp xlsx)

Bạn có thể yêu cầu:
- "Hãy lưu danh sách 10 job Backend Node.js vào file danh sách_job.xlsx" (Agent sẽ dùng skill `xlsx` có sẵn để tạo file).

## Hướng dẫn cho Agent

1. **Gạn lọc Job**: Chỉ lấy những job được đăng trong vòng 7-14 ngày gần nhất để đảm bảo tin còn hiệu lực.
2. **Ưu tiên**: Ưu tiên các trang uy tín như ITviec và TopDev cho thị trường VN.

---

> **Lưu ý**: Đây là công cụ hỗ trợ tìm kiếm, bạn vẫn nên check kỹ lại tin tuyển dụng trước khi nộp CV.
