---
title: "coroutine 을 pipeline 처럼 이용하기"
layout: post
date: 2017-11-19 08:48
categories: python
---
텍스트 파일을 읽어서 특수문자로 표시된 따옴표, 쉼표, 하이픈 등을 일반문자로 변경하고 UTF-8 로 저장하는 모듈이 필요해 졌다.  
어떻게 만들까 고민하다 coroutine 을 이용해서 pipeline 처럼 구성해 보기로 했다.  

## Encoding 자동 인식  
텍스트 파일을 에디터로 열어보면 문자가 깨져서 보이는 경우가 있다.  
특히 한글인 경우는 EUC-KR, CP949 같은 다양한 인코딩 방식으로 저장된 파일들이 많은데 맞는 인코딩 방식으로 파일을 열지 않으면  
문자는 깨져서 보이게 된다. 대부분의 에디터들은 파일을 열고난 후 사용자가 인코딩을 바꿀 수 있도록 지원하기 때문에  
깨진 문자가 보이는 경우 `알맞은 인코딩을 찾아서 바꿔`주면 정상적으로 문자가 보이게 된다.  

그런데 `알맞은 인코딩을 찾아서 바꿔`주는 것을 코드로 구현하려면 쉽지가 않다.  
왜냐하면 사람이 눈으로 보고 이것이 맞는 인코딩인지 아닌지 인지하는 과정을 구현해야 하기 때문이다.  
다행이 이걸 구현한 모듈([chardet])이 있다. 이걸 이용해서 텍스트 파일을 알맞는 인코딩으로 읽어들이도록 한다.  

```shell
pip install chardet
```
> 먼저 라이브러리를 설치한다.  

```python
import os
import glob

from parsers import Document
from chardet.universaldetector import UniversalDetector


if __name__ == "__main__":

    src = "텍스트 파일이 있는 디렉토리 경로"
    dst = "수정한 파일을 저장할 디렉토리 경로"

    files = glob.glob(os.path.join(src, '*.txt'))
    detector = UniversalDetector()
    
    for f in files:
        print(f)
        detector.reset()
        with open(f, 'rb') as ff:
            for line in ff:
                detector.feed(line)
                if detector.done:
                    break

        detector.close()
        print("Encoding:", detector.result['encoding'])

        f_name = os.path.basename(f)
        print(f_name)

        doc = Document(src=f, dst=os.path.join(dst, f_name), encoding=detector.result['encoding'])
        doc.adjust()
```
> chardet 에서 UniversalDetector 를 import 하여 인스턴스를 만들고  
> 파일을 binary 모드로 열어서 어떤 인코딩이 쓰였는지 확인한다.  
> 확인한 인코딩을 인자로 해서 변환 클래스를 생성한다.  

## 변환 클래스 만들기  
```python
import os
import re


class Document(object):
    def __init__(self,  src=None, dst=None, encoding='UTF-8'):
        self.src = os.path.normpath(src)
        self.dst = os.path.normpath(dst)
        self.encoding = encoding
```        
> 파일 경로와 저장할 경로, 그리고 파일 인코딩을 인자로 받는 클래스를 만든다.  
> 인코딩은 default 로 UTF-8을 지정한다.  

```python
    def adjust_line(self, line_write):
        # 연속으로 공백이 들어간 부분은 한 개로 한다
        pattern = re.compile(r"\s{2,}")

        line = yield

        while True:
            # 문장의 따옴표, 쉼표 등의 특수문자를 일반문자로 변경한다.
            edited = line.replace("，", ",")

            for sym in ["’", "‘", "’", "`"]:
                edited = edited.replace(sym, "'")

            for sym in ["“", "”"]:
                edited = edited.replace(sym, '"')

            for sym in ["—", "―", "ㅡ", "─", "－"]:
                edited = edited.replace(sym, " - ")

            edited = pattern.sub(" ", edited)

            line = yield line_write.send(edited)

    def write_lines(self, dst):
        with open(dst, mode='w', encoding='utf-8') as f:
            while True:
                line = yield
                if line is not None:
                    f.write(line + "\n")
```
> 두 개의 coroutine을 만든다.  
> adjust_line : 한 줄을 받아서 각종 특수 문자를 일반 문자로 변경하고 변경된 결과를 write_lines 에 보낸다.  
> write_lines : 변경된 결과를 파일로 저장한다. 파일은 UTF-8 로 한다.  

```python
    def adjust(self):
        line_write = self.write_lines(self.dst)
        next(line_write)

        line_adjust = self.adjust_line(line_write)
        next(line_adjust)

        with open(self.src, 'r', encoding=self.encoding) as f:
            for line in f:
                line_adjust.send(line.strip())

        line_write.close()
        line_adjust.close()
```
> 두 개의 coroutine 을 생성하고 두 coroutine 을 연결(`self.adjust_line(line_write)`) 한다.  
> 파일을 열어서 한 줄씩 coroutine 에 보낸다.

[chardet]: https://github.com/chardet/chardet