---
title: List Comprehension trong Python
author: LongBB
date: 2019-11-13 11:00:00+07:00
tags: ['python', 'pyin2mins']
---

*Bài viết thuộc series [#2mins_with_python](/tags/pyin2mins)*

List comprehension là một trong những tính năng đặc biệt của python cho phép tạo ra một list một cách "thông minh" với chỉ một dòng code. List comprehension có thể sử dụng được trong nhiều trường hợp khác nhau, có thể sử dụng để thay thế cho cả `map` và `filter`, tạo ra các đoạn mã ngắn gọn dễ đọc và dễ hiểu.

Tuy nhiên, không phải lúc nào cũng nên áp dụng list comprehension một cách bừa bãi. Chúng có thể làm cho code chạy chậm hơn hoặc sử dụng nhiều bộ nhớ hơn:

- Cẩn thận khi sử dụng nested comprehensions: Khi có nhiều hơn 1 for trong comprehension, nên suy nghĩ đến phương pháp dùng vòng lặp để đoạn code trở nên dễ hiều và dễ debug hơn
- Sử dụng generator cho dữ liệu lớn: Với các lists có kích thước nhỏ và trung bình (cỡ vài nghìn phần tử), việc sử dụng list comprehension nhìn chung là chấp nhận được. Tuy nhiên với các tác vụ xử lý trên một lượng dữ liệu lớn như cộng 1 tỷ integer thì việc sử dụng list comprehension không phải là một phương án tốt do Python sẽ cố gắng tạo ra 1 list gồm 1 tỷ integer trước và việc này sẽ tốn rất nhiều bộ nhớ, thậm chí có thể gây tràn nhớ, crack chương trình. Thay vào đó, nên sử dụng generator. Generator không tạo ra 1 cấu cấu trúc lớn trong bộ nhớ mà trả về iterable sử dụng lazy evaluation: iterable sẽ loại bỏ giá trị cũ đồng thời tính toán rồi trả về giá trị mới mỗi khi được gọi và code sẽ yêu cầu giá trị tiếp theo từ iterable đến khi gặp `StopIteration` Exception. Quá trình này giữ cho bộ nhớ luôn nhỏ.
- Trong các trường hợp cần hiệu năng, list comprehension không phải lúc nào cũng là một lựa chọn tốt. Có thể sử dụng nhiều phương án khác nhau như loop, map hoặc list comprehension và sử dụng các công cụ đo đạc để có được phương án phù hợp nhất.

**Tham khảo**: [When to Use a List Comprehension in Python](https://realpython.com/list-comprehension-python/#when-not-to-use-a-list-comprehension-in-python)
