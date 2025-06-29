# BUỔI 2 - SQL

# Giới thiệu về GROUP BY
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

## Kết hợp GROUP BY
### Lọc kết quả nhóm với HAVING
- **HAVING** là mệnh đề dùng để lọc các nhóm dữ liệu sau khi đã áp dụng GROUP BY. Nó cho phép bạn đặt điều kiện trên kết quả của các hàm tổng hợp.

```sql
-- Lấy danh sách sản phẩm có tổng số lượng đặt hàng lớn hơn 100:
SELECT product_id, SUM(quantity) AS total_quantity
FROM ordered_items
GROUP BY product_id
HAVING SUM(quantity) > 100;
```

# GIỚI THIỆU VỀ JOIN
1. Khái niệm tổng quan về JOIN
- **JOIN** là lệnh dùng để kết hợp dữ liệu từ 2 hay nhiều bảng dựa trên mối quan hệ giữa các cột chung (thường là khóa chính và khóa ngoại).

- Mục đích: Giúp bạn truy xuất dữ liệu liên quan từ nhiều bảng, tạo ra báo cáo tổng hợp, phân tích đa chiều.

### Bảng tổng hợp các loại JOIN trong SQL

| Loại JOIN        | Ý nghĩa                                                                 | Kết quả trả về                                                                                     | Ví dụ cơ bản                                                                                   |
|------------------|------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| INNER JOIN       | Chỉ lấy các dòng có giá trị chung ở cả hai bảng                         | Chỉ dữ liệu khớp ở cả hai bảng                                                                     | `SELECT ... FROM A INNER JOIN B ON A.key = B.key;`                                            |
| LEFT JOIN        | Lấy tất cả dòng ở bảng bên trái, nếu không khớp thì cột bên phải là NULL| Tất cả dòng bảng trái, bảng phải có thể NULL                                                       | `SELECT ... FROM A LEFT JOIN B ON A.key = B.key;`                                             |
| RIGHT JOIN       | Lấy tất cả dòng ở bảng bên phải, nếu không khớp thì cột bên trái là NULL| Tất cả dòng bảng phải, bảng trái có thể NULL                                                       | `SELECT ... FROM A RIGHT JOIN B ON A.key = B.key;`                                            |
| FULL OUTER JOIN  | Lấy tất cả dòng ở cả hai bảng, nếu không khớp thì giá trị là NULL       | Tất cả dòng của cả hai bảng, chỗ không khớp là NULL                                                | `SELECT ... FROM A FULL OUTER JOIN B ON A.key = B.key;`                                       |
| CROSS JOIN       | Kết hợp tất cả các dòng của hai bảng (tích Descartes)                   | Số dòng = dòng bảng 1 × dòng bảng 2                                                                | `SELECT ... FROM A CROSS JOIN B;`                                                             |
| SELF JOIN        | Kết nối một bảng với chính nó                                           | Kết quả như JOIN hai bảng, nhưng thực hiện trên cùng một bảng                                      | `SELECT ... FROM A a1 LEFT JOIN A a2 ON a1.manager_id = a2.employee_id;`                      |

### Ví dụ minh họa JOIN
```sql
-- INNER JOIN: Lấy danh sách các đơn hàng (ordered_items) kèm tên sản phẩm (products):
SELECT oi.order_id, oi.product_id, p.product_name, oi.quantity
FROM ordered_items oi
INNER JOIN products p ON oi.product_id = p.product_id;

-- LEFT JOIN: Lấy tất cả đơn hàng và tên sản phẩm (kể cả trường hợp sản phẩm đã bị xóa khỏi bảng products):
SELECT oi.order_id, oi.product_id, p.product_name, oi.quantity
FROM ordered_items oi
LEFT JOIN products p ON oi.product_id = p.product_id;

-- RIGHT JOIN: Lấy tất cả sản phẩm, kèm số lượng đã đặt (nếu có):
SELECT p.product_id, p.product_name, oi.quantity
FROM products p
RIGHT JOIN ordered_items oi ON oi.product_id = p.product_id;

-- FULL JOIN: Lấy tất cả sản phẩm và tất cả đơn hàng, ghép theo product_id
SELECT p.product_id, p.product_name, oi.order_id, oi.quantity
FROM products p
FULL JOIN ordered_items oi ON oi.product_id = p.product_id;

-- CROSS JOIN: Tạo tổ hợp tất cả sản phẩm với tất cả nhà cung cấp
SELECT p.product_name, s.name AS supplier_name
FROM products p
CROSS JOIN suppliers s;

-- SELF JOIN: Liệt kê mọi cặp nhân viên làm cùng một phòng ban (department), giúp bạn biết ai là đồng nghiệp trực tiếp của ai trong từng phòng

SELECT
  e1.first_name || ' ' || e1.last_name AS employee1,
  e2.first_name || ' ' || e2.last_name AS employee2,
  e1.department
FROM employees e1
INNER JOIN employees e2
  ON e1.employee_id <> e2.employee_id
  AND e1.department = e2.department
ORDER BY e1.department, employee1, employee2;

-- NATURAL JOIN: Tự động kết nối các cột có cùng tên giữa hai bảng 
-- Không cần chỉ định điều kiện kết nối, nhưng chỉ sử dụng khi chắc chắn các cột có cùng tên và kiểu dữ liệu
-- Có thể dùng NATURAL kết hợp với các loại JOIN khác như INNER, LEFT, RIGHT, FULL

SELECT *
FROM customer_orders
NATURAL JOIN customers;

```

