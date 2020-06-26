---
title: Pure Function
author: LongBB
date: 2020-06-26 21:18:00+07:00
tags: ['clean_code']
---


# 1. Giới thiệu về pure function
Trong lập trình, một pure function là một hàm mà có những tính chất sau:

- Hàm luôn luôn trả về cùng một giá trị với cùng một dữ liệu đầu vào
- Thực thi không có hiệu ứng phụ. Hiệu ứng phụ ở đây đề cập đến việc thay đổi các thuộc tính khác của chương trình không nằm trong hàm như thay đổi giá trị biến toàn cục hoặc sử dụng luồng I/O.

Giá trị trả về của một pure function chỉ phụ thuộc vào đầu vào của nó mà không hề phụ thuộc hay chịu ảnh hưởng vào yếu tố nào khác trên chương trình.

Pure function có tư tưởng gần tương tự với các hàm toán học. Với bất kỳ giá trị đầu vào nào, một pure function phải trả về chính xác chỉ một giá trị có thể. Tuy nhiên, một pure function được phép trả về cùng một giá trị cho các đầu vào khác nhau.

Ví dụ:
```
def add_1(x):
    return x + 1
```
Hàm trên đây là một pure function. Nó không có hiệu ứng phụ nào và luôn trả về cùng giá trị đầu ra với cùng giá trị đầu vào
```
import random

def f(x):
    if random.randint(1, 2) == 1:
        return x + 1
    return x + 2
```
Hàm trên không phải là pure function bởi vì hàm sẽ không trả về cùng một giá trị với cùng một đầu vào mà còn phụ thuộc cả vào giá trị được random trong `random.randint(1, 2)`
```
y = 1

def f(x):
    global y
    y += 1
    return x + 1
```
Tương tự, hàm trên cũng không phải là một pure function mặc dù với cùng một giá trị đầu vào thì hàm luôn trả về cùng một giá trị đầu ra. Nguyên nhân là do trong thân hàm có hiệu ứng phụ làm thay đổi biến toàn cục `y`.

# 2. Lợi ích của pure function

Khi sử dụng các pure function ta sẽ thu được một vài lợi ích cả về hiệu suất và khả năng sử dụng:

- Dễ đọc: tất cả yêu cầu về dữ liệu đầu vào của hàm đều được cung cấp dưới dạng tham số nên sẽ không có các yếu tố ảnh hưởng đến kết quả đầu ra ngoài các dữ liệu đầu vào. Điều này giúp chúng ta có thể dễ dàng hiểu được hàm và các yếu tố mà hàm phụ thuộc chỉ bằng cách đọc định nghĩa hàm. Ví dụ với một pure function được định nghĩa là `f(a, b, c)` thì chúng ta có thể hiểu rằng hàm `f` chỉ  phụ thuộc vào `a`, `b` và `c` mà không phụ thuộc thêm vào bất kỳ yếu tố nào khác
- Khả chuyển: Do tất cả các yếu tố phụ thuộc đều nằm trong tham số đầu vào nên 1 function có thể có những cách sử dụng khác nhau tuỳ thuộc vào yếu tố đầu vào. Điều này làm cho code khả chuyển và có thể tái sử dụng vào các ngữ cảnh khác nhau thay vì phải viết các hàm khác nhau chỉ để sử dụng cho các cách thực thi khác nhau của cùng 1 class. Ví dụ, để có thể ghi log vào các level khác nhau, ta có thể viết 1 pure function nhận logger làm dữ liệu đầu vào thay vì việc viết 2 function khác nhau cho các logger khác nhau.
- Dễ kiểm thử: Việc không có hiệu ứng phụ làm cho các pure function rất dễ viết test. Chúng ta chỉ việc đưa các giá trị input để kiểm tra các giá trị đầu ra mong muốn mà không cần thiết phải lo lắng về việc duy trì trạng thái cục bộ xuyên suốt các bài kiểm tra của chúng ta.
- Tham chiếu minh bạch: Tham chiếu minh bạch đề cập đến việc có thể thay đổi một lệnh gọi hàm bằng giá trị đầu ra tương ứng của hàm mà không thay đổi hành vi của chương trình. Để đạt được điều này, hàm phải là pure function. Điều này có lợi ích về việc dễ đọc code và hiệu năng của hệ thống. Trình biên dịch thường tối ưu code mà có sự tham chiếu minh bạch.
- Caching: Pure function luôn luôn trả về cùng một đầu ra với cùng một giá trị đầu vào nên chúng ta có thể cache kết quả của pure function. Một pure function `f: input -> output` có thể được caching thành 1 hash map từ `input` sang `output`. Khi thực thi hàm, đầu tiên ta kiểm tra xem hash map có tồn tại inout như là key hay không. Nếu có thì chúng ta sẽ trả về giá trị chứa trong hash map tương ứng với key input, ngược lại, chúng ta sẽ tính `f(input)` rồi lưu trữ `output` vào hash map và trả về giá trị `output`

# 3. Tổng kết

Pure function là những hàm luôn trả về cùng một giá trị đầu ra với cùng một giá trị đầu vào và việc thực thi hàm không có hiệu ứng phụ. Việc sử dụng các pure function giúp cho code dễ đọc, khả chuyển, dễ kiểm thử, tham chiếu mình bạch và có thể caching.

# Tài liệu tham khảo
[1] [https://medium.com/better-programming/what-is-a-pure-function-3b4af9352f6f](https://medium.com/better-programming/what-is-a-pure-function-3b4af9352f6f)
