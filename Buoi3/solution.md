# BÀI TẬP

## Sử dụng Subquery 
- Tìm tên các sản phẩm chưa từng được đặt hàng bởi bất kỳ khách hàng nào.

```sql
SELECT product_name
FROM products p
WHERE NOT EXISTS (
  SELECT 1 FROM customer_orders co WHERE co.product_id = p.product_id
);
```

- Lấy tên nhà cung cấp, tổng số đơn hàng đã giao thành công, số sản phẩm khác nhau đã giao, tổng số lượng đã giao thành công
```sql
SELECT 
  s.name AS supplier_name,
  -- Tổng số đơn hàng đã giao thành công
  (SELECT COUNT(*) FROM ordered_items oi WHERE oi.shipper_id = s.supplier_id AND oi.status = 3) AS delivered_orders,
  -- Số sản phẩm khác nhau đã giao thành công
  (SELECT COUNT(DISTINCT product_id) FROM ordered_items oi WHERE oi.shipper_id = s.supplier_id AND oi.status = 3) AS delivered_products,
  -- Tổng số lượng đã giao thành công
  (SELECT COALESCE(SUM(quantity), 0) FROM ordered_items oi WHERE oi.shipper_id = s.supplier_id AND oi.status = 3) AS total_quantity_delivered
FROM suppliers s;
```

- Tìm sản phẩm có tổng số lượng bán ra lớn hơn trung bình tổng số lượng bán của tất cả sản phẩm
```sql
SELECT product_name, total_sold
FROM (
  SELECT p.product_name, SUM(oi.quantity) AS total_sold
  FROM products p
  JOIN ordered_items oi ON p.product_id = oi.product_id
  GROUP BY p.product_name
) AS sales_summary
WHERE total_sold > (
  SELECT AVG(total_quantity) FROM (
    SELECT SUM(quantity) AS total_quantity
    FROM ordered_items
    GROUP BY product_id
  ) AS avg_table
);

```
