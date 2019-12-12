---
title: Python3 tìm kiếm thuộc tính trong class như thế nào?
author: LongBB
date: 2019-06-29 11:38:00+07:00
tags: ['python']
---


Nếu là người đã từng làm việc với Python thì chắc hẳn chẳng ai lạ gì cú pháp `C.x` để truy cập thuộc tính `x` của class `C`. Tuy nhiên, làm thế nào để Python có thể tìm biết được class `C` có tồn tại thuộc tính `x` hay không và nếu có tồn tại thì thuộc tính `x` này sẽ được lấy ra từ đâu khi mà Python cho phép đa kế thừa ? (Giả sử trong trường hợp `C` kế thừa 2 classs và cả 2 class base của `C` đều có thuộc tính `x`, vậy `C` biết nghe theo ai, bố mẹ hay ông hàng xóm)
Bài viết dưới đây sẽ trình bày một cách ngắn gọn về cách mà Python tìm kiếm một thuộc tính của class (hoặc instance) khi được yêu cầu. Tuy nhiên, trước khi đi vào phần chính này, ta cần phải biết một khái niệm cơ bản trong Python là `descriptor`.

**Chú ý:** Nội dung bài viết được áp dụng cho **Python3**, có thể không chính xác cho **Python2** và các phiên bản cũ hơn.

## 1. Descriptor là gì? Các loại descriptor
### 1.1. Khái niệm về descriptor
`Descriptor` là bất kỳ đối tượng nào trong Python mà cung cấp 1 phương thức đặc biệt là `__get__`. Đối với class, khi có một thuộc tính là descriptor thì ta có thể kiểm soát việc truy xuất thuộc tính này thông qua phương thức đặc biệt là `__get__` và `__set__` của descriptor. Nói cách khác, khi truy cập một thuộc tính của class, Python sẽ tiến hành lấy giá trị của thuộc tính bằng cách gọi hàm `__get__` trên descriptor tương ứng (nếu tồn tại).

**Ví dụ 1:**
```python
class TestDescriptor(object):
	def __init__(self, value):
		self.value = value
	def __get__(self, *_):
		return self.value**2
class TestClass(object):
	x = TestDescriptor(2)

test_instance = TestClass()
print(test_instance.x) # 4
```

### 1.2. Các loại descriptor
Như đã trình bày ở phần trên, bất kỳ đối tượng nào cung cấp phương thức `__get__` đều được gọi là `descriptor`. Tuy nhiên, nếu descriptor này cung cấp thêm phương thức `__set__` thì descriptor này sẽ được gọi là `overriding descriptor`. Ngược lại, nếu không có phương thức `__set__` thì descriptor sẽ được gọi là `nonoverriding descriptor`. Ở ví dụ trên, `TestDescriptor` là một `nonoverriding descriptor`.
Đối với `overriding descriptor`, phương thức `__set__` giúp quản lý việc nhập giá trị đầu vào. Nếu 1 class có 1 thuộc tính là `overriding descriptor`, khi nhập giá trị cho thuộc tính đó, Python sẽ thiết lập giá trị của thuộc tính bằng cách gọi phương thức `__set__` của thuộc tính (nếu có).
Vậy là ta đã hiểu về khái niệm descriptor trong Python, sau đây chúng ta sẽ đi đến phần chính cùa bài viết: tìm hiểu về cách thức tìm kiếm thuộc tính trong Python.

## 2. Cách thức tìm kiếm thuộc tính của Python
### 2.1. Cách thức tìm kiếm thuộc tính trong class của Python
Python là một ngôn ngữ hường đối tượng và đương nhiên cho phép việc thực thi tính kế thừa trong hường đối tượng. Vì vậy, một class trong Python thay vì chỉ có các thuộc tính của nó mà nó sẽ có thể truy cập cả các thuộc tính của lớp cha (lớp base) nữa (Thuộc tính ở đây bao gồm cả các`callable attributes` hay còn gọi là các `methods`).
Việc xác định thuộc tính `x` khi gọi `C.x` sẽ rất đơn giản trong trường hợp chỉ có đơn kế thừa. Lúc đó, nếu thuộc tính không tồn tại ở lớp con sẽ được tìm dần lên các lớp base của nó, cứ như vậy cho đến khi tìm thấy thuộc tính `x`. Tuy nhiên, Python cho phép đa kế thừa nên cần có độ ưu tiên giữa các class base.
Để có thể tìm kiếm một thuộc tính `x` của class `C` trong môi trường đa kế thừa, Python thực hiện theo chiến lược gồm 2 bước sau:

