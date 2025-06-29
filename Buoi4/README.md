# CTE and Temporary Tables

## 1. Common Table Expressions (CTEs)

- Khái niệm: 
  - CTE là một tập kết quả tạm thời, được định nghĩa trong truy vấn bằng mệnh đề WITH, tồn tại trong phạm vi của truy vấn đó.
  - CTE giúp chia nhỏ truy vấn phức tạp thành các phần dễ đọc, dễ bảo trì, và có thể tái sử dụng trong cùng một truy vấn

```sql
WITH cte_name AS (
  SELECT ...
  FROM ...
  WHERE ...
)
SELECT ...
FROM cte_name
WHERE ...;
```
- Có thể định nghĩa nhiều CTE cùng lúc, mỗi CTE cách nhau bằng dấu phẩy.

- Thực hành:
```sql
-- Tính tổng số lượng bán ra của từng sản phẩm
WITH product_sales AS (
  SELECT p.product_id, p.product_name, SUM(oi.quantity) AS total_sold
  FROM products p
  JOIN ordered_items oi ON p.product_id = oi.product_id
  GROUP BY p.product_id, p.product_name
)
SELECT *
FROM product_sales
ORDER BY total_sold DESC;

--  Lọc sản phẩm bán chạy hơn mức trung bình
WITH product_sales AS (
  SELECT p.product_id, p.product_name, SUM(oi.quantity) AS total_sold
  FROM products p
  JOIN ordered_items oi ON p.product_id = oi.product_id
  GROUP BY p.product_id, p.product_name
),
avg_sales AS (
  SELECT AVG(total_sold) AS avg_total FROM product_sales
)
SELECT ps.product_name, ps.total_sold
FROM product_sales ps, avg_sales
WHERE ps.total_sold > avg_sales.avg_total;

--  xếp hạng sản phẩm bán chạy 
WITH product_sales AS (
  SELECT p.product_id, p.product_name, SUM(oi.quantity) AS total_sold
  FROM products p
  JOIN ordered_items oi ON p.product_id = oi.product_id
  GROUP BY p.product_id, p.product_name
),
ranked_sales AS (
  SELECT product_name, total_sold,
         RANK() OVER (ORDER BY total_sold DESC) AS sales_rank
  FROM product_sales
)
SELECT *
FROM ranked_sales
WHERE sales_rank <= 3;
```

## 2. CTE Nâng Cao với Recursive CTEs
- Recursive CTEs cho phép truy vấn dữ liệu theo cấu trúc đệ quy, thường dùng để xử lý cây hoặc đồ thị.
- Cấu trúc:
```sql
WITH RECURSIVE cte_name AS (
  -- Câu truy vấn cơ sở
  SELECT ...
  UNION ALL
  -- Câu truy vấn đệ quy
  SELECT ...
  FROM cte_name
  WHERE ...
)
```
- Thực hành:
```sql
-- Đếm từ 1 đến 10
WITH RECURSIVE nums(n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM nums WHERE n < 10
)
SELECT * FROM nums;

-- Tính giai thừa của một số 5 
WITH RECURSIVE factorial(n, fact) AS (
  SELECT 1, 1
  UNION ALL
  SELECT n+1, fact*(n+1) FROM factorial WHERE n < 5
)
SELECT fact FROM factorial WHERE n = 5;
```

## 3. Temporary Table (Bảng tạm thời)
- Khái niệm:
  - Temporary table là bảng chỉ tồn tại trong phiên làm việc (session) hoặc transaction hiện tại, tự động xóa khi phiên kết thúc.
  - Thường dùng để lưu trữ dữ liệu trung gian, batch processing, staging data, hoặc tối ưu hiệu năng truy vấn phức tạp.

```sql 
CREATE TEMP TABLE temp_table_name (
  column1 datatype,
  column2 datatype,
  ...
);

-- Or
CREATE TEMP TABLE temp_table_name AS
SELECT ...
```

- Thực hành:
- Cách xem bảng tạm thời trong PostgreSQL:
  - Cách 1: Sử dụng câu lệnh với `information_schema`:
  ```sql
    SELECT * FROM information_schema.tables
    WHERE table_type = 'LOCAL TEMPORARY';
  ```
  - Cách 2: Vào Settings -> Display -> Show System Objects
```sql
-- Tạo bảng tạm để lưu tổng số lượng bán ra của từng sản phẩm
CREATE TEMP TABLE temp_product_sales AS
SELECT product_id, SUM(quantity) AS total_sold
FROM ordered_items
GROUP BY product_id;

-- Sử dụng bảng tạm để phân tích sản phẩm bán chạy

SELECT p.product_name, t.total_sold
FROM temp_product_sales t
JOIN products p ON p.product_id = t.product_id
WHERE t.total_sold > 100
ORDER BY t.total_sold DESC;
```

## 4. So sánh CTE và Temporary Table
- Thời gian tồn tại: Temp Table (bảng tạm) có thể dùng nhiều lần trong một phiên làm việc; CTE (Common Table Expression) chỉ dùng được trong một truy vấn rồi tự mất đi
- Cách lưu trữ: Temp Table lưu dữ liệu trên ổ đĩa tạm của hệ thống, còn CTE chỉ lưu trong bộ nhớ khi chạy truy vấn, không tạo bảng thật
- Khi nào nên dùng: Temp Table phù hợp khi bạn cần xử lý dữ liệu lớn hoặc dùng lại dữ liệu nhiều lần; CTE phù hợp khi bạn chỉ cần kết quả tạm thời cho một truy vấn và muốn viết truy vấn cho dễ đọc
- Tính năng đặc biệt: Temp Table có thể thêm chỉ mục (index) để tăng tốc truy vấn; CTE có thể dùng cho truy vấn đệ quy (ví dụ: cây thư mục, cấp bậc), giúp viết truy vấn phức tạp dễ hiểu hơn
- Dễ đọc và bảo trì: CTE giúp chia nhỏ truy vấn dài thành các phần dễ hiểu hơn, thuận tiện cho việc kiểm tra và chỉnh sửa; Temp Table giống như một bảng thật, cần tạo và xóa thủ công nếu không dùng nữa

## Bài tập





