# TRIGGER

## Trigger là gì?
Trigger là một đối tượng trong PostgreSQL tự động thực thi một hàm (function) khi xảy ra một sự kiện nhất định trên bảng hoặc view, như INSERT, UPDATE, DELETE, hoặc TRUNCATE. Trigger giúp bạn tự động hóa các thao tác như kiểm tra dữ liệu, ghi log, cập nhật bảng liên quan, hoặc thực thi các quy tắc nghiệp vụ mà không cần can thiệp từ ứng dụng bên ngoài

## Các loại Trigger phổ biến
- BEFORE trigger: Kích hoạt trước khi sự kiện xảy ra (ví dụ: kiểm tra, chỉnh sửa dữ liệu trước khi lưu).
- AFTER trigger: Kích hoạt sau khi sự kiện xảy ra (ví dụ: ghi log, cập nhật bảng phụ).
- INSTEAD OF trigger: Dùng cho view, thay thế hoàn toàn thao tác gốc. 
  - Điểm khác biệt cốt lõi của nó là nó ngăn chặn thao tác INSERT, UPDATE, hoặc DELETE dự định trên View và thay thế (instead of) bằng việc thực thi một hàm Trigger đã định nghĩa.

Theo phạm vi: 

- Row-level trigger: Kích hoạt cho từng dòng bị tác động (FOR EACH ROW).
- Statement-level trigger: Kích hoạt một lần cho mỗi câu lệnh SQL, bất kể bao nhiêu dòng bị tác động (FOR EACH STATEMENT)

## Cú pháp tạo Trigger
```sql
