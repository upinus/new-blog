---
title: Indentation style
author: LongBB
date: 2020-05-26 21:38:00+07:00
tags: ['clean_code']
---

Indentation style  một quy ước nhằm quản lý việc thụt đầu dòng của các khối lệnh trong cấu trúc một chương trình. Thụt đầu dòng thường không phải là yêu cầu với phần lớn các ngôn ngữ lập trình. Thay vào đó, việc thụt đầu dòng giúp code dễ đọc hơn, cấu trúc của chương trình rõ ràng hơn. Tuy nhiên, một số ngôn ngữ như python sử dụng thụt đầu dòng để xác định cấu trúc thay vì sử dụng cặp dấu ngoặc nhọn hoặc các keyword. Với các ngôn ngữ này, việc thụt đầu dòng có ý nghĩa đối với trình biên dịch chứ không chỉ là một kiểu coding style nữa.

Rất nhiều phần mềm trước đây sử dụng `tab` để thụt đầu dòng. Tuy nhiên, các trình soạn thảo trên Unix thường coi `tab` tương đương với 8 dấu cách trong khi các trình soạn thảo trên môi trường Window lại coi `tab` là 4 dấu cách dẫn đến những nhầm lẫn khi code được chuyển giữa các môi trường. Với các trình soạn thảo hiện đại bây giờ, chúng ta có thể cấu hình `tab` với số lượng dấu cách nên sự khác biệt này được khắc phục.

Các indentation styles phổ biến bao gồm: K&R style, Allman style, Whitesmiths style, GNU style, Hostmann style, Pico style, Ratliff style, Lisp style

## K&R style

K&R style (Kernighan & Ritchie Style) hay còn được gọi là “the one true brace style” (viết tắt là 1TBS) là phong cách thường được sử dụng trong C, C++ và các ngôn ngữ sử dụng dấu ngoặc nhọn. Với K&R, mỗi hàm sẽ có một dấu ngoặc nhọn mở ở ngay dòng dưới và cùng level thụt đầu dòng với định nghĩa hàm. Các lệnh bên trong thân hàm sẽ bị thụt đầu dòng hơn so với định nghĩa hàm và sẽ có một dấu ngoặc nhọn đóng có cùng level với dấu ngoặc mở và định nghĩa hàm. Các khối lệnh ở bên trong hàm sẽ có dấu ngoặc nhọn mở ở cùng dòng với lệnh điều khiển tương ứng (`if`, `else`, `for`, `while`, ...) và dấu ngoặc nhọn đóng của khối lệnh sẽ nằm trên 1 dòng khác, cùng level với ngoặc mở. Ví dụ:

```
int main(int argc, char *argv[])
{
    ...
    while (x == y) {
        something();
        somethingelse();

        if (some_error)
            do_correct();
        else
            continue_as_usual();
    }

    finalthing();
    ...
}
```
Các biến thể của K&R:

- 1TBS (OTBS): Hai điểm khác biệt chính so với kiểu K&R là các hàm có các dấu ngoặc nhọn mở trên cùng dòng, phân tách nhau bởi dấu cách và các dấu ngoặc nhọn không được phép bị loại bỏ kể cả khi câu lệnh điều khiển chỉ có một lệnh trong phạm vi của nó. Ưu điểm của style này là ngoặc mở không cần nhằm một mình trên 1 dòng mới, tuy nhiên ngoặc đóng lại cần 1 dòng một mình kể cả khi chỉ có 1 lệnh trong phạm vi của các khối lệnh điều khiển `if`, `else`, `while`, `do …while`
```
void checknegative(x) {
    if (x < 0) {
        puts("Negative");
    } else {
        nonnegative(x);
    }
}
```
- Linux kernel: đây là style được biết đến với việc sử dụng rộng rãi trong mã nguồn của Linux kernel. Style này sử dụng `tab stops` (với tab được thiết lập mỗi 8 ký tự) để indentation. Dấu ngoặc mở của hàm nằm ngay dưới định nghĩa hàm. Bất kỳ dấu ngoặc mở nào khác đều nằm trên cùng một dòng với câu lệnh tương ứng và được phân tách bởi dấu cách. Nếu phần thân của câu lệnh điều khiển (`if`, `while`, `do-while`) chỉ có 1 lệnh thì nó không cần được bao bởi cặp dấu ngoặc nhọn. Tuy nhiên, nếu có ít nhất 1 trong các câu lệnh trong cấu trúc `if-else` cần ngoặc nhọn thì tất cả các lệnh đều nên được bao trong các cặp ngoặc nhọn. Số ký tự tối đa trên 1 dòng nên là 80 ký tự.
```
int power(int x, int y)
{
    int result;

    if (y < 0) {
        result = 0;
    } else {
        result = 1;
        while (y-- > 0)
            result *= x;

    }
    return result;
}
```

