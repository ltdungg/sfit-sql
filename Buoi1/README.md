# BUỔI 1 - GIỚI THIỆU VỀ SQL VÀ CÁC CÂU LỆNH CƠ BẢN

## CÀI ĐẶT MÔI TRƯỜNG PostgreSQL

Cài đặt PostgreSQL trên máy tính của bạn. 
- Bạn có thể tải xuống từ trang chính thức của PostgreSQL: [PostgreSQL Downloads](https://www.postgresql.org/download/).
- Làm theo hướng dẫn cài đặt cho hệ điều hành của bạn (Windows, macOS, Linux).
- Trong quá trình cài đặt, hãy nhớ ghi lại tên người dùng và mật khẩu bạn đã tạo cho tài khoản quản trị viên (thường là `postgres`).

## SETUP DỮ LIỆU MẪU CHO BUỔI HỌC
- Dữ liệu mẫu sẽ được sử dụng trong buổi học này là một cơ sở dữ liệu về cửa hàng bánh ngọt (bakery). Có các bảng như sau:
![image](../static/images/bakery-db.png)
- Mở PGAdmin, trong giao diện chính, tạo một cơ sở dữ liệu mới với tên `bakery`.
- Chọn cơ sở dữ liệu `bakery` vừa tạo, sau đó mở tab "Query Tool".
- Sao chép và dán đoạn mã SQL của file `bakery-db.sql` vào cửa sổ truy vấn. Link file: [bakery-db.sql](./bakery-db.sql)

## SCRIPT THỰC HÀNH
1. SELECT ... FROM cơ bản:
```sql
-- Lấy tất cả các sản phẩm
SELECT * FROM products;

-- Lấy danh sách tên và giá bán của sản phẩm
SELECT product_name, sale_price FROM products;

-- Lấy tên sản phẩm, giá bán và giá sau khi giảm 10%
SELECT 
    product_name, 
    sale_price, 
    sale_price * 0.9 AS discounted_price
FROM 
    products;
```

2. WHERE để lọc dữ liệu:
```sql
-- Lấy các sản phẩm còn nhiều hơn 50 đơn vị trong kho
SELECT * FROM products WHERE units_in_stock > 50;

-- Lấy thông tin khách hàng ở thành phố Scranton
SELECT * FROM customers WHERE city = 'Scranton';
```

3. SELECT với IN, BETWEEN
```sql
-- Lấy các sản phẩm có mã là 1001, 1003 hoặc 1005
SELECT * FROM products WHERE product_id IN (1001, 1003, 1005);

-- Lấy các đơn hàng có tổng tiền từ 10 đến 50
SELECT * FROM customer_orders WHERE order_total BETWEEN 10 AND 50;

-- Lấy các nhân viên thuộc phòng 'Bakery' hoặc 'Marketing'
SELECT * FROM employees WHERE department IN ('Bakery', 'Marketing');
```

4. SELECT với các toán tử logic
- Sử dụng AND, OR
```sql
-- Sản phẩm có giá từ $1 đến $3 VÀ còn hơn 100 sản phẩm trong kho
SELECT * 
FROM products 
WHERE sale_price BETWEEN 1 AND 3 
  AND units_in_stock > 100;

-- Khách hàng ở Texas (TX) HOẶC Pennsylvania (PA)
SELECT *
FROM customers
WHERE state = 'TX' OR state = 'PA';
```
- Sử dụng NOT
```sql
-- Khách hàng KHÔNG đến từ Texas (TX) hoặc California (CA)
SELECT *
FROM customers
WHERE state NOT IN ('TX', 'CA');

-- Sản phẩm KHÔNG có từ 'Cookie' trong tên
SELECT *
FROM products
WHERE product_name NOT LIKE '%Cookie%';

```

5. SELECT với LIKE
```sql
-- Lấy các sản phẩm có tên bắt đầu bằng 'Ch'
SELECT * FROM products WHERE product_name LIKE 'Ch%';

-- Lấy khách hàng có họ kết thúc bằng 'son'
SELECT * FROM customers WHERE last_name LIKE '%son';

-- Lấy các sản phẩm có tên chứa từ 'Cake'
SELECT * FROM products WHERE product_name LIKE '%Cake%';
```

## BÀI TẬP
Link bài tập: [Bài tập Buổi 1](https://forms.gle/9JwXXDwZwhWeggxo9)