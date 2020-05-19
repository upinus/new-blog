---
title: Index và các kỹ thuật quét dữ liệu chủ yếu trong PostgreSQL
author: LongBB
date: 2020-05-19 21:58:00+07:00
tags: ['database', 'pgsql_basic']
---

# 1. Sơ lược về index trong PostgreSQL
Trong PostgreSQL, index là các đối tượng database đặc biệt được sử dụng để tăng tốc độ truy vấn dữ liệu. Chúng là các cấu trúc phụ trợ, có thể bị xoá hoặc tạo lại từ dữ liệu có trong bảng.
Có rất nhiều kiểu index khác nhau trong PostgreSQL nhưng chúng cuối cùng sẽ liên kết 1 khoá (ví dụ như giá trị của cột được đánh index) với các hàng trong bảng có chứa khoá này. Mỗi hàng trong bảng được xác định duy nhất bởi 1 tuple identifier (TID) mang thông tin về vị trí (offset) của block trong file và vị trí của hàng ở trong block. Điều này đồng nghĩa với việc nếu biết về khoá hoặc một vài thông tin về khoá, chúng ta có thể dễ dàng đọc và truy cập thông tin của hàng mà không cần quét toàn bộ bảng.
Tuy nhiên, cái gì cũng có giá của nó và việc đánh index cũng vậy. Để có thể tăng tốc độ truy vấn, các index sẽ phải đánh đổi bằng một số chi phí bảo trì index nhất định. Với mỗi thao tác thêm, sửa, xoá dữ liệu ở trong bảng có liên quan đến các trường được đánh index thì index cũng cần phải được cập nhật ngay trong cùng 1 transaction và sẽ làm giảm tốc độ của các thao tác này. Do đó, mỗi khi muốn tạo một index mới thì cần cân nhắc một cách cẩn thận về lợi ích thu được và sự đánh đổi.
Để cho phép dễ dàng thêm một kiểu index vào hệ thống, một interface chung của index engine đã được bổ sung. Nhiệm vụ chính của nó là lấy các TID từ index và làm việc với chúng. Index engine được tham gia vào quá trình thực thi các câu truy vấn. Nó được gọi thông qua một kế hoạch truy vấn được tạo ra tại bước tối ưu hoá. Trình tối ưu hoá (optimizer) sắp xếp và đánh giá các cách khác nhau để thực thi câu truy vấn và  do đó, trình tối ưu hoá cần phải hiểu hết về khả năng của tất các index mà có thể áp dụng. Vì vậy, các index phải cung cấp các thông tin cần thiết của chính chúng. Với các phiên bản thấp hơn PostgreSQL 9.6, dữ liệu này sẽ được lưu trữ tại bảng `pg_am` và từ phiên bản 9.6 trở đi, dữ liệu được chuyển đến các tầng sâu hơn, bên trong các hàm đặc biệt.

