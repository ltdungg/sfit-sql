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
### Ví dụ: Ghi log khi giá sản phẩm thay đổi
- Trigger function là một hàm đặc biệt trả về kiểu trigger, không nhận tham số, sử dụng các biến đặc biệt như NEW (dữ liệu mới), OLD (dữ liệu cũ), TG_OP (loại thao tác), v.v.

Bước 1: Tạo Trigger Function
```sql
-- Tạo bảng log
CREATE TABLE product_price_log (
    log_id SERIAL PRIMARY KEY,
    product_id INTEGER,
    old_price NUMERIC(6,2),
    new_price NUMERIC(6,2),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- Tạo trigger function
CREATE OR REPLACE FUNCTION log_price_change()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.sale_price <> OLD.sale_price THEN
        INSERT INTO product_price_log(product_id, old_price, new_price)
        VALUES (NEW.product_id, OLD.sale_price, NEW.sale_price);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```
Bước 2: Tạo Trigger và gắn vào bảng
```sql
CREATE TRIGGER trg_product_price_update
AFTER UPDATE ON products
FOR EACH ROW
EXECUTE FUNCTION log_price_change();
```

Bước 3: Kiểm tra Trigger

### Ví dụ: Kiểm tra tồn kho trước khi đặt hàng
Bước 1: Tạo Trigger Function
```sql
CREATE OR REPLACE FUNCTION check_stock_before_order()
RETURNS TRIGGER AS $$
DECLARE
    stock INTEGER;
BEGIN
    SELECT units_in_stock INTO stock FROM products WHERE product_id = NEW.product_id;
    IF stock < NEW.quantity THEN
        RAISE EXCEPTION 'Not enough stock for product %', NEW.product_id;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

Bước 2: Tạo Trigger và gắn vào bảng
```sql
CREATE TRIGGER trg_check_stock_before_order
BEFORE INSERT ON ordered_items
FOR EACH ROW
EXECUTE FUNCTION check_stock_before_order();
```

Bước 3: Kiểm tra Trigger
```sql
SELECT * FROM products;
SELECT * FROM ordered_items;

INSERT INTO ordered_items VALUES
(100, 1001, 1, 500, 2, CURRENT_DATE::DATE, 1);
```
