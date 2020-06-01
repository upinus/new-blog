---
title: Index Btree trong PostgresSQL
author: LongBB
date: 2020-05-31 22:58:00+07:00
tags: ['database', 'pgsql_basic']
---

# 1. Giới thiệu về Btree
## 1.1. Cấu trúc của Btree
B-tree index phù hợp cho các kiểu dữ liệu có thể sắp xếp được, các toán tử so sánh như lớn hơn, nhỏ hơn, lớn hơn hoặc bằng, nhỏ hơn hoặc bằng phải được định nghĩa cho kiểu dữ liệu này.
Các hàng index của B-tree được lưu trữ trong các trang (page). Ở các page lá (leaf page), các hàng sẽ chứa các dữ liệu được đánh index (các key) và tham chiếu đến dữ liệu trong bảng chính thông qua TID. Ở các page trung gian, mỗi hàng sẽ tham chiếu đến các page con (child page) của index và chứa giá trị nhỏ nhất trong các page con này.
Một số đặc điểm quan trọng của B-tree:

- B-tree là cây cân bằng (balanced  tree). Mỗi page lá đều cách gốc một khoảng cách như nhau (cùng cần đi qua số page trung gian như nhau) nên việc tìm kiếm một giá trị bất kỳ đều tốn thời gian như nhau.
- B-tree là cấu trúc đa nhánh (multi-branched) nên mỗi trang (thường là 8KB) sẽ chứa rất nhiều TID. Vì vậy, chiều sâu của B-tree thường tương đối nhỏ, thông thường chỉ có chiều sau là 4 - 5 với những bảng có nhiều dữ liệu
- Data ở trong B-tree được lưu trữ theo thứ tự không tăng (giữa các page và trong mỗi page). Các page ở cùng 1 độ sâu sẽ được liên kết với nhau thông qua một danh sách liên kết 2 chiều. Do đó, việc lấy các dữ liệu được sắp xếp đơn giản chỉ là đi dọc theo 1 chiều và không cần phải quay lại nút gốc.

