# 📁 Experiences Manager

Hệ thống quản lý và bóc tách kinh nghiệm dự án cho CV & Phỏng vấn.

## 🔄 Workflow đề xuất:

1. **Khởi tạo:** Tạo thư mục cho dự án mới: `mkdir -p experiences/[Company]/[Project]`.
2. **Clone:** Di chuyển vào thư mục đó và clone code vào folder `.clone/`: 
   `git clone [URL] .clone/`
3. **Review:** 
   - Mở code trong `.clone/` để ôn lại.
   - Copy nội dung từ `TEMPLATE.md` vào `review.md` cùng cấp với `.clone/`.
4. **Bóc tách:** Sử dụng AI để phân tích code trong `.clone/` và điền vào `review.md` theo chuẩn STAR.
5. **Dọn dẹp:** Xóa folder `.clone/` để tiết kiệm bộ nhớ: `rm -rf .clone/`.

## 💡 Lưu ý:
- Folder `.clone/` đã được cấu hình trong `.gitignore` để không bị commit lên git nếu bạn dùng hệ thống này như một repo cá nhân.
- Hãy lưu các đoạn code quan trọng vào phần **Key Snippets** trong `review.md` trước khi xóa code.
