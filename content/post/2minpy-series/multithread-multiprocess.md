---
title: Multi-Threading và Multi-Processing trong Python
author: LongBB
date: 2019-11-15 10:50:00+07:00
tags: ['python', 'pyin2mins']
---

*Bài viết thuộc series [#2mins_with_python](/tags/pyin2mins)*

Multi-Threading và Multi-Processing trong Python:

- Các tác vụ **IO-bound** là các tác vụ dành phần lớn thời gian để chời đợi kết quả phản hồi từ các hệ thống khác như network, database, file hoặc thậm chí là user
- Các tác vụ **CPU-bound** là các tác vụ dành phần lớn thời gian của chúng để tính toán trong CPU
- Với các tác vụ **IO-bound**, ta có thể sử dụng `multithreading` để  tăng hiệu năng. Tuy nhiên với các tác vụ **CPU-bound**, việc sử dụng `multithreading` sẽ có thể làm suy giảm hiệu năng. Nguyên nhân của vấn đề này là do Python có một cơ chế để quản lý thread gọi là **Global Interpreter Lock (GIL)**. GIL sẽ luôn đảm bảo rằng chỉ có 1 thread được thực thi tại một thời điểm bất kỳ để tránh việc 2 thread có thể cùng truy cập và tương tác với 1 object (với multithreading, bộ nhớ được chia sẻ nên có thể dẫn đến tình trạng nhiều thread cùng tranh chấp tương tác với object tại một thời điểm). Với các tác vụ **IO-bound**, GIL không ảnh hưởng nhiều do 1 thread chỉ thực hiện request và đợi kết quả phản hồi nên trong thời gian chờ đợi thread khác sẽ được chuyển vào thực hiện các request, nhờ đó mà tối ưu được thời gian chờ đợi, dẫn đến tăng hiệu năng. Còn với các tác vụ **CPU-bound**, mỗi thread sẽ phải thực thi tính toán đến khi hoàn tất rồi mới chuyển sang thread khác, dẫn đến quá trình thực thi không khác gì tuần tự. Tuy nhiên, khi sử dụng `multithreading` còn phát sinh chi phí hoạt động và quản lý các luồng nên về cơ bản hiệu suất sẽ bị giảm nếu ta sử dụng `multithreading` cho các tác vụ **CPU-bound**
- Để có thể tăng hiệu năng của các tác vụ **CPU-bound**, thay vì sử dụng `multithreading`, ta có thể sử dụng `multiprocessing`. Mỗi một processing là một chương trình với bộ nhớ độc lập (không xảy ra tình trạng 2 process cùng tranh chấp 1 object trong bộ nhớ như thread). Về bản chất, mỗi process sẽ sử dụng một CPU khác nhau để làm việc cùng 1 lúc, do đó có thể xử lý đồng thời nhiều tác vụ **CPU-bound**. Tuy nhiên, có 1 lưu ý là khi sử dụng `multiprocessing` không phải cứ tăng số lượng process là sẽ tăng hiệu năng, thậm chí khi sử dụng quá nhiều process, hiệu năng còn bị suy giảm so với sử dụng đủ số process. Do mỗi process sẽ sử dụng 1 CPU khác nhau để hoạt động đồng thời nên số process có thể hoạt động đồng thời tối đa bằng số luồng của CPU. Việc sử dụng nhiều process hơn số luồng của CPU sẽ không tăng hiệu năng do cùng thời điểm cũng chỉ có tối đa số process bằng số luồng của CPU hoạt động. Mặt khác, việc sinh nhiều process dẫn đến chi phí quản lý process cao hơn làm suy giảm hiệu năng.
- Với các tác vụ **IO-bound**, việc sử dụng `multiprocessing` cũng có thể làm tăng hiệu năng nhưng sẽ không hiệu quả bằng `multithreading`. Nguyên nhân là do khi sử dụng `multiprocessing`, lượng process đồng thời bị giới hạn bời phần cứng nên các tác vụ có thời gian chờ cao sẽ không tối ưu bằng việc sử dụng `multithreading` với nhiều luồng, đồng thời mỗi process sẽ là một chương trình riêng biệt với bộ nhớ riêng nên việc quản lý cũng tốn chi phí hơn. Do vậy, với các tác vụ **IO-bound** nên sử dụng `multithreading` thay vì `multiprocessing`.

**Tham khảo**:

- [The Why, When, and How of Using Python Multi-threading and Multi-Processing](https://medium.com/towards-artificial-intelligence/the-why-when-and-how-of-using-python-multi-threading-and-multi-processing-afd1b8a8ecca)
- [What is the Python Global Interpreter Lock (GIL)?](https://realpython.com/python-gil/)