Page đầu tiên của index là metapage và được tham chiếu đến gốc của index. Các page trung gian sẽ nằm bên dưới page gốc và các page lá nằm ở hàng cuối cùng.
![Btree-1](https://postgrespro.com/media/2019/05/23/i4_001.png)
## 1.2. Tìm kiếm dữ liệu trong B-tree
### 1.2.1. Tìm kiếm bằng
Để có thể dễ hình dung, giả sử cần tìm kiếm giá trị 49 trong cây B-tree ở phần 1.

Việc tìm kiếm dữ liệu sẽ bắt đầu với page gốc và cần xác định page con nào sẽ cần di chuyển xuống tiếp. Với B-tree, mỗi page trung gian sẽ chứa tham chiếu đến page con thông qua key là giá trị nhỏ nhất trong page con. Ở đây ta thấy 49 > 4 nhưng 49 > 32 nên 49 không thể nằm trong page được tham chiếu bởi số 4 (page con tham chiếu bởi giá trị 4 sẽ chứa các giá trị phải lớn hơn hoặc bằng 4 và nhỏ hơn hoặc bằng 32) và 64 > 49 nên 49 cũng không thể nằm trong page con được tham chiếu bởi 64 (page con tham chiếu trong giá trị 64 sẽ chỉ chứa các giá trị phải lớn hơn hoặc bằng 64). 32 < 49 < 64 nên giá trị 49 sẽ có thể nằm trong nhánh được tham chiếu bởi giá trị 32.

Đi tiếp xuống page con được tham chiếu bởi giá trị 32, có 3 giá trị được tham chiếu trong page này là 32, 43 và 49. Ở đây ta thấy, page con tham chiếu bởi giá trị 43 sẽ chứa các giá trị trong khoảng từ 43 đến 49 và page con tham chiếu bởi giá trị 49 sẽ chứa các giá trị lớn hơn 49 nên nếu chọn đi xuống page tham chiếu bởi giá trị 49 thì có thể ta sẽ bị mất vài giá trị ở các page con tham chiếu bởi value 43.

Đi xuống page con tham chiếu bởi giá trị 43 ta thấy đây đã là page lá và có chứa 2 giá trị là 43 và 49 link đến các TID ở bảng dữ liệu. Đi dọc theo các page lá từ vị trí page lá đang đứng sang phải cho đến khi gặp các giá trị lớn hơn giá trị 49 cần tìm ta sẽ thu được danh sách các TID ứng với giá trị 49 trên bảng dữ liệu.
![Btree-2](https://postgrespro.com/media/2019/05/23/i4_002.png)
### 1.2.2. Tìm kiếm không bằng
Khi tìm kiếm không bằng bởi điều kiện x <= b (hoặc x >= b). Đầu tiên, chúng ta sẽ tiến hành tìm kiếm các giá trị ở trong index mà thoả mãn điều kiện bằng: x = b rồi duyện các page lá sang bên trái với tìm kiếm x <= b hoặc duyệt sang phải với điều kiện x >= b
Hình sau minh hoạ quá trình tìm kiếm x < 35
![Btree-3](https://postgrespro.com/media/2019/05/23/i4_003.png)
### 1.2.3. Tìm kiếm trong khoảng giá trị
Khi tìm kiếm trong khoảng giá trị a <= x <= b, chúng ta sẽ tiến hành tìm kiếm các index thoả mãn tìm kiếm bằng x = a, tiếp theo se duyệt dọc theo các page lá từ index đầu tiên thoả mãn x = a sang bên phải cho đến khi gặp giá trị đầu tiên x > b thì dừng lại. Bỏ đi giá trị cuối cùng mà khi ấy x > b (tức là tất cả các giá trị thu được sẽ luôn thoả mãn x <= b) sẽ thu được danh sách các index cần tìm kiếm
Hình sau minh hoạ quá trình tìm kiếm các index thoả mãn điều kiện 23 <= n <= 64
![Btree-4](https://postgrespro.com/media/2019/05/23/i4_004.png)
## 1.3. Sắp xếp
Như đã mô tả ở phần trên, dữ liệu lưu trữ trong Btree là dữ liệu đã được sắp xếp. Do đó, nếu bảng có index theo điều kiện sắp xếp của câu truy vấn, trình tối ưu hoá sẽ xem xét cả 2 tuỳ chọn: sử dụng index scan với kết quả trả về đã được sắp xếp sẵn hoặc sử dụng sequential scan sau đó sắp xếp kết quả thu được.
# 2. Sử dụng Btree index với nhiều cột
Btree index có thể được sử dụng trên nhiều cột để thích hợp với các truy vấn có nhiều điều kiện.

Khi Btree sử dụng trên nhiều cột, thứ tự các cột sẽ rất quan trọng: data tại các page sẽ được sắp xếp theo thứ tự cột đầu tiên trước; với các trường hợp có cùng giá trị của cột đầu tiên, thứ tự sẽ được sắp xếp tiếp theo giá trị của cột thứ 2.

Giả sử ta có bảng customers bao gồm 2 trường là customer_name và city. Có 1 index Btree index_customer_name_city được hình thành trên 2 trường theo thứ tự (customer_name, city). Khi đó, với các truy vấn yêu cầu tìm các khách hàng có tên là “John” hoặc truy vấn tìm khách hàng tên “John" ở thành phố “Los Angeles” thì hệ thống chỉ đơn giản sử dụng index scan với tìm kiếm bằng để trả về kết quả. Tuy nhiên, với các truy vấn như tìm kiếm các khách hàng ở thành phố “Los Angeles” thì hệ thống không thể sử dụng Btree index do khi đi từ page gốc, ta không thể biết cần phải đi xuống page nào tiếp theo (các page sắp xếp theo customer_name trước rồi mới đến city nên trong index_customer_name_city, các city sẽ không có thứ tự nếu không cùng tên khách hàng).

Trong các trường hợp như ví dụ trên, tuỳ thuộc vào các nhu cầu của các truy vấn mà ta sẽ có những chiến lược đánh index cụ thể. Với hệ thống, hầu hết các truy vấn là tìm theo tên khách hàng và tìm cả tên khách hàng lẫn city ta sẽ đánh index trên 2 cột theo thứ tự (customer_name, city). Với các hệ thống mà hầu hết truy vấn là truy vấn tìm khách hàng theo city và tìm khách hàng theo tên thuộc về 1 city xác định thì việc đánh index trên 2 cột theo thứ tự (city, customer_name) sẽ hợp lý hơn. Còn các hệ thống có cả truy vấn tìm khách hàng theo tên, tìm khách hàng theo city và tìm khách hàng theo tên thuộc về 1 city xác định thì việc đánh 2 index riêng lẻ sẽ hợp lý hơn. Khi đó, các truy vấn trên cả 2 trường sẽ sử dụng toán tử `AND` trên 2 bitmap được thành lập từ 2 index.
# 3. Btree index với NULL
Các giá trị NULL sẽ được đặt trên một đầu phía trước (NULLS FIRST) hoặc phía sau (NULLS LAST) của index tuỳ thuộc vào cách tạo index. Điều này sẽ quan trọng với các truy vấn có sắp xếp: index có thể được sử dụng nếu câu truy vấn yêu cầu thứ tự của các NULLS trong mệnh đề `order_by` phù hợp với cách định nghĩa index. Mặc định, các giá trị NULL sẽ được đặt ở cuối (NULLS LAST) của index
Quay lại ví dụ ở mục 2, giả sử ta tạo chỉ tạo index trên cột city của customers:
```
create index customer_city on customers(city);
```
Khi đó nếu ta yêu cầu truy vấn tất cả các customers và sắp xếp theo thứ tự các customer chưa xác định được city đứng trước thì trình tối ưu hoá sẽ quyết định sử dụng sequential scan rồi sắp xếp kết quả tìm được.
```
explain(costs off) select * from customers order by city NULLS FIRST;
          QUERY PLAN
------------------------------
 Sort
   Sort Key: city NULLS FIRST
   ->  Seq Scan on customers
```
Nếu ta tạo index trên cột city với định nghĩa các giá trị NULL được đặt ở phía trước thì lúc này khi thực hiện truy vấn trên, trình tối ưu hoá sẽ quyết định sử dụng index scan và trả về kết quả mà không phải sắp xếp
```
create index customer_city_null_first on customers(city NULL FIRST);
explain(costs off) select * from customers order by city NULLS FIRST;
                       QUERY PLAN
--------------------------------------------------------
 Index Scan using customer_city_null_first on customers
```
# 4. Tổng Kết
Bài viết trên đã giới thiệu một cách sơ lược và ngắn gọn về index Btree trong PostgreSQL. Đây là loại index phổ biến và hay được sử dụng nhất. Hiểu về nguyên lý và cách thức hoạt động của Btree index sẽ giúp chúng ta thực hiện việc đánh index một cách hiệu quả hơn.
# Tài liệu tham khảo
[1] [https://postgrespro.com/blog/pgsql/4161516](https://postgrespro.com/blog/pgsql/4161516)
