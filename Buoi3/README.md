# BUỔI 3

## SUBQUERY

### Giới thiệu về Subquery
- **Subquery** (truy vấn con) là một truy vấn SQL nằm bên trong một truy vấn khác. 
- Cho phép thực hiện các thao tác phức tạp như lọc, tính toán, tổng hợp hoặc so sánh dữ liệu dựa trên kết quả trung gian mà không cần nhiều bước truy vấn riêng biệt

### Các loại Subquery phổ biến
1. **Subquery trong WHERE hoặc HAVING**
2. **Subquery trong SELECT**
3. **Subquery trong FROM**
    
### Subquery trong WHERE hoặc HAVING
1. **Sử dụng Subquery với WHERE**
```sql
-- Liệt kê tên và lương của các nhân viên có lương cao hơn mức lương trung bình toàn công ty.
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Có thể sử dụng IN - Liệt kê tên sản phẩm đã từng được đặt hàng.
SELECT product_name
FROM products
WHERE product_id IN (SELECT product_id FROM customer_orders);

-- Lấy sản phẩm có giá bán cao hơn bất kỳ giá bán nào đã từng được đặt hàng:
SELECT product_name, sale_price
FROM products
WHERE sale_price > ANY (SELECT order_total FROM customer_orders);

-- Lấy sản phẩm có giá bán thấp hơn tất cả các đơn hàng đã từng đặt
SELECT product_name, sale_price
FROM products
WHERE sale_price < ALL (SELECT order_total FROM customer_orders);

-- Lấy tên nhà cung cấp đã từng giao hàng thành công
SELECT name
FROM suppliers s
WHERE EXISTS (
  SELECT 1 FROM ordered_items oi WHERE oi.shipper_id = s.supplier_id AND oi.status = 3
);

-- Lấy tên sản phẩm chưa từng được review
SELECT product_name
FROM products p
WHERE NOT EXISTS (
  SELECT 1 FROM customer_orders_review r WHERE r.product_id = p.product_id
);

-- Lấy khách hàng đã từng đặt hàng nhưng chưa từng review
SELECT first_name, last_name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM customer_orders co WHERE co.customer_id = c.customer_id
)
AND NOT EXISTS (
  SELECT 1 FROM customer_orders_review r WHERE r.customer_id = c.customer_id
);

-- Lấy sản phẩm đã từng được đặt hàng bởi ít nhất 2 khách hàng khác nhau
SELECT product_name
FROM products p
WHERE EXISTS (
  SELECT 1 FROM customer_orders co WHERE co.product_id = p.product_id
  GROUP BY co.product_id
  HAVING COUNT(DISTINCT co.customer_id) >= 2
);
```
- Sử dụng Subquery với HAVING
```sql
-- Lấy các thành phố có tổng tiền chi tiêu khách hàng lớn hơn trung bình các thành phố
SELECT city, SUM(total_money_spent) AS total_spent
FROM customers
GROUP BY city
HAVING SUM(total_money_spent) > (
  SELECT AVG(city_total) FROM (
    SELECT SUM(total_money_spent) AS city_total FROM customers GROUP BY city
  ) AS city_avgs
);

-- Lấy các nhà cung cấp đã giao thành công nhiều hơn mức trung bình các nhà cung cấp
SELECT s.name, COUNT(*) AS delivered_count
FROM suppliers s
JOIN ordered_items oi ON s.supplier_id = oi.shipper_id
WHERE oi.status = 3
GROUP BY s.name
HAVING COUNT(*) > (
  SELECT AVG(delivered) FROM (
    SELECT COUNT(*) AS delivered
    FROM ordered_items
    WHERE status = 3
    GROUP BY shipper_id
  ) AS avg_delivered
);
```

2. **Subquery trong SELECT**
```sql
-- Hiển thị tổng số đơn hàng của từng khách hàng
SELECT 
  first_name, 
  last_name,
  (SELECT COUNT(*) FROM customer_orders co WHERE co.customer_id = c.customer_id) AS total_orders
FROM customers c;

-- Hiển thị ngày đặt hàng gần nhất của từng khách hàng
SELECT 
  first_name,
  last_name,
  (SELECT MAX(order_date) FROM customer_orders co WHERE co.customer_id = c.customer_id) AS last_order_date
FROM customers c;

-- Tính tổng số lượng đã bán, số đơn hàng, số khách hàng khác nhau đã mua từng sản phẩm, và số lượng tồn kho còn lại của mỗi sản phẩm
SELECT 
  product_name,
  units_in_stock,
  -- Tổng số lượng đã bán
  (SELECT COALESCE(SUM(quantity), 0) FROM ordered_items oi WHERE oi.product_id = p.product_id) AS total_sold,
  -- Số đơn hàng đã bán
  (SELECT COUNT(DISTINCT order_id) FROM ordered_items oi WHERE oi.product_id = p.product_id) AS order_count,
  -- Số khách hàng khác nhau đã mua
  (SELECT COUNT(DISTINCT co.customer_id) 
     FROM customer_orders co 
     WHERE co.product_id = p.product_id) AS distinct_customers
FROM products p;

-- Lấy thông tin khách hàng, tổng số đơn hàng, tổng tiền đã chi tiêu qua các đơn hàng, và số sản phẩm khác nhau đã mua
SELECT 
  first_name, last_name,
  -- Tổng số đơn hàng
  (SELECT COUNT(*) FROM customer_orders co WHERE co.customer_id = c.customer_id) AS total_orders,
  -- Tổng tiền đã chi tiêu qua các đơn hàng
  (SELECT COALESCE(SUM(order_total), 0) FROM customer_orders co WHERE co.customer_id = c.customer_id) AS total_spent,
  -- Số sản phẩm khác nhau đã mua
  (SELECT COUNT(DISTINCT product_id) FROM customer_orders co WHERE co.customer_id = c.customer_id) AS product_variety
FROM customers c;
```

3. **Subquery trong FROM**
- Là một SELECT nằm trong mệnh đề FROM, tạo ra một bảng tạm thời để truy vấn ngoài tiếp tục xử lý.
- Mỗi subquery trong FROM phải đặt bí danh (alias).

```sql 
SELECT ... 
FROM (SELECT ... FROM ...) AS alias
WHERE ...;
```

```sql
-- Lấy các sản phẩm có tổng số lượng đã bán lớn hơn 100
SELECT product_name, total_sold
FROM (
  SELECT 
    p.product_name,
    SUM(oi.quantity) AS total_sold
  FROM products p
  JOIN ordered_items oi ON p.product_id = oi.product_id
  GROUP BY p.product_name
) AS sales_summary
WHERE total_sold > 100;

-- Rút gọn các truy vấn phức tạp bằng cách sử dụng subquery trong FROM
SELECT * FROM (
	SELECT
	  product_name,
	  sale_price,
	  CASE
	    WHEN sale_price < 2 THEN 'Rẻ'
	    WHEN sale_price BETWEEN 2 AND 10 THEN
	      CASE
	        WHEN sale_price < 5 THEN 'Trung bình thấp'
	        ELSE 'Trung bình cao'
	      END
	    ELSE 'Đắt'
	  END AS price_category
	FROM products
) AS x
WHERE price_category = 'Rẻ'
;
```