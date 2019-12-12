---
title: is và ==
author: LongBB
date: 2019-11-26 11:32:00+07:00
tags: ['python', 'pyin2mins']
---

*Bài viết thuộc series [#2mins_with_python](/tags/pyin2mins)*

`is` và `==`:

- `is` được sử dụng để so sánh 2 object trong Python. Python quản lý bộ nhớ bằng cách lưu trữ tất cả các object vào 1 private heap (Heap ở đây có thể hiểu đơn giản là một mảng bộ nhớ hoặc 1 mảng dữ liệu khổng lồ). Mỗi khi 1 object được gán cho 1 biến (variable), biến sẽ refer đến địa chỉ của object đó trong heap. Ở đây, việc so sánh 2 object sử dụng `is` chính là việc so sánh địa chỉ của object trong heap
- `==` được sử dụng để so sánh giá trị của object.
- Các số nguyên từ -5 đến 256 sẽ được CPython lưu trữ trước trong 1 mảng gọi là `small_ints` để có thể tối ưu về mặt quản lý bộ nhớ với các số nguyên nhỏ. Khi gán 1 biến cho 1 số thuộc `small_ints`, CPython sẽ không tạo ra 1 không gian nhớ mới cho số này mà refer thẳng biến này đến địa chỉ bộ nhớ lưu trữ số đã tồn tại từ trước của `small_ints`. Vì vậy, lúc này ta so sánh 2 số bằng `is` hay `==` đều cho ra kết quả là `True`:
```
>>> a = 5
>>> b = 5
>>> a is b
True
>>> a == b
True
```
- Tuy nhiên, với các số tự nhiên nằm ngoài khoảng của `small_ints`, CPython sẽ tạo ra 1 không gian nhớ mới và refer biến đến địa chỉ của không gian nhớ này. Lúc này, sử dụng `is` để compare 2 số sẽ trả về `False` chứ không còn trả về giá trị `True`
```
>>> a = 257
>>> b = 257
>>> a is b
False
>>> a == b
True
>>> id(a)
140328281627728
>>> id(b)
140328281627680
```
- Tương tự với string, các string nhỏ (với Python3 có độ dài dưới 4096 ký tự) có cùng giá trị sẽ được CPython lưu trữ trong cùng 1 không gian nhớ mà không tạo thêm không gian nhớ mới khi có 1 biến khác refer đến. Lúc này, sử dụng `is` và `==` để so sánh 2 string nhỏ sẽ cùng cho giá trị `True`. Tuy nhiên, với các string lớn hơn, CPython sẽ tạo ra các bộ nhớ riêng biệt dẫn đến việc sử dụng `is` để so sánh sẽ cho giá trị `False`. Nếu hiểu rõ vấn đề này, đây có thể là 1 tip để tăng tốc độ cho Python khi phải so sánh các string nhỏ và đã biết trước độ dài như usku trong hệ thống chẳng hạn (sử dụng `is` chỉ so sánh địa chỉ nên sẽ nhanh hơn việc duyệt string để so sánh value bằng `==`) nhưng cũng sẽ là con dao 2 lưỡi nếu sử dụng để so sánh các string mà ta không biết trước kích thước.
```
>>> a = 'a' * 4096
>>> b = 'a' * 4096
>>> a is b
True
>>> a == b
True
>>> c = 'a' * 4097
>>> d = 'a' * 4097
>>> c is d
False
>>> c == d
True
```
**Warning**: Việc sử dụng `is` để compare có thể sẽ gây ra những hậu quả cực kỳ nguy hiểm nếu ta không hiểu rõ nó. Ví dụ như so sánh id trong database. Với môi trường development, id hầu hết là nhỏ nên việc sử dụng `is` sẽ cho ra kết quả `True` nhưng ở trên môi trường production, khi dữ liệu tăng lên liên tục, các id sẽ vượt ngoài tầm của `small_ints`, lúc này `is` sẽ trả về `False` và gây ra lỗi logic rất khó để phát hiện như trong bài viết [dưới đây](https://medium.com/peloton-engineering/the-dangers-of-using-is-in-python-f42941124027).

**P/S (#magicpython)**: Nếu thay đổi giá trị lưu trữ tại địa chỉ của các số thuộc `small_ints` khi đang chạy chương trình, ta sẽ có 1 bất ngờ nho nhỏ :v
```
>>> import ctypes
>>> def mutate_int(an_int, new_value):
...     ctypes.memmove(id(an_int) + 24, id(new_value) + 24, 8)
...
>>> mutate_int(7, 13)
>>> list(range(1, 20))
[1, 2, 3, 4, 5, 6, 13, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
>>> 7
13
>>> 6 + 1
13
>>> a = 7
>>> a + 1
14
```
