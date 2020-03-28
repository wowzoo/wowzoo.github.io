---
title: "coroutine을 사용해서 excel에서 데이터 수집하기"
layout: post
date: 2017-11-04 09:34
categories: python
---
최근 데이터 작업을 하다가 엑셀에 여러 시트가 있고 각 시트의 값들을 한 군데로 모아야 하는 상황이 생겼다.  
각 시트의 값들은 같은 키워드를 가지고 있으나 그 키워드에 대응하는 구성요소를 한 개씩만 가지고 있어서  
각 시트의 값들을 합쳐야 하나의 온전한 데이터가 된다.  

예를 들면 단어와 단어의 뜻, 단어의 발음이 있는데 한 시트에는 단어, 뜻, 발음이 다 있고, 한 시트에는 단어와 뜻 만,  
다른 시트에는 단어와 발음 만 있는 상태이다.   

각 시트를 돌며 데이터를 수집하는 방법은 여러 개가 있겠지만 이번에는 coroutine 을 사용해서 구현해 보았다.  
coroutine 을 사용한 이유는 coroutine 이 자신만의 데이터 공간을 가지고 있어서 수집한 데이터를 한곳에 모으기에 좋아서이다.  
물론 이건 class 를 이용한다던지, global 변수를 이용해도 되는 것이지만 coroutine 도 한가지 방법이라 생각해서 구현해 보았다.  

coroutine 의 개념을 알고 있다는 전제하에 글을 작성했다.  

## decorator 만들기
coroutine 을 이용하려면 항상 coroutine 을 선언한 후 next 를 이용해서 yield 까지 코드 포인트를 이동해 줘야 한다.  
```python
def gogodan():
    
    while True:
        value = yield

        for i in range(1, 10):
            print("{0} X {1} = {2}".format(value, i, value * i))

it = self.gogodan()
next(it) # 혹은 it.send(None) 를 써도 된다. 
it.send(3)
it.close()
```
```
3 X 1 = 3
3 X 2 = 6
3 X 3 = 9
3 X 4 = 12
3 X 5 = 15
3 X 6 = 18
3 X 7 = 21
3 X 8 = 24
3 X 9 = 27
```
> 입력한 단수에 따른 구구단을 출력하는 coroutine 을 만들었다고 했을 때  
> `it = self.gogodan()` 을 선언하고 나서 `next(it)` 을 호출하지 않으면 generator 를 시작하라고 에러가 난다.  
> `it = self.gogodan()` 선언한 직후 코드 포인트는 coroutine 의 시작부분인데 여기는 값을 받을 만한 구문이 없기 때문이다.  
> `next(it)` 나 `it.send(None)` 를 써서 yield 까지 코드 포인트를 이동해 줘야 한다.  

이렇게 coroutine 선언 후 반드시 next 를 불러줘야 하는데 이걸 신경쓰지 않고 next 를 호출하지 않는 실수를 없애기 위해서  
decorator 를 만들어 준다.

```python
def coroutine(func):
    def trigger(*args, **kwargs):
        it = func(*args, **kwargs)
        next(it)
        return it

    return trigger
```

이후 코드는 아래와 같이 변한다.

```python
@coroutine
def gogodan(self):

    while True:
        value = yield

        for i in range(1, 10):
            print("{0} X {1} = {2}".format(value, i, value * i))

it = self.coroutine_func()
it.send(3)
it.close()
```

## namedtuple 사용하기  

collections 의 namedtuple (`from collections import namedtuple`)은 쓰임새가 많은 tuple 이다.  
tuple 인데 각 항목에 이름을 지정할 수가 있고 class 멤버 변수에 접근하는 것처럼 이름으로 접근이 가능하다.  
```python
Word = namedtuple("Word", ("word", "pron", "meaning"))
w = Word(word="today", meaning="오늘", pron=None)
print(w.word)
```
namedtuple 은 tuple 이기 때문이 이 자체로 any 나 all 에 쓸수 있다.  
그래서 수집한 데이터에 빈 항목은 없는지 모두 다 잘 채워졌는지 확인하는데 그대로 any, all 에 넣어 사용할 수 있다.  

