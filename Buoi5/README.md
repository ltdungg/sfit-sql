# View

View trong PostgreSQL là một đối tượng cơ sở dữ liệu cho phép bạn lưu trữ các truy vấn SQL đã được định nghĩa trước. Khi bạn tạo một view, nó sẽ lưu trữ một truy vấn SQL và có thể được sử dụng như một bảng thông thường trong các truy vấn khác.

# Các loại view trong PostgreSQL
## 1. Standard View: Là view thông thường, lưu trữ truy vấn SQL và có thể được sử dụng trong các truy vấn khác.
- Standard view là một bảng ảo được định nghĩa bởi một truy vấn SELECT. Khi bạn truy vấn view, PostgreSQL sẽ thực thi lại câu lệnh SELECT gốc, lấy dữ liệu mới nhất từ các bảng cơ sở. View không lưu trữ dữ liệu, chỉ lưu truy vấn.
- **Ứng dụng:**
  - Đơn giản hóa truy vấn phức tạp: Bạn có thể gói gọn các phép JOIN, lọc, tổng hợp... vào một view, sau đó chỉ cần SELECT * FROM view_name.
  - Tăng bảo mật: Chỉ cung cấp một phần dữ liệu (ví dụ, không cho phép truy cập trực tiếp vào các cột nhạy cảm).
  - Tạo lớp trừu tượng: Giúp ứng dụng hoặc người dùng cuối làm việc với dữ liệu dễ dàng hơn.
- **Hạn chế:**
  - Truy vấn phức tạp trong view có thể làm giảm hiệu năng nếu view được lồng nhiều lớp hoặc truy vấn trên dữ liệu lớn.
  - Không lưu dữ liệu, nên không tối ưu cho báo cáo lớn cần truy vấn nhiều lần cùng một kết quả.

```sql
CREATE [OR REPLACE] VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

```sql
CREATE VIEW view_product_stock AS
SELECT product_id, product_name, units_in_stock FROM products;
-- Khi gọi SELECT * FROM view_product_stock;, bạn luôn nhận được dữ liệu tồn kho mới nhất.

-- Chỉ lấy các sản phẩm sắp hết hàng
CREATE VIEW view_low_stock AS
SELECT product_id, product_name, units_in_stock
FROM products
WHERE units_in_stock < 50;

-- Tổng hợp số lượng đã bán của từng sản phẩm.
CREATE VIEW view_product_sales AS
SELECT 
  p.product_id,
  p.product_name,
  SUM(oi.quantity) AS total_sold
FROM products p
JOIN ordered_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name;

-- View lồng view Lấy sản phẩm bán chạy nhất từ view tổng hợp doanh số.
CREATE VIEW view_top_product AS
SELECT *
FROM view_product_sales
WHERE total_sold = (SELECT MAX(total_sold) FROM view_product_sales);


-- View chỉ cho phép cập nhật dữ liệu thỏa mãn điều kiện (WITH CHECK OPTION)
CREATE VIEW view_cheap_products AS
SELECT * FROM products WHERE sale_price < 5
WITH CHECK OPTION;
-- Khi một View có thể cập nhật và có một mệnh đề WHERE trong định nghĩa, WITH CHECK OPTION đảm bảo rằng mọi thay đổi (INSERT/UPDATE) thông qua View phải tuân thủ điều kiện WHERE đó. Nếu không, thao tác sẽ bị từ chối
```
Lưu ý:
- Nếu view được tạo từ nhiều bảng (JOIN) hoặc có các phép tổng hợp (GROUP BY, HAVING), DISTINCT, LIMIT, UNION, v.v., thì view này mặc định là read-only (chỉ đọc)
- Khi view chỉ đọc, mọi thao tác cập nhật sẽ báo lỗi.
- Nếu view được tạo từ một bảng duy nhất, không có các phép tổng hợp (GROUP BY, HAVING), không có DISTINCT, không có JOIN, không có LIMIT/OFFSET, không có các hàm cửa sổ hoặc tập hợp, thì view này là updatable (cập nhật được tự động)
- Khi bạn thực hiện INSERT, UPDATE hoặc DELETE trên view này, PostgreSQL sẽ chuyển đổi truy vấn thành thao tác tương ứng trên bảng gốc.
- **Kết quả:** Dữ liệu trong bảng gốc sẽ được thay đổi tương ứng với thao tác trên view.

## 2. Temporary View: Là view tạm thời
    - Temporary view là view chỉ tồn tại trong phiên làm việc hiện tại (session). Khi session kết thúc, view sẽ tự động bị xóa. Temporary view rất hữu ích cho các truy vấn tạm thời, xử lý dữ liệu trung gian hoặc thử nghiệm.
    - **Ứng dụng:**
      - Phân tích dữ liệu ad-hoc, thử nghiệm truy vấn phức tạp mà không ảnh hưởng đến cấu trúc database lâu dài.
      - Giúp giảm tải bộ nhớ và tài nguyên khi không cần thiết phải lưu trữ view lâu dài.
    - **Đặc điểm**:
      - Phạm vi session: Chỉ có session tạo ra view mới nhìn thấy và sử dụng được view này. Người dùng khác không thể truy cập.
      - Không ảnh hưởng đến view hoặc bảng thường cùng tên: Nếu có view hoặc bảng thường cùng tên, trong session đó sẽ ưu tiên temporary view
      - Tự động xóa: Khi session kết thúc, view sẽ tự động bị xóa, không cần phải xóa thủ công.
```sql
CREATE TEMPORARY VIEW view_name AS
SELECT ... FROM ... WHERE ...;
```
```sql
--  Phân tích sản phẩm tồn kho thấp trong phiên làm việc
CREATE TEMPORARY VIEW temp_low_stock AS
SELECT product_id, product_name, units_in_stock
FROM products
WHERE units_in_stock < 50;

