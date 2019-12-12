---
title: Lưu ý khi làm việc với datetime trong Python
author: LongBB
date: 2019-11-08 12:06:00+07:00
tags: ['python', 'pyin2mins']
---

*Bài viết thuộc series [#2mins_with_python](/tags/pyin2mins)*

Một vài lưu ý khi làm việc với datetime trong Python:

- Các object datetime là immutable. Do đó các hàm tương tác với object datetime như replace sẽ không thay đổi object datetime ban đầu mà trả về 1 object datetime mới.
- Đặc biệt chú ý về vấn đề múi giờ của object datetime thông qua thuộc tính tzinfo. Nếu tzinfo không được set, python sẽ tự động lấy thông tin về múi giờ tại local. Điều này sẽ gây ra rất nhiều lỗi phát sinh nếu không hiểu rõ vấn đề khi phát triền như ở máy cá nhân code vẫn chạy đúng nhưng lên server lại bị lệch thời gian (máy cá nhân sẽ ở múi giờ +7 còn server thường là +0). Luôn luôn thiết lập tzinfo cho thời gian.
- Để thiết lập tzinfo, mình thường sử dụng hàm `replace` kết hợp với `pytz`. Ví dụ:
```python
from datetime import datetime
import pytz

now = datetime.now()
timeoffset = pytz.FixedOffset(7 * 60) # +7
now = now.replace(tzinfo=timeoffset)
```
- Rất nhiều hàm của datetime sẽ return về  thời gian ở  múi giờ local  như  `now`(lấy thời gian ở hiện tại), `fromtimestamp`  (chuyển timestamp về datetime). Cần tránh sử dụng các hàm kiểu này do khi deploy múi giờ local tại server sẽ khác với khi develop tại máy cá nhân. Sử dụng các hàm convert về utctime để thay thế , như `utcnow ` lấy thời gian hiện tại ở múi giờ +00:00 hay `utcfromtimestamp` để chuyển timestamp về datetime ở múi giờ `+00:00`. (Lưu ý là convert xong thì vẫn phải thiết lập tzinfo bằng hàm replace nhé vì kết quả của các hàm này sẽ  trả về object datetime nhưng lại không có tzinfo đâu).
- Để chuyển thời gian về các múi giờ khác nhau, mình thường sử dụng `astimezone` kết hợp với `pytz` . Ví dụ sau convert 1 timestamp về múi giờ +7:
```python
from datetime import datetime
import pytz

time = 1572293772
utc_datetime = datetime.utcfromtimestamp(time).replace(tzinfo=pytz.utc)
hanoi_timeoffset = pytz.FixedOffset(7 * 60)
hanoi_datetime = utc_datetime.astimezone(hanoi_timeoffset)
```