- Stroustrup: đây là style do Bjarne Stroustrup tuỳ biến từ K&R style cho C ++ và được sử dụng trong những quyển sách của ông như “Programming: Principles and Practice using C++” và “The C++ Programming Language”. Không giống như các biến thể trước, Stroustrup không viết mệnh đề `else` sau dấu ngoặc đóng của mệnh đề `if`. Ngoài ra, trong Stroustrup, dấu ngoặc mở hàm sẽ được viết xuống mới còn dấu ngoặc mở class sẽ được viết trên cùng dòng với định nghĩa class. Stroustrup cho phép viết các hàm ngắn trên cùng 1 dòng.

```
   if (x < 0) {
        puts("Negative");
        negative(x);
    }
    else {
        puts("Non-negative");
        nonnegative(x);
    }
```

- BSD KNF: còn được gọi là Kernel Normal Form, đây là mẫu của phần lớn code được sử dụng trong hệ điều hành Berkeley Software Distribution (BSD). Trong style này, 8 khoảng trắng sẽ được sử dụng cho việc lùi đầu dòng các khối lệnh trong khi 4 khoảng trắng sẽ được sử dụng để lùi đầu dòng cho tất cả các câu lệnh dài bị chia làm nhiều dòng.

```
if (data != NULL && res > 0) {
        if (JS_DefineProperty(cx, o, "data",
            STRING_TO_JSVAL(JS_NewStringCopyN(cx, data, res)),
            NULL, NULL, JSPROP_ENUMERATE) != 0) {
                QUEUE_EXCEPTION("Internal error!");
                goto err;
        }
        PQfreemem(data);
} else {
        if (JS_DefineProperty(cx, o, "data", OBJECT_TO_JSVAL(NULL),
            NULL, NULL, JSPROP_ENUMERATE) != 0) {
                QUEUE_EXCEPTION("Internal error!");
                goto err;
        }
}
```

## Allman style

Phong cách này được đặt theo tên của Eric Allman. Nó cũng có thể được gọi là BSD style bời vì Allman đã viết rất nhiều ứng dụng cho BSD Unix. Style này đặt cặp ngoặc nhọn đóng mở của các lệnh điều khiển trên dòng riêng biệt cùng level với câu lệnh điều khiển. Các lệnh bên trong khối lệnh được thụt vào 1 level

```
while (x == y)
{
    something();
    somethingelse();
}

finalthing();
```

Kết quả của style này là các đoạn mã thực thi bên trong khối lệnh điều khiển được tách biệt rõ ràng. Việc comment hoặc loại bỏ một câu lệnh điều khiển hoặc khối lệnh sẽ ít có khả năng bị các lỗi cú pháp do việc bị thiếu dấu ngoặc đóng hoặc mở,. Ví dụ như đoạn mã sau vẫn đúng cú pháp:

```
// while (x == y)
{
    something();
    somethingelse();
}
```

## Whitesmiths style

Whitesmiths style hay còn được gọi là Wishart style, ban đầu được sử dụng trong tài liệu cho trình biên dịch C thương mại đầu tiên: trình biên dịch Whitesmiths.
Style này đặt cặp ngoặc đóng mở của câu lệnh điều khiển ở dòng riêng nhưng lại thụt vào 1 level so với câu lệnh điều khiển. Các lệnh trong khối lệnh của câu lệnh điều khiển nằm cùng level với cặp ngoặc đóng mở.

```
while (x == y)
    {
    something();
    somethingelse();
    }

finalthing();
```

