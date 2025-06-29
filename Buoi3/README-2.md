# BUỔI 5

## WINDOW FUNCTION
### Giới thiệu về Window Function
- **Window function** là hàm thực hiện tính toán trên một tập các dòng liên quan đến dòng hiện tại mà không làm giảm số dòng kết quả (khác với aggregate).
- Hầu hết các Aggregate Functions(các hàm chúng ta vẫn hay sử dụng như COUNT, SUM, MAX, MIN, . . .) cũng có thể sử dụng như các Window Functions khi kết hợp với OVER()

### Cú pháp chung
```sql
function_name(...) OVER ([PARTITION BY ...] [ORDER BY ...] [ROWS ...])
```

### Thực hành
1. Row Number
- Ý nghĩa: Đánh số thứ tự dòng trong mỗi partition (hoặc toàn bộ bảng nếu không partition).
```sql
-- Đánh số thứ tự đơn hàng theo ngày đặt hàng mới nhất trước
SELECT order_id, customer_id, order_date,
       ROW_NUMBER() OVER (ORDER BY order_date DESC) AS order_rank
FROM customer_orders;

-- Đánh số thứ tự sản phẩm theo số lượng tồn kho giảm dần
SELECT product_id, product_name, units_in_stock,
       ROW_NUMBER() OVER (ORDER BY units_in_stock DESC) AS stock_rank
FROM products;

-- Đánh số thứ tự đơn hàng của từng khách hàng
SELECT order_id, customer_id, order_date,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS customer_order_num
FROM customer_orders;
```

2. Rank và Dense Rank

- Đều đánh số thứ tự dựa trên giá trị nhưng có sự khác biệt khi gặp các giá trị bằng nhau.
- **RANK**:
  - Nếu có các giá trị bằng nhau, chúng sẽ có cùng thứ hạng. Tuy nhiên, thứ hạng tiếp theo sẽ bị bỏ qua tương ứng với số lượng giá trị trùng lặp đó.
  - Ví dụ: nếu có hai giá trị bằng nhau ở vị trí thứ 1, thì giá trị tiếp theo sẽ là 3 (thay vì 2).
- **DENSE_RANK**:
  - Tương tự như RANK nhưng không bỏ qua thứ hạng. Nếu có hai giá trị bằng nhau ở vị trí thứ 1, giá trị tiếp theo vẫn sẽ là 2.

  
```sql
-- Xếp hạng sản phẩm theo số lượng tồn kho (RANK)
SELECT product_id, product_name, units_in_stock,
       RANK() OVER (ORDER BY units_in_stock DESC) AS stock_rank
FROM products;

-- Xếp hạng đơn hàng theo giá trị đơn hàng trong từng thành phố (DENSE_RANK)
SELECT co.order_id, c.city, co.order_total,
       DENSE_RANK() OVER (PARTITION BY c.city ORDER BY co.order_total DESC) AS city_order_rank
FROM customer_orders co
JOIN customers c ON co.customer_id = c.customer_id;

-- Xếp hạng nhân viên theo lương trong từng phòng ban
SELECT department, first_name, last_name, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees;
```

3. Aggregate Functions với Window Function

- Sử dụng các hàm tổng hợp như SUM, AVG, COUNT, MIN, MAX với OVER() để tính toán trên toàn bộ tập dữ liệu hoặc theo từng partition.
```sql
-- Tính tổng tiền chi tiêu lũy kế của từng khách hàng theo thời gian
SELECT customer_id, order_date, order_total,
       SUM(order_total) OVER (PARTITION BY customer_id ORDER BY order_date) AS running_total
FROM customer_orders;

-- Trung bình số lượng bán của từng sản phẩm qua các đơn hàng
SELECT product_id, order_id, quantity,
       AVG(quantity) OVER (PARTITION BY product_id) AS avg_quantity_per_product
FROM ordered_items;
```

4. LAG(), LEAD()
- LAG và LEAD là các window function rất mạnh trong PostgreSQL, giúp bạn truy xuất giá trị của dòng trước (LAG) hoặc dòng sau (LEAD) so với dòng hiện tại trong một tập hợp dữ liệu đã được sắp xếp. Điều này cực kỳ hữu ích khi phân tích xu hướng, so sánh biến động, hoặc tính toán chênh lệch giữa các dòng liên tiếp.
- LAG(expression, offset, default): Hàm này truy cập một hàng nằm trước hàng hiện tại trong cùng một phân vùng.
  - expression: Cột hoặc biểu thức mà bạn muốn lấy giá trị
  - offset (tùy chọn): Số hàng để "lùi" lại. Mặc định là 1.
  - default (tùy chọn): Giá trị trả về nếu không có hàng nào ở offset được chỉ định. Mặc định là NULL.
- LEAD(expression, offset, default): Hàm này truy cập một hàng nằm sau hàng hiện tại trong cùng một phân vùng.
  - expression: Cột hoặc biểu thức mà bạn muốn lấy giá trị.
  - offset (tùy chọn): Số hàng để "tiến" tới. Mặc định là 1.
  - default (tùy chọn): Giá trị trả về nếu không có hàng nào ở offset được chỉ định. Mặc định là NULL.

```sql
-- Bạn muốn xem mỗi khách hàng đã chi bao nhiêu trong đơn hàng hiện tại so với đơn hàng trước đó của họ.
SELECT
    customer_id,
    order_date,
    order_total,
    LAG(order_total, 1, 0) OVER (PARTITION BY customer_id ORDER BY order_date) AS previous_order_total
FROM
    customer_orders
ORDER BY
    customer_id, order_date;

-- Liệt kê customer_id, order_id, tip hiện tại, tip của đơn hàng tiếp theo (next_tip) của cùng khách hàng.
SELECT
  customer_id,
  order_id,
  tip,
  LEAD(tip) OVER (PARTITION BY customer_id ORDER BY order_date) AS next_tip
FROM customer_orders
ORDER BY customer_id, order_date;
```