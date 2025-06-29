# Stored Procedure

## Giới thiệu về Stored Procedure
- Stored Procedure (Thủ tục lưu trữ), thường được gọi tắt là Procedure hoặc SP, là một tập hợp các lệnh SQL được lưu trữ trong cơ sở dữ liệu. Nó được thiết kế để thực hiện một hoặc nhiều thao tác logic phức tạp, có thể được gọi (thực thi) nhiều lần từ các ứng dụng khác nhau hoặc từ các script SQL khác.
- Trong PostgreSQL, việc hỗ trợ Stored Procedure (với khả năng quản lý giao dịch - transaction control) đã được bổ sung đầy đủ từ phiên bản PostgreSQL 11 trở đi. Trước đó, PostgreSQL chủ yếu sử dụng Functions (Hàm), đôi khi cũng được gọi chung là "stored procedures" nhưng có một số khác biệt quan trọng.

## Tại sao sử dụng Stored Procedure?
- **Tái sử dụng mã (Code Reusability):** Viết một lần, sử dụng nhiều lần. Điều này giúp giảm thiểu việc viết lại cùng một đoạn mã SQL.
- **Hiệu suất (Performance):** Khi một stored procedure được gọi lần đầu tiên, PostgreSQL sẽ biên dịch (compile) nó. Các lần gọi sau đó sẽ sử dụng phiên bản đã biên dịch, giúp thực thi nhanh hơn so với việc gửi các câu lệnh SQL riêng lẻ mỗi lần. Ngoài ra, việc giảm lượng dữ liệu truyền qua mạng (chỉ truyền tên procedure và các tham số) cũng cải thiện hiệu suất.
- **Bảo mật (Security):** Bạn có thể cấp quyền thực thi một procedure cho người dùng mà không cần cấp quyền trực tiếp truy cập vào các bảng cơ sở dữ liệu underlying. Điều này giúp kiểm soát quyền truy cập dữ liệu chặt chẽ hơn.
- **Tính toàn vẹn dữ liệu (Data Integrity):** Bằng cách gói gọn logic nghiệp vụ trong procedure, bạn đảm bảo rằng các thao tác dữ liệu phức tạp luôn được thực hiện một cách nhất quán, tuân thủ các quy tắc nghiệp vụ.
- **Giảm lưu lượng mạng (Reduced Network Traffic):** Thay vì gửi nhiều câu lệnh SQL riêng lẻ qua mạng, ứng dụng chỉ cần gửi một lệnh gọi procedure cùng với các tham số, giảm đáng kể lưu lượng mạng.
- **Quản lý giao dịch (Transaction Management):** Stored Procedure có thể bao gồm các lệnh bắt đầu, cam kết hoặc hủy giao dịch, giúp quản lý các thao tác dữ liệu phức tạp một cách an toàn và hiệu quả.

## Cú pháp cơ bản của Stored Procedure
```sql
CREATE PROCEDURE procedure_name (
    parameter1 data_type [ DEFAULT default_value ],
    parameter2 data_type [ DEFAULT default_value ],
    ...
)
LANGUAGE plpgsql -- Chỉ định ngôn ngữ mà procedure được viết. plpgsql là ngôn ngữ thủ tục mặc định và phổ biến nhất trong PostgreSQL.
AS $$ --  Là block mã của procedure. Bạn có thể sử dụng $tag$...$tag$ để tránh xung đột với các ký tự đặc biệt như dấu nháy đơn.
DECLARE -- Phần tùy chọn để khai báo các biến cục bộ (local variables) sử dụng trong procedure.
    -- Khai báo biến (tùy chọn)
BEGIN -- Chứa các lệnh SQL và logic xử lý chính của procedure.
    -- Các lệnh SQL và logic xử lý
    -- Có thể sử dụng COMMIT; và ROLLBACK; ở đây
END;
$$;
```

## Ví dụ về Stored Procedure
1. Ví du cơ bản
```sql
-- Thêm một sản phẩm mới vào bảng products
CREATE OR REPLACE PROCEDURE add_product(
    p_id INTEGER,
    p_name VARCHAR,
    p_stock INTEGER,
    p_price NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO products (product_id, product_name, units_in_stock, sale_price)
    VALUES (p_id, p_name, p_stock, p_price);
END;
$$;

-- Gọi thử:
CALL add_product(1011, 'Lemon Tart', 50, 3.75);
```

```sql
-- Cập nhật giá bán cho sản phẩm theo product_id
CREATE OR REPLACE PROCEDURE update_product_price(
    p_id INTEGER,
    new_price NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE products SET sale_price = new_price WHERE product_id = p_id;
END;
$$;

-- Gọi thử:
CALL update_product_price(1001, 1.99);
```
2. Ví dụ phức tạp hơn với transaction control
- Quản lý tồn kho sản phẩm với giao dịch nhập/xuất kho
```sql
CREATE TABLE product_transactions (
    transaction_id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL,
    transaction_type VARCHAR(10) NOT NULL, -- 'IN' hoặc 'OUT'
    quantity INTEGER NOT NULL,
    transaction_date DATE NOT NULL DEFAULT CURRENT_DATE,
    note TEXT
);

CREATE OR REPLACE PROCEDURE process_product_transaction(
    IN p_product_id INTEGER,
    IN p_type VARCHAR,
    IN p_quantity INTEGER,
    IN p_note TEXT DEFAULT NULL
)
LANGUAGE plpgsql
AS $$
DECLARE
    current_stock INTEGER;
BEGIN
    -- Kiểm tra tồn kho hiện tại
    SELECT units_in_stock INTO current_stock FROM products WHERE product_id = p_product_id FOR UPDATE;

    IF p_type = 'OUT' THEN
        IF current_stock IS NULL THEN
            RAISE EXCEPTION 'Product does not exist!';
        ELSIF current_stock < p_quantity THEN
            RAISE EXCEPTION 'Not enough stock! Transaction cancelled.';
        END IF;
        -- Trừ tồn kho
        UPDATE products SET units_in_stock = units_in_stock - p_quantity WHERE product_id = p_product_id;
    ELSIF p_type = 'IN' THEN
        -- Cộng tồn kho
        UPDATE products SET units_in_stock = units_in_stock + p_quantity WHERE product_id = p_product_id;
    ELSE
        RAISE EXCEPTION 'Invalid transaction type!';
    END IF;

    -- Ghi nhận giao dịch
    INSERT INTO product_transactions (product_id, transaction_type, quantity, note)
    VALUES (p_product_id, p_type, p_quantity, p_note);

END;
$$;

-- Gọi thử:
-- Giao dịch thành công
CALL process_product_transaction(1001, 'OUT', 10, 'Xuất kho bán lẻ');

-- Giao dịch thất bại (xuất kho quá số lượng tồn kho)
CALL process_product_transaction(1001, 'OUT', 10000, 'Xuất kho vượt quá tồn kho');

-- Giao dịch nhập kho
CALL process_product_transaction(1001, 'IN', 20, 'Nhập thêm hàng');
```