Tương tự với Allman, style này có ưu điểm là các khối lệnh được đặt rõ ràng so với các câu lệnh điều khiển. Sự liên kết của dấu ngoặc với các khối lệnh nhấn mạnh rằng toàn bộ là một khối về mặt lập trình, là 1 câu lệnh ghép. Việc thụt 1 level của cặp dấu ngoặc có ý nghĩa rằng chúng phụ thuộc vào câu lệnh điều khiển.

## GNU style

Giống như Allman và Whitesmiths, GNU style đặt cặp dấu ngoặc nhọn đóng mở ở dòng riêng, thụt vào 2 khoảng trắng. Khi mở định nghĩa hàm, chúng sẽ không bị thụt đầu dòng. Các đoạn mã ở trong luôn luôn bị thụt vào 2 khoảng trắng so với dấu ngoặc.
```
static char *
concat (char *s1, char *s2)
{
  while (x == y)
    {
      something ();
      somethingelse ();
    }
  finalthing ();
}
```
Style này kết hợp ưu điểm của cả 2 kiểu Allman và Whitesmiths, GNU Coding Standards giới thiệu style này và gần như tất cả những maintainer của dự án GNU sử dụng nó.

## Hostmann style

Năm 1997, trong quyển “Computing Concepts with C++ Essentials” của Cay S.Horstman đã điều chỉnh Allman style bằng cách đặt câu lệnh đầu tiên của khối lệnh trong câu lệnh điều khiển trên cùng 1 dòng với dấu ngoặc mở.

```
while (x == y)
{   something();
    somethingelse();
    //...
    if (x < 0)
    {   printf("Negative");
        negative(x);
    }
    else
    {   printf("Non-negative");
        nonnegative(x);
    }
}
finalthing();
```

Style này thừa hưởng những ưu điểm của Allman bằng cách giữ sự thẳng hàng của các dấu ngoặc đóng mở cho dễ đọc và dễ dàng xác định các khối với việc lưu 1 dòng theo kiểu K&R.

## Pico style
Đây là style được sử dụng thông dụng nhất trong ngôn ngữ Pico. Pico thiếu các câu lệnh return và sử dụng dấu chấm phẩy để phân tách các câu lệnh:

```
stuff(n):
{ x: 3 * n;
  y: doStuff(x);
  y + x }
```

Ưu nhược điểm của style này giống như phong cách K&R.

## Ratliff style
Trong quyển sách “Programmers at Work”, C. Wayne Ratliff đã thảo luận việc sử dụng style này. Style này bắt đầu giống với 1TBS nhưng dấu ngoặc đóng lại có level giống với level của khối lệnh con. Style này đôi khi còn được gọi là “banner style”. Style này làm cho việc quét mã code có thể dễ dàng hơn do tiêu đề của bất kỳ khối lệnh nào là thứ duy nhất tồn tại ở level đó.

```
for (i = 0; i < 10; i++) {
     if (i % 2 == 0) {
         doSomething(i);
         }
     else {
         doSomethingElse(i);
         }
     }
```

## Lisp style
Các lập trình viên có thể thêm dấu ngoặc đóng vào dòng cuối cùng của khối lệnh. Style này giúp cho việc thụt đầu dòng là cách duy nhất để phân biệt các khối mã, nhưng có lợi thế là không chứa các dòng không có thông tin (các dấu ngoặc đóng mở không đứng ở dòng riêng biệt). Style này có thể được gọi là Lisp style (bởi vì style này rất phổ biến trong Lisp code) hoặc Python style (Python không sử dụng dấu ngoặc đóng mở, nhưng layout là rất giống. Trong Python, layout là một phần của ngôn ngữ)

```
// In C
for (i = 0; i < 10; i++)
    {if (i % 2 == 0)
        {doSomething(i);}
     else
        {doSomethingElse(i);
         doThirdThing(i);}}
```
```
# In Python
for i in range(10):
    if i % 2 == 0:
        do_something(i)
    else:
        do_something_else(i)
        do_third_thing(i)
```
```
;; In Lisp
(dotimes (i 10)
    (if (= (rem i 2) 0)
        (do-something i)
        (progn
            (do-something-else i)
            (do-third-thing i))))
```

## Tài liệu tham khảo
[1] [https://en.wikipedia.org/wiki/Indentation_style](https://en.wikipedia.org/wiki/Indentation_style)