- **Bước 1:** Kiểm tra xem `x` có phải là key của `C.__dict__` hay không (Thông thường, 1 class sẽ lưu trữ các thông tin về các attribute của nó trong 1 dictionary với key là các attribute của class và value tương ứng là giá trị của các attribute đó. Python cho phép truy cập dictionary đó thông qua một attribute đặc biệt gọi là `__dict__`). Nếu `x` đúng là một key của `C.__dict__`, Python sẽ lấy ra giá trị `v` của `x` từ `C.__dict__`: `v = C.__dict__[x]`. Nếu `v` là một `descriptor` thì kết quả của lời gọi `C.x` là:
```python
C.x = type(v).__get__(v, None, C)
```
- **Bước 2:** Nếu `x` không phải là key nằm trong `C.__dict__`, Python sẽ tiếp tục tìm kiếm trên các lớp base của `C`. Quá trình lặp lại cho đến khi tìm được thuộc tính `x`. Nếu khi lặp qua tất cả các class tổ tiên của `C` mà vẫn không tìm được thuộc tính `x` thì Python sẽ đưa ra lỗi `AttributeError`.

Python thực hiện tìm kiếm thuộc tính `x` trong các class base của `C` ở bước 2 theo thứ tự ***"left to right, depth first"***. Ta có thể tưởng tượng thứ tự ưu tiên của tìm kiếm của Python tương tự với duyệt cây theo chiều sâu (`DFS`) với gốc là class gọi tìm kiếm thuộc tính, các node phía dưới là các lớp base của node phía trên.
Để có thể hiểu về thứ tự này, ta có theo dõi ví dụ sau:

**Ví dụ 2:**
```python
class A(): x = 'a'
class B(): x = 'b'
class C(A): pass
class D(C, B): pass
```
Khi call `D.x`, đầu tiên do ta không thực hiện định nghĩa thuộc tính `x` trong `D` nên `x` sẽ không nằm trong `D.__dict__`
```python
>>> 'x' in D.__dict__
False
```
Do đó, Python sẽ chuyển sang tìm tại các class base của `D`. `D` có 2 class base là `C` và `B`. Theo như thứ tự ưu tiên ở trên, class base ở bên trái sẽ được ưu tiên trước nên `C` sẽ được ưu tiên tìm trước (`C` ở bên trái trong khi ta định nghĩa class `D`). Tương tự như `D`, `C` không định nghĩa thuộc tính `x` nên `x` cũng sẽ không nằm trong `C.__dict__`.
```python
>>> 'x' in C.__dict__
False
```
Lúc này, theo thứ tự ta sẽ ưu tiên tìm đến class `A` trước chứ không phải `B` (***depth first***). `A` có định nghĩa thuộc tính `x` nên `x` sẽ nằm trong `A.__dict__`.
```python
>>> 'x' in A.__dict__
True
>>> A.__dict__['x']
'a'
```
Giá trị của thuộc tính `x` của class A là string `'a'`, không phải là `descriptor` nên `A.x = 'a'`. Do đó, `D.x` sẽ trả về giá trị của `A.x` là `'a'` và quá trính tìm kiếm kết thúc.
```python
>>> D.x
'a'
>>> id(D.x) == id(A.x)
True
```
Tuy nhiên, trong nhiều trường hợp, nếu chỉ đơn thuần sử dụng thứ tự ***"left to right, depth first"*** thì sẽ gặp phải tính huống 1 class bị duyệt qua 2 lần. Để có thể tránh tình trạng này, Python sẽ chỉ duyệt qua các class bị trùng vào lần xuất hiện cuối cùng của chúng, các lần trước đó sẽ bị bỏ qua trong quá trình duyệt (***rightmost occurrence***).