# 2. Các kỹ thuật quét dữ liệu chủ yếu trong PostgreSQL
Trước khi đi vào các kỹ thuật này, ta sẽ tạo mộ bảng để minh hoạ các kỹ thuật
```
create table t(a integer, b text, c boolean);
insert into t(a,b,c) select s.id, chr((32+random()*94)::integer), random() < 0.01 from generate_series(1,100000) as s(id) order by random();
create index on t(a);
analyze t;
```
Bảng này sẽ có 3 cột: cột a bao gồm các các số từ 1 đến 100000, và ta có đánh index trên trường này. Cột b chứa các ký tự ASCII có mã từ 32 đến 126. Cột c là một giá trị boolean với khoảng 1% số hàng nhận giá trị `true` và phần còn lại nhận giá trị `false`.
## 2.1. Index scan
Thử phân tích câu truy vấn lấy các bản ghi có cột a nhận giá trị bằng 1
```
explain (costs off) select * from t where a = 1;
```
Kết quả của câu lệnh này như sau:
```
          QUERY PLAN
-------------------------------
 Index Scan using t_a_idx on t
   Index Cond: (a = 1)
```
Trong trường hợp này, trình tối ưu hoá đã quyết định sử dụng index scan. Với index scan, index sẽ trả về lần lượt từng giá trị TID của các bản ghi phù hợp cho đến khi TID cuối cùng được tìm thấy. Index engine sẽ truy cập các bản ghi thông qua các TID, lấy các row version, kiểm tra tính visibility để chống lại multiversion concurrency rules và trả về dữ liệu thu được
Tuy nhiên, do index engine sẽ truy cập lần lượt các TID để đọc dữ liệu nên có thể dẫn đến việc 1 page bị đọc lại nhiều lần. Với các ổ đĩa HDD, việc đọc dữ liệu sẽ cần phải di chuyển đưa đầu đọc dữ liệu vào rãnh ghi và hành động này thì mất nhiều thời gian hơn rất nhiều so với việc đọc dữ liệu. Do đó, với các câu truy vấn có số lượng kết quả nhiều, các TID sắp xếp một cách ngẫu nhiên thì việc sử dụng index scan có thể sẽ không đem lại hiệu quả, thậm chí còn bị giảm hiệu năng do mất quá nhiều thời gian di chuyển đầu đọc để đọc các page.
## 2.2. Bitmap scan
Như đã phân tích ở trên, index scan chỉ thích hợp khi câu truy vấn chỉ có 1 vài giá trị. Với trường hợp nhiều giá trị trả về, trình tối ưu hoá có thể lựa chọn chuyển sang bitmap scan
```
explain (costs off) select * from t where a <= 100;
```
Kết quả thu được như sau:
```
             QUERY PLAN
------------------------------------
 Bitmap Heap Scan on t
   Recheck Cond: (a <= 100)
   ->  Bitmap Index Scan on t_a_idx
         Index Cond: (a <= 100)
```
Index đầu tiên sẽ return tất cả các TID mà thoả mãn điều kiện, sắp xếp chúng trong một cấu trúc bitmap lưu trữ trong bộ nhớ. Hiểu một cách đơn giản, bitmap sẽ chứa một hash của tất cả các page (được hash dựa vào thứ tự page) và mỗi một giá trị của hash sẽ link đến 1 danh sách của tất cả các offset của tuple trong page đó. Tiếp theo, hệ thống sẽ đọc theo bitmap của pages và sau đó quét dữ liệu từ heap tương ứng với trang được lưu trữ và offset. Do vậy, mỗi trang chỉ được đọc duy nhất 1 lần. Bitmap của các page được tạo riêng cho mỗi query, không được cached hay tái sử dụng lại và bị xoá bỏ khi quá trình truy vấn kết thúc.
Tuy nhiên, cần chú ý rằng ở bước thứ hai, điều kiện có thể bị kiểm tra lại (Recheck Cond). Nếu kích thước của bitmap quá lớn, chúng ta có thể convert nó thành kiểu “lossy”, trong đó chúng ta chỉ nhớ những trang mà chứa dữ liệu thoả mãn thay vì nhớ mỗi tuple một cách độc lập. Khi điều này xảy ra, hệ thống cần phải kiểm tra mỗi tuple trong page và kiểm tra lại điều kiện để xem tuple nào sẽ được trả về.
Nếu điều kiện ở câu truy vấn nằm trên nhiều cột của bảng và các cột này đều được đánh index, bitmap scan cho phép sử dụng đồng thời trên một số index. Với mỗi index ,bitmap của các row versions sẽ được xây dựng và thực thi tổng hợp dữ liệu bằng các phép toán trên bit. Ví dụ:
```
create index on t(b);
analyze t;
explain (costs off) select * from t where a <= 100 and b = 'a';
```
```
                   QUERY PLAN
--------------------------------------------------
 Bitmap Heap Scan on t
   Recheck Cond: ((a <= 100) AND (b = 'a'::text))
   ->  BitmapAnd
         ->  Bitmap Index Scan on t_a_idx
               Index Cond: (a <= 100)
         ->  Bitmap Index Scan on t_b_idx
               Index Cond: (b = 'a'::text)
```
Ở đây BitmapAnd được sử dụng để tổng hợp 2 bitmap bằng cách sử dụng bitwise `and`
## 2.3. Sequential scan
Với các trường hợp điều kiện không chọn lọc, số lượng bản ghi phù hợp quá lớn, trình tối ưu hoá sẽ lựa chọn quét tuần tự (sequential scan) toàn bộ bảng thay vì sử dụng index. Ví dụ:
```
explain (costs off) select * from t where a <= 40000;
```
```
       QUERY PLAN
------------------------
 Seq Scan on t
   Filter: (a <= 40000)
```
Các index sẽ hoạt động càng tốt khi điều kiện chọn lọc càng tốt, có ít bản ghi thoả mãn điều kiện. Khi số lượng bản ghi thoả mãn điều kiện tăng lên, index scan sẽ không còn phù hợp do tài nguyên sử dụng để đọc các page cũng như xác suất 1 page bị đọc lại nhiều lần sẽ tăng lên. Tuỳ thuộc vào việc tính toán và so sánh các kế hoạch thực hiện của trình tối ưu hoá, khó thể nói loại scan nào sẽ được sử dụng. Tuy nhiên, để sequential scan được thực hiện, ít nhất phải có một trong các tiêu chí sau phù hợp:
Không có index nào thoả mãn truy vấn
Phần lớn các bản ghi của bảng đều thoả mãn điều kiện truy vấn

# Tài liệu tham khảo

[1] [https://postgrespro.com/blog/pgsql/3994098](https://postgrespro.com/blog/pgsql/3994098)

[2] [https://www.postgresql.org/message-id/12553.1135634231@sss.pgh.pa.us](https://www.postgresql.org/message-id/12553.1135634231@sss.pgh.pa.us)

[3] [https://severalnines.com/database-blog/overview-various-scan-methods-postgresql](https://severalnines.com/database-blog/overview-various-scan-methods-postgresq)