-- Chia nhỏ logic truy vấn phức tạp
CREATE TEMPORARY VIEW temp_orders_2022 AS
SELECT * FROM customer_orders WHERE EXTRACT(YEAR FROM order_date) = 2022;

CREATE TEMPORARY VIEW temp_top_customers_2022 AS
SELECT customer_id, SUM(order_total) AS total_spent
FROM temp_orders_2022
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 5;
```

- So sánh với CTE
  - Temporary view rất giống CTE (WITH ...) về mặt chia nhỏ logic, nhưng CTE chỉ tồn tại trong một truy vấn, còn temporary view tồn tại suốt session và có thể dùng lại nhiều lần trong các truy vấn khác nhau trong cùng session
  - Temporary view thích hợp khi bạn cần dùng lại bảng ảo này nhiều lần trong phiên làm việc, còn CTE phù hợp cho các truy vấn phức tạp nhưng chỉ dùng một lần.
- Nhược điểm:
  - Chỉ dùng được trong phiên làm việc hiện tại, không chia sẻ được giữa các session.
  - Không phù hợp cho các báo cáo, dashboard, hoặc ứng dụng cần dùng lâu dài.
  - Nếu session bị ngắt kết nối, mọi temporary view sẽ mất.

## 3. Materialized View: View vật lý
   - Materialized view là một đối tượng trong PostgreSQL lưu trữ kết quả của một truy vấn SQL dưới dạng vật lý trên đĩa, giống như một bảng tạm thời được cập nhật định kỳ
   - Khác với view thường (standard view), materialized view không truy vấn dữ liệu gốc mỗi lần bạn gọi, mà trả về dữ liệu đã được "cache" sẵn từ lần cập nhật gần nhất.
   - Để cập nhật dữ liệu mới nhất, bạn phải dùng lệnh REFRESH MATERIALIZED VIEW.
   - **Khi nào sử dụng**:
        - Khi truy vấn tổng hợp phức tạp, tốn nhiều tài nguyên, nhưng dữ liệu không cần cập nhật liên tục (ví dụ: báo cáo doanh số, dashboard, phân tích dữ liệu lớn).
        - Khi bạn cần tăng tốc độ truy vấn cho các báo cáo, BI, hoặc hệ thống đọc nhiều, ghi ít
        - Khi dữ liệu nguồn thay đổi không quá thường xuyên, hoặc bạn chấp nhận độ trễ nhỏ giữa dữ liệu gốc và dữ liệu báo cáo.
   - **Ưu điểm:**
     - Truy vấn rất nhanh: Vì dữ liệu đã được tính toán và lưu sẵn trên đĩa, truy vấn gần như nhanh như bảng thường
     - Giảm tải hệ thống: Không phải thực thi lại các phép JOIN, tổng hợp nặng mỗi lần truy vấn.
     - Có thể tạo chỉ mục: Bạn có thể tạo index trên materialized view để tăng tốc truy vấn hơn nữa
   - **Nhược điểm:**
     - Dữ liệu có thể lỗi thời: Phải chủ động cập nhật (refresh) khi muốn lấy dữ liệu mới nhất
     - Tốn thêm dung lượng lưu trữ: Vì lưu vật lý trên đĩa như bảng thật
     - Không tự động cập nhật: Nếu bảng nguồn thay đổi, materialized view không tự thay đổi cho đến khi bạn refresh.

```sql
-- Tổng hợp số lượng đã bán của từng sản phẩm
CREATE MATERIALIZED VIEW mv_product_sales AS
SELECT 
  p.product_id,
  p.product_name,
  SUM(oi.quantity) AS total_sold
FROM products p
JOIN ordered_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name;

SELECT * FROM mv_product_sales ORDER BY total_sold DESC;

REFRESH MATERIALIZED VIEW mv_product_sales;

-- Tổng hợp đơn hàng theo năm
CREATE MATERIALIZED VIEW mv_orders_by_year AS
SELECT 
  EXTRACT(YEAR FROM order_date) AS order_year,
  COUNT(*) AS total_orders
FROM customer_orders
GROUP BY order_year;
```

- Khi nào refresh materialized view?
  - Khi có thay đổi lớn ở bảng nguồn.
  - Định kỳ (theo lịch), ví dụ mỗi ngày, mỗi giờ, hoặc khi cần báo cáo mới nhất.
  - Khi người dùng yêu cầu dữ liệu cập nhật.
- Lưu ý:
  - Không nên dùng materialized view cho dữ liệu thay đổi liên tục cần cập nhật real-time.
  - Nên lên lịch refresh hợp lý để cân bằng giữa tốc độ và độ mới của dữ liệu.
  - Có thể dùng kết hợp với các bảng báo cáo, dashboard, hoặc các truy vấn tổng hợp nặng.
 