```python
w = Word(word="today", meaning="오늘", pron=None)
any(w)
$> True

all(w)
$> False
```

## coroutine 만들기
엑셀 핸들링 라이브러리로 [openpyxl] 을 사용하였다. 다른 엑셀 관련 라이브러리인 [XlsxWriter] 을 사용하는 것도 좋은 선택이다.  

```python
from collections import namedtuple
from openpyxl import load_workbook


@coroutine
def get_data(workers):
    while True:
        worksheet = yield
        worker = workers[worksheet.title]

        try:
            worker.send(worksheet)
        except StopIteration:
            print(worksheet.title, "End")
            worker.close()
```
> coroutine(iterator, generator) 은 더 이상 가져올 요소가 없으면 StopIteration Exception 을 발생시킨다.  
 
```python
@coroutine
def worker(converter, collector):
    worksheet = yield

    for row in worksheet.iter_rows(min_row=2):
        collector.send(converter([cell.value for cell in row]))
```
> 미리 lambda 로 정의해 둔 converter 를 이용해서 cell 의 값을 namedtuple 로 변형시킨다.  
> namedtuple 을 데이터를 수집하는 collector 로 보낸다.

```python
@coroutine
def word_collector():
    contents = {}

    while True:
        value = yield

        if value is None:
            yield contents

        data = contents.get(value.word, None)
        if data is None:
            contents[value.word] = value
        else:
            if value.pron is not None:
                data = data._replace(pron=value.pron)

            if value.meaning is not None:
                data = data._replace(meaning=value.meaning)

            contents[value.word] = data
```
> 데이터를 수집하는 collector 이다. worker 에서 namedtuple 을 보내줄 때 마다 dictionary 에 넣어준다.  
> 이때 같은 키워드를 갖는 namedtuple 이 있으면 검사해서 빈 부분(None 인 값) 을 채워준다.  
> tuple 은 한번 설정한 값을 바꿀 수 없듯이 namedtuple 도 값을 변경할 수 없다.  
> 빈 부분을 채워주기 위해서는 namedtuple 의 _replace 함수를 이용해서 새로운 namedtuple 을 만들어 준다.  

```python
workbook = load_workbook(filename='data.xlsx')

collector = word_collector()

Word = namedtuple("Word", ("word", "pron", "meaning"))

ws1 = workbook.get_sheet_by_name('word1')
ws2 = workbook.get_sheet_by_name('word2')
ws3 = workbook.get_sheet_by_name('word3')

workers = {
    ws1.title: worker(
        converter=lambda row: Word(word=row[0], pron=row[1], meaning=row[2]), collector=collector
    ),
    ws2.title: worker(
        converter=lambda row: Word(word=row[0], pron=None, meaning=row[2]), collector=collector
    ),
    ws3.title: worker(
        converter=lambda row: Word(word=row[0], pron=row[2], meaning=None), collector=collector
    )
}

it = get_data(workers=workers)

for worksheet in [ws1, ws2, ws3]:
    it.send(worksheet)
else:
    it.close()

data = collector.send(None)
collector.close()

print(data)
```
> namedtuple 을 정의하고 엑셀에서 어떤 시트를 가져올지 정해준다.  
> workers 라는 dictionary 를 만들고 각 시트마다 어느 cell 의 값을 넣어줄지 lambda 로 정의해 준다.    
> for-else 구문을 이용하여 get_data 를 닫아준다.  
> for-else 구문의 else 블럭은 for 문이 break 구문으로 중단되지 않고 끝까지 수행되었을 때 들어간다.  
> 각 시트마다 순회하여 데이터를 다 뽑아냈으면 collector 에 수집한 데이터를 달라는 신호(`it.send(None)`)를 보내서 값을 받아온다.

이상으로 coroutine 을 이용해서 엑셀의 여러 시트의 값들을 한 곳에 모으는 방법을 알아보았다.  

[openpyxl]: http://openpyxl.readthedocs.io/en/latest/
[XlsxWriter]: https://xlsxwriter.readthedocs.io