**Ví dụ 3:**
```python
class A(): x = 'a'
class B(A): x = 'b'
class C(A): pass
class D(C, B): pass
```
Ở ví dụ trên ta thấy, nếu chỉ duyệt theo thứ tự ***"left to right, depth first"*** thì thứ tự khi tìm thuộc tính `x` trên class `D` sẽ là: `D, C, A, B, A`. Nhưng `A` sẽ bị duyệt 2 lần nên lần duyệt đầu tiên sẽ bị bỏ qua và thứ tự tìm thuộc tính `x` trên class `D` sẽ là: `D, C, B, A`. Thực hiện theo các bước như ví dụ 2, kết quả `D.x` là `B.x` và là `'b'` chứ không phải là `'a'` như ví dụ 2.
```python
>>> D.x
'b'
>>> id(D.x) == id(B.x)
True
```
Quy tắc ***“left to right, depth first, rightmost occurrence”*** được gọi là `Method Resolution Order`, viết tắt là `MRO`. Python đã xây dựng một phương thức đặc biệt trong class là `__mro__` để đưa ra thứ tự duyệt các class base dựa theo quy tắc này.
```python
>>> D.__mro__
(<class '__main__.D'>, <class '__main__.C'>, <class '__main__.B'>, <class '__main__.A'>, <class 'object'>)
```
Với phương thức `__mro__`, ta có thể viết đoạn code thực thi việc tìm kiếm thuộc tính trong class như sau:
```python
def get_attribute(class_object, attribute_name):
    classes_order = [class_object]
    classes_order.extend(list(class_object.__mro__))
    for class_base in classes_order:
        if attribute_name not in class_base .__dict__:
            continue
        value = class_base .__dict__[attribute_name]
        try:
            return type(value).__get__(value, None, class_object)
        except Exception:
            return value
    raise AttributeError('class {} has no attribute \'{}\''.format(class_object.__name__, attribute_name))
```
### 2.2. Cách thức tìm kiếm thuộc tính trong instance của Python
Cách thức tìm kiếm thuộc tính trong instance của Python cũng gần tương tự với cách tìm kiếm thuộc tính trong class. Cụ thể, khi tìm kiếm thuộc tính `x` của instance `i` của class `C`, Python sẽ tiến hành theo 3 bước sau:

- **Bước 1:** Tìm kiếm trên class `C` (hoặc các class tổ tiên của `C`) thuộc tính `x`. Nếu tìm thấy `x` và giá trị `v` của `x` là overriding descriptor (`x` có cả phương thức `__get__` và `__set__`) thì giá trị của việc gọi `i.x` là:
```python
type(v).__get__(v, i, C)
```
- **Bước 2:** Nếu x không thỏa mãn các điều kiện ở bước 1, kiểm tra xem `x` có thuộc `i.__dict__` (trong một số trường hợp `i` không có thuộc tính `__dict__`, giả sử như class `C` định nghĩa thuộc tính `__slots__` thì `i` sẽ chỉ có `__slots__` mà không có `__dict__`, lúc này việc tìm kiếm sẽ diễn ra trên` __slots__`). Nếu `x` thuộc `i.__dict__` thì kết quả của phép gọi `i.x` là `i.__dict__[x]` (bất kể giá trị của `v = i.__dict__[x]` có phải là `descriptor` hay không - điều này khác với class)
- **Bước 3:** Nếu `x` không thuộc dictionary `i.__dict__`, thực hiện tìm kiếm `x` trên các class của `C` như bước 1. Lúc này, giá trị trả về sẽ là `type(v).__get__(v, i, C)` với `v` là giá trị của thuộc tính `x` tìm được và `v` là `descriptor`, ngược lại, giá trị trả về chính là `v` nếu `v` không phải là `descriptor`.

**Ví dụ 4:**
```python
class E():
    def __get__(self, *_):
        return 1
    def __set__(self, *_):
        pass
class F():
    x1 = E()
    x2 = 2

i = F()
i.x1 = 3
i.x2 = 4
print(i.x1) # 1 not 3
print(id(i.x1) == id(F.x1)) # True
print(i.x2) # 4
```
Ở ví dụ trên, ta thấy class `F` có thuộc tính `x1` là 1 `overriding desciptor` (thuộc tính này là instance của class `E` có cả 2 phương thức là `__get__` và `__set__`) và thuộc tính `x2` là number `2`. Khởi tạo instance `i` của class `F`, gán lại giá trị cho các thuộc tính `x1` và `x2` của `i` lần lượt là `3` và `4`. Tuy nhiên, sau đó, khi ta truy cập lại thuộc tính `x1` và `x2` của `i` thông qua `i.x1` và `i.x2` thì chỉ có `x2` thay đổi được giá trị còn `x1` vẫn giữ nguyên là giá trị của `overriding descriptor` tại class `F`.
## 3. Kết luận
Qua bài viết này chúng ta đã có thêm những kiến thức về cách mà Python3 thực hiện việc tìm kiếm thuộc tính trong class cũng như instance. Xa hơn, chúng ta có thể thấy được cách mà Python thực hiện tính kế thừa trong lập trình hướng đối tượng: thực tế, lớp con không hề có các thuộc tính của lớp cha, khi gọi các thuộc tính của lớp cha thông qua lớp con, Python sẽ tìm kiếm và trả về các object thuộc về lớp cha.
