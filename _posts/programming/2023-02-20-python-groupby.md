---
layout: post
title: 'itertools.groupby로 연속 된 값 단위 분리'
categories: ['Python']
---

아래와 같이 3일, 2일 단위로 연속된 date를 이어지는 단위(consecutive)로 분리하고자 할 때
```python
from datetime import date
from itertools import groupby

today = timezone.now().date()
dates = [today + timedelta(days=i) for i in range(3)]
dates += [today + timedelta(days=i + 5) for i in range(2)]

# dates
[
    # group1
    datetime.date(2023, 2, 16),
    datetime.date(2023, 2, 17),
    datetime.date(2023, 2, 18),
    # group2
    datetime.date(2023, 2, 21),
    datetime.date(2023, 2, 22)
]
```

date자체는 연속 된 값으로 인식되지 않으므로 키로 사용할 값을 생성
```python
# date.isocalendar()로 year, week, weekday값을 가져와 일(day)기준 값을 생성
def dt_function(dt):
    c = dt.isocalendar()
    return c.year * 365 + c.week * 7 + c.weekday

data = [(dt, dt_function(dt)) for dt in dates]

# data (우측 값이 일 단위 기준 값)
[(datetime.date(2023, 2, 16), 738448),
 (datetime.date(2023, 2, 17), 738449),
 (datetime.date(2023, 2, 18), 738450),
 (datetime.date(2023, 2, 21), 738453),
 (datetime.date(2023, 2, 22), 738454)]
```

enumerate로 1씩 증가하는 index값에서 키로 사용할 값을 빼주면 연속 된 값들은 같은 키 값을 갖게 되므로 groupby결과에서 그룹으로 리턴됨.
```python
>>> for k, g in groupby(enumerate(dates), key=dt_function):
        print(k, list(g))
    
-738448 [
  (0, datetime.date(2023, 2, 16)), 
  (1, datetime.date(2023, 2, 17)), 
  (2, datetime.date(2023, 2, 18))
]
-738450 [
  (3, datetime.date(2023, 2, 21)),
  (4, datetime.date(2023, 2, 22))
]
```