### Nâng cao hơn với JOIN
- **JOIN nhiều bảng:** Lấy danh sách đơn hàng, tên khách hàng và tên sản phẩm:
```sql
SELECT co.order_id, c.first_name || ' ' || c.last_name AS customer_name, p.product_name, co.order_total
FROM customer_orders co
JOIN customers c ON co.customer_id = c.customer_id
JOIN products p ON co.product_id = p.product_id;
```

- **JOIN với điều kiện lọc:** Lấy các đơn hàng đã giao thành công (status = 3) kèm tên shipper
```sql
SELECT oi.order_id, p.product_name, s.name AS shipper_name, oi.quantity
FROM ordered_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN suppliers s ON oi.shipper_id = s.supplier_id
WHERE oi.status = 3;
```
- **Sử dụng USING keyword**:
```sql
-- Lấy tất cả đơn hàng và tên sản phẩm
SELECT oi.order_id, oi.product_id, p.product_name, oi.quantity
FROM ordered_items oi
JOIN products p USING(product_id);
```

### Lưu ý khi sử dụng JOIN
- **Chọn loại JOIN phù hợp:** Tùy vào yêu cầu báo cáo mà chọn INNER, LEFT, RIGHT hay FULL JOIN.
- **Sử dụng alias:** Đặt tên ngắn gọn cho bảng để dễ đọc và viết câu truy vấn.
- **Kiểm tra hiệu suất:** JOIN nhiều bảng có thể làm chậm truy vấn, cần kiểm tra và tối ưu hóa nếu cần.
- **Sử dụng ON và USING:** Chọn cách kết nối phù hợp giữa các bảng, ON cho phép điều kiện phức tạp hơn, trong khi USING đơn giản hơn khi cột có cùng tên.
- **Tránh CROSS JOIN không cần thiết:** CROSS JOIN tạo ra tất cả tổ hợp có thể, có thể dẫn đến kết quả rất lớn và không mong muốn.
- **Sử dụng GROUP BY với JOIN:** Khi cần tổng hợp dữ liệu sau khi JOIN, có thể kết hợp GROUP BY để nhóm kết quả theo các cột mong muốn.
```sql
-- Số đơn hàng của từng khách hàng, sắp xếp theo số đơn giảm dần
SELECT
  c.first_name || ' ' || c.last_name AS customer_name,
  COUNT(co.order_id) AS order_count
FROM customer_orders co
JOIN customers c ON co.customer_id = c.customer_id
GROUP BY customer_name
ORDER BY order_count DESC;

-- Tổng doanh thu theo từng thành phố, sắp xếp theo doanh thu giảm dần
SELECT
  c.city,
  SUM(co.order_total) AS total_sales
FROM customer_orders co
JOIN customers c ON co.customer_id = c.customer_id
GROUP BY c.city
ORDER BY total_sales DESC;
```

- **Kiểm tra NULL:** Khi sử dụng LEFT JOIN hoặc RIGHT JOIN, cần chú ý đến các giá trị NULL trong kết quả, vì chúng có thể ảnh hưởng đến các phép toán tổng hợp hoặc điều kiện lọc.
- **Sử dụng DISTINCT khi cần:** Nếu kết quả JOIN có thể tạo ra các dòng trùng lặp, có thể sử dụng DISTINCT để loại bỏ chúng.

## UNION
- UNION dùng để kết hợp kết quả của hai hoặc nhiều truy vấn SELECT thành một tập kết quả duy nhất, loại bỏ các dòng trùng lặp.
- UNION ALL cũng kết hợp kết quả nhiều truy vấn SELECT, nhưng giữ lại tất cả các dòng, bao gồm cả dòng trùng lặp.
- Cả hai đều yêu cầu các truy vấn SELECT phải có cùng số lượng cột và kiểu dữ liệu tương thích.

```sql
SELECT column1, column2 FROM table1
UNION [ALL]
SELECT column1, column2 FROM table2;
```

### Ví dụ sử dụng UNION
```sql
-- Lấy danh sách những có lượng tip cao và số lượng khách hàng chi tiêu nhiều
SELECT first_name, last_name, 'good tipper' AS type
FROM customers c
JOIN customer_orders co ON c.customer_id = co.customer_id
WHERE tip::FLOAT > 3

UNION

SELECT first_name, last_name, 'high spender' AS type
FROM customers
WHERE total_money_spent > 1000;
```

## Bài tập
- Câu 1: Liệt kê tên sản phẩm và tổng số lượng đã bán của từng sản phẩm, sắp xếp giảm dần theo số lượng bán.
- Câu 2: Liệt kê tên khách hàng và số đơn hàng của họ nếu họ đã mua nhiều hơn 1 đơn.
- Câu 3: Liệt kê tên nhà cung cấp và tổng doanh thu từ các đơn hàng đã được giao thành công (status = 3).