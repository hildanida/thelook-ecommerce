<div style="text-align: left;">
  <h2 style="background-color: #2C3E3D; color: white; padding: 10px 20px; margin-bottom: 20px; font-weight: bold; display: inline-block; font-size: 30px">SQL Process</h2>
</div>

---

<div style='text-align: justify'>

```sql
-- Membuat tabel sementara untuk agregasi penjualan produk per bulan
WITH report_monthly_orders_product_agg AS (
  SELECT
    DATE_TRUNC(DATE(o.created_at), MONTH) AS year_month,
    p.id AS product_id,
    p.name AS product_name,
    p.brand,
    p.category,
    COUNT(*) AS total_items_sold, -- Total unit produk yang terjual
    SUM(o.sale_price) AS total_revenue -- Total pendapatan dari penjualan produk
  FROM
    `bigquery-public-data.thelook_ecommerce.order_items` AS o
  JOIN
    `bigquery-public-data.thelook_ecommerce.products` AS p
    ON o.product_id = p.id
  WHERE
    o.status = 'Complete' -- Hanya mengambil transaksi yang sudah selesai
  GROUP BY
    year_month, product_id, product_name, brand, category
),

-- Membuat peringkat produk berdasarkan total revenue tiap bulan
ranked_products AS (
  SELECT 
    *,
    RANK() OVER (PARTITION BY year_month ORDER BY total_revenue DESC) AS rank
  FROM report_monthly_orders_product_agg
)

-- Menampilkan 5 produk setiap bulan berdasarkan total revenue (penjualan tertinggi)
SELECT
  *
FROM ranked_products
WHERE rank <= 5 -- Mengambil hanya 5 produk teratas
ORDER BY year_month, rank;
```

</div>



<div style="text-align: left;">
  <h2 style="background-color: #2C3E3D; color: white; padding: 10px 20px; margin-bottom: 20px; font-weight: bold; display: inline-block; font-size: 30px">Explanation</h2>
</div>

---

<div style='text-align: justify'>

<strong>Bagian I. Agregasi penjualan produk per bulan</strong>

- `WITH report_monthly_orders_product_agg AS (...)` membuat tabel sementara (CTE) untuk menghitung total penjualan produk tiap bulan.
- `DATE_TRUNC(DATE(o.created_at), MONTH) AS year_month` digunakan untuk mengelompokkan transaksi berdasarkan bulan.
- `p.id AS product_id, p.name AS product_name, p.brand, p.category` mengambil informasi unik dari produk.
- `COUNT(*) AS total_items_sold` menghitung total unit yang terjual dalam bulan tersebut.
- `SUM(o.sale_price) AS total_revenue` menjumlahkan total pendapatan dari penjualan produk.
- JOIN dilakukan antara tabel `order_items` dan `products` agar bisa mengakses detail produk.
- Hanya transaksi berstatus `WHERE o.status = 'Complete'` yang diikutkan dalam perhitungan.
- `GROUP BY` digunakan untuk agregasi data berdasarkan bulan dan produk.

<strong>Bagian II. Peringkat produk berdasarkan revenue</strong>

- `RANK() OVER (...) AS rank` memberi peringkat produk tiap bulan berdasarkan total pendapatan.
- `PARTITION BY year_month` memastikan peringkat dihitung secara terpisah untuk setiap bulan.
- `ORDER BY total_revenue DESC` mengurutkan produk berdasarkan pendapatan tertinggi.

<strong>Bagian III. Menampilkan top 5 produk tiap bulan</strong>

- `WHERE rank <= 5` menyaring hanya produk-produk dengan peringkat 1 sampai 5.
- `ORDER BY year_month, rank` mengurutkan hasil berdasarkan bulan dan posisi peringkat.

</div>
