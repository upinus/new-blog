---
title: Cách thức PostgreSQL lưu trữ dữ liệu ở tầng vật lý
author: LongBB
date: 2019-12-24 16:08:00+07:00
tags: ['database', 'pgsql_basic']
---


# 1. Cấu trúc logic của Database Cluster

*Database cluster* là tập hợp các database được quản lý bởi PostgreSQL server. Mỗi database lại là một tập hợp của các *database object*. Các database object là các cấu trúc dữ liệu được sử dụng để lưu trữ hoặc tham chiếu dữ liệu. Table, index, view, function và nhiều thứ khác nữa là các database object. Trong PostgreSQL, bản thân các database cũng chính là các database object và được phân tách về mặt logic với nhau. Tất cả các database object (như table, index, …) đều thuộc về các database tương ứng của chúng.

![logical_database_cluster](https://lh5.googleusercontent.com/6pQ4vNM1YVXVCwUTd0i3cRYzlCirLn_Wi_bRsF5fkF41m4GTTpC66pTLDpMTXpiYDolmjaq3dcoYSrqgpl374Vn8KhspOQP0r6rzKsyw_FFtSYL0FLaHbrz7ecEbRdC4cFLiNCUL)

Tất cả các database object trong PostgreSQL được quản lý bằng các *object identifier (OID)* tương ứng của chúng. Mỗi OID là một số tự nhiên 4-byte không âm. Mối quan hệ giữa database object và OID tương ứng của chúng được lưu trữ trong system catalog thích hợp, dựa vào từng loại object. Ví dụ, OID của database sẽ được lưu trữ tại pg_database, OID của table sẽ được lưu trữ tại pg_class và chúng ta có thể truy vấn chúng thông qua các câu truy vấn như sau:

```
sampledb=# SELECT datname, oid FROM pg_database WHERE datname = 'sampledb';
datname   | oid
----------+-------
sampledb  | 16384
(1 row)

sampledb=# SELECT relname, oid FROM pg_class WHERE relname = 'sampletbl';
relname    | oid
-----------+-------
sampletbl  | 18740
(1 row)
```

# 2. Cấu trúc vật lý của Database Cluster

Về mặt vật lý, một database cluster là một thư mục, và nó sẽ bao gồm một vài thư mục con được đặt dưới thư mục con `base`. Đường dẫn của thư mục này thường đặt thành biến môi trường `$PGDATA`. Chúng ta có thể tìm đường dẫn đến thư mục này bằng cách truy cập với quyền *superuser* và thực hiện câu lệnh:

```
SHOW data_directory;
```

![physical_database_cluster](https://lh4.googleusercontent.com/9LoKSgm-1LiJ4OUtbW34OQDmVUk2rmYOk7pSmxoy0CDrqU-CKiQ4vFOeniu2rXTUtdZMYNesocklOw3a1WKI6flyZddBBGyGgAyb4RS_1GQRGbrkVx0ZHU5O8ohsbrAfNnGOHqOw)

Mỗi một database là một thư mục con nằm trong thư mục `base`, và mỗi một table, index tối thiểu là một file được lưu trữ trong thư mục của database mà chúng thuộc về. Ngoài thư mục `base`, còn một số thư mục con khác trong `$PGDATA` chứa thông tin dữ liệu cụ thể và các file cấu hình. Ngoài ra, PostgreSQL còn hỗ trợ các tablespace. Mỗi tablespace là một một thư mục chứa các dữ liệu bên ngoài thư mục `base`.

## 2.1. Cấu trúc của một Database Cluster

Database Cluster là một thư mục bao gồm một số file chính và thư mục con, được mô tả trong tài liệu chính thức ở [đây](https://www.postgresql.org/docs/current/storage-file-layout.html).

## 2.2. Cấu trúc của các Database

Mỗi một database là một thư mục con nằm trong thư mục `base`, và tên của thư mục này chính là *OID* tương ứng của database đó. Ví dụ, với database `sampledb` có OID là `16384` thì toàn bộ database `sampledb` sẽ nằm trong thư mục `$PGDATA/base/16384`.

## 2.3. Cấu trúc các file tương ứng với table và index

Mỗi table và index mà có dung lượng nhỏ hơn 1GB là một file được lưu trữ dưới thư mục database mà nó thuộc về. Table và index là các database object nên cũng được quản lý thông qua các OID của chúng, trong đó, các file sẽ được quản lý bởi các biến gọi là *relfilenode*. Giá trị relfilenode của các table và index về cơ bản là OID nhưng không phải lúc nào cũng vậy. Giá trị của relfilenode có thể bị thay đổi bởi một vài câu lệnh như `TRUNCATE`, `REINDEX`, `CLUSTER`... Ví dụ, nếu chúng ta truncate table `sampletbl`, PostgreSQL sẽ gán 1 relfilenode mới cho table, tạo ra 1 file mới và xoá bỏ file cũ đi.

```
sampledb=# SELECT relname, oid, relfilenode FROM pg_class WHERE relname = 'sampletbl';
  relname  |  oid  | relfilenode
-----------+-------+-------------
 sampletbl | 18740 |       18740
(1 row)

sampledb=# TRUNCATE sampletbl;
TRUNCATE TABLE

sampledb=# SELECT relname, oid, relfilenode FROM pg_class WHERE relname = 'sampletbl';
  relname  |  oid  | relfilenode
-----------+-------+-------------
 sampletbl | 18740 |       18812
(1 row)

```

Một table hoặc index sẽ được lưu trữ trong file có tên là relfilenode của nó. Nếu table, index có dung lượng đạt tới 1GB thì khi thêm mới dữ liệu, PostgreSQL sẽ tạo ra một file mới với tên là `relfilenode.1` và sử dụng nó cho đến khi nó đạt tới 1GB thì PostgreSQL lại tiếp tục tạo ra file mới là `relfilenode.2`. Quá trình tiếp theo cứ tiếp tục tiếp diễn như vậy. Kích thước tối đa của file (mặc định là 1 GB) có thể thay đổi được khi sử dụng cấu hình với tuỳ chọn `--with-segsize` khi builing PostgreSQL.

Ví dụ, với bảng `sampletbl` có relfilenode là `18812`, thuộc về database sampledb có OID là `16384` thì dữ liệu của bảng sẽ được lưu trữ trong các file có đường dẫn là `$PGDATA/base/16384/18812`, `$PGDATA/base/16384/18812.1`, `$PGDATA/base/16384/18812.2` …

Ngoài các file chứa dữ liệu của table, index có dạng `relfilenode.*`, mỗi table, index còn được liên kết với các file có hậu tố là `_fsm` (relfilenode_fsm) và `_vm` (relfilenode_vm). relfilenode_fsm (free space map) chứa thông tin về dung lượng trống và relfilenode_vm (visibility map) chứa thông tin về các file không chứa các dữ liệu lỗi thời (Có dữ liệu lỗi thời do PostgreSQL có cơ chế lưu version của các bản ghi). Index chỉ có free space map mà không có visibility map.

# 3. Cấu trúc file dữ liệu của table

Các file dữ liệu (dữ liệu của table, index, cũng như là free space map và visibility map) sẽ được chia thành từng trang (*page*) với độ dài cố định, mặc định là `8192 byte (8KB)`. Các trang trong mỗi file sẽ được đánh số lần lượt từ 0. Số này được gọi là *block number*.

![internal_layout_table_file](https://lh6.googleusercontent.com/FuzdosP0tESOWRkrRsuBPH6XLAQMM3CGVCwJo17euwqPkEf9KXXsFbaj81lMTSKkiF42AqxtSuQCw91z-fjnuHueEcZ8IyKdFvzgphXqVCOkNaMtXiF2GC4NoIzfRpbIiBB_kI32)

Mỗi trang trong table sẽ bao gồm các loại dữ liệu sau:
- *heap tuple*: Mỗi heap tuple là một bản ghi dữ liệu (1 row). Chúng được sắp xếp từ phía cuối của trang.
- *line pointer*: Mỗi line pointer là một chuỗi `4 byte` chứa thông tin con trỏ của mỗi heap tuple. Các line pointer tạo thành một mảng đơn giản, mỗi line pointer sẽ có chỉ số xác định vị trí của line pointer trong mảng và được đánh tuần tự từ 1 và gọi là *offset number* của line pointer. Khác với các heap tuple, các line pointer sẽ được sắp xếp từ phía đầu của trang, bắt đầu từ sau thông tin *header data*. Khi 1 tuple mới được thêm vào trang, một line pointer mới sẽ được thêm vào để trỏ tới tuple này.
- *header data*: header data được định nghĩa bởi cấu trúc `PageHeaderData` và được đặt ở đầu của mỗi trang. Header data là 1 chuỗi `24 byte` chứa các thông tin cơ bản về trang:
    - *pd_lsn*: Đây là biến chứa thông tin LSN và XLOG được ghi bởi lần cuối thay đổi của tran. Nó là 1 số tự nhiên `8 byte` không âm, liên quan đến cơ chế WAL (Write-Ahead Logging)
    - *pd_checksum*: Biến này sẽ chứa giá trị checksum của trang
    - *pd_lower* và *pd_upper*: pd_lower là con trỏ trỏ tới vị trí cuối của các line pointers. pd_upper là con trỏ trỏ tới vị trí bắt đầu của tuple mới nhất.
    - *pd_special*: Trong 1 trang của table, nó chỉ tới vị trí kết thúc trang.

Khoảng không gian trống giữa vị trí kết thúc các line pointers và bắt đầu các tuple được gọi là *free space* hoặc *hole*.

PostgreSQL sử dụng *tuple identifier (TID)* để định danh cho 1 tuple trong table. Mỗi TID là một cặp giá trị gồm *block number* của trang chứa tuple và *offset number* của line pointer mà trỏ tới tuple đó. Để truy vấn TID của 1 tuple (1 row), ta có thể thêm `ctid` vào `select` trong câu lệnh truy vấn:

```
select ctid, id from sampletbl order by id asc limit 1;
  ctid     |  id
-----------+-------
 (0, 1)    | 1
(1 row)
```

Ở trong ví dụ trên, bản ghi của `sampletbl` có id là 1 sẽ nằm trong tuple ở trang có block number là 0 và line pointer trỏ đến nó trong trang này có offset number là 1

# 4. Đọc và ghi dữ liệu trong tuple

## 4.1. Ghi dữ liệu vào tuple

Giả sử table chỉ gồm 1 trang và chỉ chứa 1 tuple. `pd_lower` của trang sẽ trỏ đến vị trí line pointer đầu tiên và duy nhất của trang, `pd_upper` trỏ đến tuple đầu tiên và cũng là duy nhất của trang lúc này.

Khi một tuple thứ 2 được thêm vào, nó sẽ được đặt ngay sau tuple thứ nhất tính từ cuối trang lên, `pd_upper` sẽ được cập nhật đến vị trí của tuple thứ 2. Một con trỏ thứ 2 sẽ được sinh ra để trỏ đến tuple mới và được đặt sau con trỏ thứ nhất tính từ đầu trang xuống. `pd_lower` được cập nhật đến vị trí của con trỏ thứ 2 này.

![insert_data_to_tuple](https://lh5.googleusercontent.com/laTYY6hcu5L6kA5CKHJCmLjT7bY5Z2Yg0zgdhzONC9oWLrV-cXhnD3ZOx_JDa8zl2VPpxyOSr72loCp_v5tb5fC-HIwiMxlwSg0HWQEVj92sCWbtxnEGeGM0fnXdfXCLb9tLLG2e)

## 4.2. Đọc dữ liệu từ tuple

Để đọc dữ liệu từ tuple, thông thường sẽ sử dụng 2 cách là `sequential scan` và `B-tree index scan`:
- Sequential scan: Tất cả các tuple trong tất cả các trang được đọc một cách tuần tự bằng cách duyệt tất cả các line pointer trong mỗi trang.
- B-tree index scan: Mỗi index file bao gồm các *index tuple*. Mỗi *index tuple* là 1 cặp của *index key* và *TID* chứa thông tin của tuple có index key đó. Nếu index tuple với index key đang tìm kiếm được tìm thấy, PostgreSQL sẽ đọc thông tin của tuple dựa vào TID chứa trong index tuple.

![read_data_from_tuple](https://lh6.googleusercontent.com/HWLWXPNvDxBBG45F4d4fON9v7DDobs8Mznz9pOpA-IDPNrGUlfv2W_y5dgLt9amT085Z8lsOCJrEKi7c9bzAEUJDmnbLGbF54Tcpcf2xZ_5_5EsraYYu40PzxnFVZ0_gSEuI0AyW)

Ngoài ra, PostgreSQL còn hỗ trợ TID-Scan, Bitmap-Scan và Index-Only-Scan.

# 5. Tổng kết

Về cơ bản, dữ liệu ở tầng vật lý sẽ được PostgreSQL tổ chức thành các tệp và thư mục theo cấu trúc sau:
- Toàn bộ database cluster sẽ được lưu trữ trong một thư mục có đường dẫn tuỳ vào hệ điều hàng và thường được đặt trong biến môi trường `$PGDATA`. Trong thư mục database cluster, ngoài thư mục `base` chứa dữ liệu sẽ có các tệp và thư mục chứa các thông tin tổng quan cũng như chứa các tham số cấu hình của PostgreSQL
- Các database sẽ được lưu trữ thành các thư mục nằm dưới thư mục `base`. Mỗi thư mục sẽ được đặt tên theo `OID` của database
- Các table và index sẽ được lưu trữ trong các file của thư mục database mà chúng thuộc về. Mỗi file dữ liệu của table lại được chia nhỏ thành các page. Mỗi page sẽ có header data chứa thông tin tồng quan về toàn bộ page, danh sách các line pointers trỏ đến các tuple được đặt ở đầu page và các tuple chứa thông tin cúa từng bản ghi được đặt từ cuối page.


# Tài liệu tham khảo

[1] [http://www.interdb.jp/pg/pgsql01.html](http://www.interdb.jp/pg/pgsql01.html)

[2] [http://rachbelaid.com/introduction-to-postgres-physical-storage/](http://rachbelaid.com/introduction-to-postgres-physical-storage/)
