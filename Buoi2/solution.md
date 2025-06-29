## Bài 1: 
- Liệt kê tên sản phẩm và tổng số lượng đã bán của từng sản phẩm, sắp xếp giảm dần theo số lượng bán.
```sql
SELECT
  p.product_name,
  SUM(oi.quantity) AS total_sold
FROM ordered_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.product_name
ORDER BY total_sold DESC;
```

## Bài 2:
- Liệt kê tên khách hàng và số đơn hàng của họ nếu họ đã mua nhiều hơn 1 đơn.
```sql
SELECT
  c.first_name || ' ' || c.last_name AS customer_name,
  COUNT(co.order_id) AS order_count
FROM customer_orders co
JOIN customers c ON co.customer_id = c.customer_id
GROUP BY customer_name
HAVING COUNT(co.order_id) > 1
ORDER BY order_count DESC;

```

## Bài 3:
- Liệt kê tên nhà cung cấp và tổng doanh thu từ các đơn hàng đã được giao thành công (status = 3).
```sql
SELECT
  s.name AS supplier_name,
  SUM(oi.unit_price * oi.quantity) AS total_revenue
FROM ordered_items oi
JOIN suppliers s ON oi.shipper_id = s.supplier_id
WHERE oi.status = 3
GROUP BY s.name
ORDER BY total_revenue DESC;
```