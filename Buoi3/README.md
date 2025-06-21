# BUỔI 3 - SQL

## Giới thiệu về GROUP BY
**GROUP BY là gì?**
- GROUP BY là một mệnh đề trong SQL dùng để nhóm các dòng dữ liệu có giá trị giống nhau ở một hoặc nhiều cột lại với nhau, thường được sử dụng cùng các hàm tổng hợp như COUNT, SUM, AVG, MIN, MAX để tạo ra các báo cáo, thống kê tổng hợp theo từng nhóm.

**Cách hoạt động của GROUP BY**

- **Cơ bản:**

GROUP BY sẽ gom các dòng có cùng giá trị ở cột chỉ định thành một nhóm. Sau đó, bạn có thể áp dụng các hàm tổng hợp để tính toán trên từng nhóm này, ví dụ: tổng số lượng sản phẩm bán ra theo từng loại bánh.

- **Cú pháp chuẩn:**

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1;
```
Trong đó, column1 là cột dùng để nhóm dữ liệu, aggregate_function là hàm tổng hợp được áp dụng lên cột column2.
Trong đó, aggregate_function là các hàm tổng hợp như COUNT, SUM, AVG, MIN, MAX.

## Ví dụ sử dụng GROUP BY

```sql
-- Đếm số lượng sản phẩm theo từng mức giá:
SELECT sale_price, COUNT(*) AS product_count
FROM products
GROUP BY sale_price;


-- Tổng số lượng từng sản phẩm đã được đặt:
SELECT product_id, SUM(quantity) AS total_ordered
FROM ordered_items
GROUP BY product_id;

-- Đếm số lượng nhân viên theo phòng ban:
SELECT department, COUNT(*) AS num_employees
FROM employees
GROUP BY department;


-- Group By nhiều cột:
SELECT department, title, AVG(salary)
FROM employees
GROUP BY department, title;
```

## Một số lưu ý khi sử dụng GROUP BY
### 1. Sử dụng HAVING để lọc nhóm
- **HAVING** là mệnh đề dùng để lọc các nhóm dữ liệu sau khi đã áp dụng GROUP BY. Nó cho phép bạn đặt điều kiện trên kết quả của các hàm tổng hợp.

```sql
-- Lấy danh sách sản phẩm có tổng số lượng đặt hàng lớn hơn 100:
SELECT product_id, SUM(quantity) AS total_quantity
FROM ordered_items
GROUP BY product_id
HAVING SUM(quantity) > 100;
```
