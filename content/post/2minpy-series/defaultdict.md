---
title: defaultdict trong Python
author: LongBB
date: 2019-12-05 15:13:00+07:00
tags: ['python', 'pyin2mins']
---

*Bài viết thuộc series [#2mins_with_python](/tags/pyin2mins)*

`defaultdict` trong python:

- `defaultdict` là một dictionary mà trả về giá trị mặc định tuỳ theo khai báo kiểu dữ liệu ban đầu khi không tìm thấy key trong dict.
Ví dụ với `defaultdict` là int thì giá trị mặc định trả về sẽ là 0 nếu không có key:
```
>>> from collections import defaultdict
>>> a = defaultdict(int)
>>> a['a']
0
```
- `defaultdict` sẽ có tác dụng trong các trường hợp như phân loại hoặc đếm tần suất xuất hiện. Ví dụ như gom nhóm theo shop từ 1 danh sách order cho trước (có thể lấy từ file csv chẳng hạn)
```
orders = [('shop1', 'order_info1'), ('shop2', 'order_info2'), ('shop1', 'order_info3')]
```
Nếu không sử dụng `defaultdict` ta sẽ có thể code như sau:
```
shop_dict = {}
for order in orders:
    shop = order[0]
    if shop not in shop_dict:
        shop_dict[shop] = []
    shop_dict[shop].append(order[1])
```
Nếu sử dụng `defaultdict`, code sẽ gọn gàng hơn nhiều:
```
from collections import defaultdict
shop_dict = defaultdict(list)
for order in orders:
    shop_dict[order[0]].append(order[1])
```
- `dict` và `defaultdict` có thể convert qua lại lẫn nhau:
```
>>> from collections import defaultdict
>>> a = defaultdict(int)
>>> a['b'] = 1
>>> a
defaultdict(<class 'int'>, {'b': 1})
>>> b = dict(a)
>>> b
{'b': 1}
>>> c = defaultdict(int, b)
>>> c
defaultdict(<class 'int'>, {'b': 1})
```

**Tham khảo**: [Python Pro Tip: Start using Python defaultdict and Counter in place of dictionary](https://towardsdatascience.com/python-pro-tip-start-using-python-defaultdict-and-counter-in-place-of-dictionary-d1922513f747?gi=5879e703468d)
