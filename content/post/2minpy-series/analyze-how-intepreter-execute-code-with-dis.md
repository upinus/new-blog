---
title: Phân tích cách interpeter thực thi code trong Python với dis
author: LongBB
date: 2019-12-06 17:48:00+07:00
tags: ['python', 'pyin2mins']
---

*Bài viết thuộc series [#2mins_with_python](/tags/pyin2mins)*

`dis` là một module đặc biệt bao gồm các function để làm việc với Python bytecode bằng cách minh hoạ bytecode bằng một form mà dễ đọc hơn với con người. Việc xem xét các bytecode được trình thông dịch thực thi là một cách tốt để có thể tối ưu hoá mã code.

Hàm `dis.dis()` sẽ in ra cách thức mà Python thực thi source code (có thể là module, class, function hoặc code object). Ví dụ khi ta khai báo hàm return về 1 dict:
```
import dis
test = lambda: {'a': 1}
dis.dis(test)
```
sẽ cho ra kết quả như sau:
```
  1           0 LOAD_CONST               1 ('a')
              2 LOAD_CONST               2 (1)
              4 BUILD_MAP                1
              6 RETURN_VALUE
```
Trong đó:
- Cột đầu tiên là thử tự của dòng trong source code.
- Cột thử 2 là địa chỉ của mã byte tương ứng với byte index
- Cột thứ 3 là tên lệnh. Mỗi tên sẽ được giải thích ngắn gọn ở [đây](https://docs.python.org/3.7/library/dis.html#python-bytecode-instructions)
- Argument (nếu có) của lệnhmaf thường được sử dụng một cách nội bộ bởi Python để lấy 1 số constant hoặc biến, quản lý stack  ...
- Argument ở dạng thân thiện với con người (human-readable)

`dis` có thể giúp chúng ta hiểu về cách thực thi mã code và giúp lý giải về kết quả thu được. Ví dụ với đoạn mã sau:
```
def test_func(a, b):
    a, b = b+1, a+b
    return (a, b)
print(test_func(5, 6))
```
Với đoạn mã trên ta thu được kết quả là `(7, 11)`. Để lý giải cho kết quả, khi sử dụng `dis.dis(test_func)` ta thu được đoạn mã sau:
```
  2           0 LOAD_FAST                1 (b)
              2 LOAD_CONST               1 (1)
              4 BINARY_ADD
              6 LOAD_FAST                0 (a)
              8 LOAD_FAST                1 (b)
             10 BINARY_ADD
             12 ROT_TWO
             14 STORE_FAST               0 (a)
             16 STORE_FAST               1 (b)

  3          18 LOAD_FAST                0 (a)
             20 LOAD_FAST                1 (b)
             22 BUILD_TUPLE              2
             24 RETURN_VALUE
```
Ở đây, ta có thể thấy, ở dòng code thứ 2, đầu tiên Python sẽ load biến `b`(giá trị là 6) và const `1`, sau đó cộng lại (giá trị là 6+1=7). Tiếp theo, Python sẽ load 2 biến `a` (giá trị là 5) và `b` (giá trị là 6) và cộng lại (6+7=11). Cuối cùng mới đem giá trị thu được gán lại cho 2 biến `a` và `b` thông qua lệnh `STORE_FAST`. Ở dòng lệnh thứ 3, Python lại load 2 biến `a` và `b`, build thành tuple và trả về kết quả nên kết quả cuối cùng của hàm sẽ là (7, 11)

`dis` là một công cụ mạnh và hay thường sử dụng khi debug các ***"one-line statement"*** như map hay list comprehension cũng như tối ưu hiệu năng của code dựa trên chiến lược thực thi của Python.

**Tham khảo**: [The Magic Behind One-Line Expressions in Python](https://medium.com/swlh/the-magic-behind-one-line-expressions-in-python-816c10180c5c)
