---
title: "Multiprocess로 Excel 읽기"
layout: post
date: 2017-12-03 12:03
categories: python
---
Excel의 데이터를 읽어서 분석할 때 몇 천 라인을 한 줄씩 읽어서 분석하다 보니 시간도 오래 걸리고 비효율적이라는 생각이 들었다.  
multiprocess를 이용하면 좀 더 빠르게 분석할 수 있을 것 같아서 구현해 보았다.  

```python
from openpyxl import load_workbook
from multiprocessing import cpu_count, current_process, Manager, Process, Value, Lock
from collections import namedtuple


class RetrieveData(object):
    def __init__(self, path, sheet_name):
        self._workbook = load_workbook(filename=path)
        self._worksheet = self._workbook.get_sheet_by_name(sheet_name)
```
> openpyxl을 이용해서 엑셀을 로드한다.  

```python
    def do(self):
        total = self._worksheet.max_row

        workers = cpu_count()
        quotient = total // workers

        Pointer = namedtuple("Pointer", ("start", "end"))
        bucket = []
        start_pointer = 2
        end_pointer = start_pointer + quotient
        for _ in range(workers):
            bucket.append(Pointer(start=start_pointer, end=end_pointer))
            start_pointer += quotient
            end_pointer += quotient
            end_pointer = end_pointer if end_pointer < total else total
```
> 엑셀의 전체 라인 수를 가져온다.  
> `total = self._worksheet.max_row`  
> cpu 개수를 가져와서 한 cpu 당 몇 개의 라인을 처리하면 될지 정한다.  
> `workers = cpu_count()`  
> `quotient = total // workers`  
> 엑셀에서 column header를 넣은 부분이 있을 수도 있으니 몇 번째 row 부터 읽을 지 정해준다.  
> `start_pointer = 2`

```python
        lock = Lock()
        cnt = Value('i', 0)
        previous = 0

        with Manager() as manager:
            wsd = manager.dict()

            procs = []
            for pointer in bucket:
                p = Process(target=self._worker, args=(pointer, wsd, cnt, lock,))
                p.start()
                procs.append(p)

            while cnt.value < total:
                if cnt.value != previous:
                    print('\r{0}'.format(cnt.value), end='\r')
                    previous = cnt.value

            for proc in procs:
                proc.join()

            words = set()
            for _, values in sorted(wsd.items()):
                words.update(values)

            print(len(words))
```
> 라인을 읽어서 처리해 줄 Process를 만들고 시작한다.  
> `p = Process(target=self._worker, args=(pointer, wsd, cnt, lock,))`  
> 라인이 처리되는 숫자를 보여준다.  
> `print('\r{0}'.format(cnt.value), end='\r')`  
> Process 가 종료되기를 기다린다.  
> `proc.join()`  
> Process 별로 처리된 데이터를 읽어온다.  
> `for _, values in sorted(wsd.items()):`

```python
    def _worker(self, pointer, wsd, cnt, lock):

        print(pointer)

        words = []
        for rows in self._worksheet.iter_rows(min_row=pointer.start, max_row=pointer.end):
            words.append(rows[1].value)

            with lock:
                cnt.value += 1

        wsd[current_process().name] = words
```
> 각 Process 가 엑셀에서 라인을 읽어서 처리하는 부분이다.  
> openpyxl 의 iter_rows 를 이용해서 읽을 라인의 시작과 끝을 정한다.  
> `for rows in self._worksheet.iter_rows(min_row=pointer.start, max_row=pointer.end):`
> 라인이 처리되는 숫자를 보여주기 위해 count 값을 업데이트 시킨다.  
> `cnt.value += 1`
> 처리된 데이터는 Process 이름을 key 로 하는 dictonary 에 넣어준다.  
> `wsd[current_process().name] = words`

```python
if __name__ == "__main__":
    rd = RetrieveData(path="엑셀 파일 경로", sheet_name="시트 이름")
    rd.do()
```
> 실행 할 때 엑셀 파일 경로와 시트 이름을 정해준다.  