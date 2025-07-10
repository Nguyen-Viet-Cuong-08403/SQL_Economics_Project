# SQL_Economics_Project
Đây là một dự án SQL thể hiện khả năng phân tích dữ liệu từ một bộ dữ liệu bán hàng thương mại điện tử thực tế. Mục tiêu của dự án là trình bày kỹ năng viết truy vấn SQL ở nhiều mức độ, từ cơ bản đến nâng cao, phục vụ cho các nhu cầu phân tích và ra quyết định trong kinh doanh.
## Mục tiêu chính
Thể hiện kỹ năng sử dụng SQL trong các tác vụ:

+ Cơ bản: lọc dữ liệu, đếm, phân nhóm.

+ Trung bình: phân tích theo thời gian, tính trung bình, phần trăm đóng góp.

+ Nâng cao: sử dụng CTE, hàm cửa sổ (window functions), xếp hạng, tính tăng trưởng theo năm.

Mô phỏng các bài toán phân tích dữ liệu thực tế trong lĩnh vực thương mại điện tử, như:

+ Phân tích doanh thu và đơn hàng

+ Hiệu suất của từng nhà bán hàng

+ Hành vi khách hàng theo khu vực

+ Đóng góp của từng nhóm sản phẩm

Làm nền tảng để phát triển thành báo cáo, dashboard hoặc pipeline tự động sau này.
## Top những câu hỏi 
### 1. Liệt kê tất cả các thành phố khách hàng đang sinh sống

```sql
Select distinct customer_city as Tên_thành_phố
from Customers
```
### 2. Đếm số lượng đơn hàng được đặt trong năm 2017.

```sql
select count(order_id) as Số_lương_đơn_hàng
from Orders 
where year(order_purchase_timestamp) = 2017
```
### 3. Tính tổng doanh thu theo từng loại sản phẩm.

```sql
select  p.product_category_name, sum(i.price), sum(i.freight_value) , sum(i.price) + sum(i.freight_value)
from Order_items as i
join products as p
on i.product_id=p.product_id
group by p.product_category_name
```
### 4. Tính phần trăm đơn hàng đã được thanh toán trả góp.

```sql
with table_1 as (
select count(order_id) as Tổng_đươn_TC, (
    select count(order_id) as Tổng_đơn_hàng
    from orders
) as Tổng_đơn_hàng
from orders
where order_status = 'delivered'
)
select *, cast(Tổng_đươn_TC as decimal(10,2) )/Tổng_đơn_hàng * 100 as _tỉ_lệ
from table_1
```
### 5. Đếm số lượng khách hàng theo từng bang/khu vực.

```sql
select customer_state as Khu_vực, count(distinct customer_id) as Số_lượng 
from customers
group by customer_state
order by count(distinct customer_id) DESC
```
### 6. Đếm số lượng đơn hàng theo từng tháng trong năm 2018.

```sql
select month(order_purchase_timestamp) as Tháng, year(order_purchase_timestamp) as Năm,  count(order_id) as Tổng_đơn_hàng
from orders 
where year(order_purchase_timestamp) = 2018
group by month(order_purchase_timestamp), year(order_purchase_timestamp)
```
### 7. Tính số lượng sản phẩm trung bình trên mỗi đơn hàng, phân theo thành phố của khách hàng.

```sql
select customer_city, cast(sum(order_item_id) as Decimal(10,2)) /count(order_items.order_id) as Số_lượng_SPTB 
from Order_items
join orders 
on orders.order_id=Order_items.order_id
join customers
on orders.customer_id=customers.customer_id
group by customer_city
```
### 8. Tính phần trăm doanh thu mà mỗi nhóm sản phẩm đóng góp vào tổng doanh thu.

```sql
with table_1 as (
select Products.product_category_name, sum(Order_items.price) as Tổng_doanh_thu_theo_loại, (
    select sum(order_items.price)
    from order_items
) as Tổng_doanh_thu
from Order_items
join Products 
on Order_items.product_id=Products.product_id
group by Products.product_category_name
)
select * , cast (Tổng_doanh_thu_theo_loại as Decimal(10,2))/ Tổng_doanh_thu * 100 as _tỉ_lệ
from table_1
```
### 9. Phân tích mối tương quan giữa giá sản phẩm và số lần sản phẩm đó được mua

```sql
select Products.product_category_name, count(order_id) as Tổng_đơn, avg(Order_items.price) as Trung_bình_trên_đơn
from Order_items
join Products
on Order_items.product_id=Products.product_id
group by Products.product_category_name
```

### 10. Tính tổng doanh thu mà mỗi nhà bán hàng tạo ra và xếp hạng theo doanh thu.

```sql
select Order_items.seller_id,  sum(Order_payments.payment_value) as Tổng_doanh_thu , row_number () over (order by sum(Order_payments.payment_value) DESC ) as Xếp_hạng
from Order_items
join Order_payments 
on Order_payments.order_id= Order_items.order_id
group by Order_items.seller_id
```
### 11. Tính doanh thu tích lũy theo từng tháng và từng năm.

```sql
select concat(month(order_purchase_timestamp), '.',  year(order_purchase_timestamp)) as Tháng, sum(Order_items.price)
from orders
join Order_items
on orders.order_id=Order_items.order_id
group by concat(month(order_purchase_timestamp), '.',  year(order_purchase_timestamp))
order by concat(month(order_purchase_timestamp), '.',  year(order_purchase_timestamp)) ASC
```
### 13. Tính tốc độ tăng trưởng doanh thu qua từng năm.

```sql
with Table_1 as (
select year(order_purchase_timestamp) as Năm, sum(Order_items.price) as Tổng_doanh_thu
from orders
join Order_items
on orders.order_id=Order_items.order_id
group by year(order_purchase_timestamp)
), Table_2 as (
Select *, lag (Tổng_doanh_thu, 1) over (order by Năm ASC ) as Doanh_thu_năm_trước
from Table_1
)
select *, (Tổng_doanh_thu- Doanh_thu_năm_trước) / Doanh_thu_năm_trước * 100
from Table_2
```
### 14.Xác định top 3 khách hàng chi tiêu nhiều nhất trong mỗi năm.

```sql
with table_1 as (
select c.customer_id, sum(i.price) as Tiền_chi , year(o.order_purchase_timestamp) as Năm , ROW_NUMBER() over (partition by year(o.order_purchase_timestamp) order by sum(i.price) DESC ) as Xếp_hạng
from Customers as c
join Orders as o
on c.customer_id= o.customer_id
join Order_items as i
on o.order_id=i.order_id
group by year(o.order_purchase_timestamp), c.customer_id
)
select *
from table_1
where Xếp_hạng <=3
```







































