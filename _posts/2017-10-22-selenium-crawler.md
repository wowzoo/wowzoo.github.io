---
title: "Selenium 으로 Web Crawler 만들기"
layout: post
date: 2017-10-22 08:35
categories: etc
---
[selenium] 은 웹 브라우저를 이용해 웹 애플리케이션을 테스트 할 수 있게 하는 tool 이다. 
자동화가 가능하고 여러 웹 브라우저(IE, Chrome, Firefox 등)를 이용할 수 있기 때문에 
웹 애플리케이션 테스트에 많이 이용한다.  

[selenium] 의 또 다른 쓰임새는 [webdriver] 를 이용해 web crawler 를 만드는데 있다.  
일반적으로 web crawling 은   
`웹 사이트 접속 -> crawling 할 페이지로 이동 (혹은 바로 crawling 할 페이지로 접속) -> html source 받기 -> html 분석 -> data 추출`  
의 과정을 거치는데 이러한 과정이 마치 [selenium] 으로 웹 사이트를 테스트하는 것과 비슷하고 [webdriver] 를 이용한 다면 
html element 에 접근하는 것도 쉽게 할 수 있어 web crawler 를 만드는데 [selenium] 을 사용하는 경우가 많다.  

여기서는 Mac 의 python 개발 환경에서 [selenium] 을 이용해 web crawler를 만들어 본다.  

## selenium 설치 하기  
```shell
pip install selenium
```

별거 없다. pip 를 이용해서 설치한다. 

## 웹 브라우저 선택하기  
selenium 의 webdriver python 소스를 살펴보면 지원하는 웹 브라우저로  
`android, blackberry, chrome, edge, firefox, ie, opera, phantomjs, safari`  
가 있다는 것을 알 수 있다.  

[webdriver] 는 웹 브라우저를 실제 구동하여 웹 사이트에 접속하고 html element 에 접근하는 것이기 때문에  
웹 브라우저도 미리 깔려 있어야 하지만 웹 브라우저를 제어할 driver 도 있어야 한다.  
driver 는 [webdriver] doc 페이지에서 링크를 발견 할 수 있다.

여기서는 chrome 을 이용하고 [chromedriver] 를 미리 받아 놓는다.

평소 Gmarket 에서 출첵을 통해 포인트를 조금씩 쌓아 놓는데 이걸 [selenium] 을 이용해 자동화 해 봤다.  
crawling 이라는 것은 결국 웹 사이트에 올려진 contents 를 스크랩하는 것이기 때문에 여기서 쓰는 방법들로  
html elements 에 접근하여 값을 받아오면 된다.  
 
## webdriver load
미리 받아 놓은 [chromedriver] 의 경로를 인자로 webdriver의 chrome 클래스를 호출 한다.
```python
from selenium import webdriver
from selenium.webdriver.common.by import By

class GmarketAttendance:
    def __init__(self):
        self._driver = webdriver.Chrome('chromedriver 가 있는 디렉토리 경로')
``` 

`chromedriver 가 있는 디렉토리 경로`는 꼭 절대경로일 필요는 없고 python 실행시 찾아들어 갈 수 있는 상대경로 된다.

## login 하기
```python
def sing_in(self):
    self._driver.get('http://www.gmarket.co.kr')

    login = self._driver.find_element(By.ID, 'css_login_box')
    login.click()

    user_id = self._driver.find_element(By.ID, 'id')
    user_id.send_keys('사용자 ID')
    user_pw = self._driver.find_element(By.ID, 'pwd')
    user_pw.send_keys('사용자 PW')

    submit = self._driver.find_element(By.XPATH, '//div[@class="btn-login"]/a')
    submit.click()
```

gmarket에 접속하여 로그인 페이지로 이동하고  
> 사이트에 접속하기 위해서는 url을 인자로 하는 get 을 호출한다.  
> `self._driver.get('http://www.gmarket.co.kr')`

로그인 페이지에서 사용자 ID 와 PW 를 입력할 폼을 찾아서 값을 넣어 준다.  
> 원하는 html elements 를 찾기 위해서는 먼저 해당 페이지의 소스를 봐야 한다.  
> chrome 의 개발자 도구를 이용해서 ID와 PW를 입력받는 input 폼을 찾아서 id 값을 확인하고  
> find_element 혹은 find_element_by_id 함수를 이용해서 element 를 찾는다.  
> `user_id = self._driver.find_element(By.ID, 'id')`  
> element 를 찾았으면 값을 입력한다.  
> `user_id.send_keys('사용자 ID')`  

값이 들어갔으면 submit 버튼을 찾아서 로그인 한다.  
> html 구조 에서 복잡한 경로의 html element 는 [xpath] 를 이용해서 찾을 수도 있다.  
> `submit = self._driver.find_element(By.XPATH, '//div[@class="btn-login"]/a')`  

## 출석 페이지로 이동하여 출석체크 하기  
```python
def attendance(self):
    coupon = self._driver.find_element(By.XPATH, '//div[@id="navbar"]/div[2]/span[2]/a')
    coupon.click()

    couponzone = self._driver.find_element(By.XPATH, '//div[@class="couponzone"]/a')
    couponzone.click()

    self._driver.switch_to.frame('AttendRulletFrame')
    button_start = self._driver.find_element(By.XPATH, '//div[@id="wrapper"]/a')
    button_start.click()
```

로그인 후 메인 페이지에서 출석 페이지를 찾아 이동하고  
> html elements 를 찾는 과정은 사용자 ID 와 PW 를 입력할 폼을 찾는 과정과 똑같다.  
> 일단 API 사용법에 익숙해 지면 그 다음은 html 코드를 읽어서 가장 효율적으로 목적하는 element 에  
> 접근하는 경로를 찾는 것이 crawler 를 만드는 기술이 된다.  
> `coupon = self._driver.find_element(By.XPATH, '//div[@id="navbar"]/div[2]/span[2]/a')`  

출석 페이지에서 출석체트 페이지로 간다음  
> `couponzone = self._driver.find_element(By.XPATH, '//div[@class="couponzone"]/a')`  

출석체크를 한다.  
> frame 으로 나누어진 html 의 경우 먼저 해당 frame 으로 옮겨간 다음 element 를 찾는다.  
> `self._driver.switch_to.frame('AttendRulletFrame')`    
> `button_start = self._driver.find_element(By.XPATH, '//div[@id="wrapper"]/a')`    

## driver 종료  
driver 를 다 사용했으면 닫아주어서 메모리에서 리소스를 내려야 한다. 항상 리소스 정리 함수 호출에 대해 염두해 두지 않고 원하는 동작을 마치고 나면
리소스 정리 함수 호출이 되는 것을 보장하려면 python 의 with 구문과 같이 사용하면 좋다.  
이 경우 코드 구현은 단순한 함수 호출이 아나리 클래스로 만들어 두어야 한다.  

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

class GmarketAttendance:
    def __init__(self):
        self._driver = webdriver.Chrome('chromedriver 가 있는 디렉토리 경로')
        
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        self._driver.quit()  
``` 

클래스가 with 구문에서 동작할 수 있게 한다.
> `__enter__` 와 `__exit__` 두 함수를 클래스에 구현하다.  
> `__enter__` 단순히 생성된 인스턴스를 넘겨준다.  
> `__exit__` with 구문을 빠져나갈 때 호출된다. 여기서 리소스 정리 함수를 호출한다. 
> [webdriver] 에서는 quit 이란 함수를 사용할 수 있다.

이렇게 만들어진 클래스는 아래와 같이 사용할 수 있다.  
```python
with GmarketAttendance() as gmarket:
    gmarket.sing_in()
    gmarket.attendance()
```

이렇게 간단히 [selenium] 을 이용한 crawling 에 대해서 살펴보았다.  



[selenium]: http://www.seleniumhq.org
[webdriver]: http://www.seleniumhq.org/docs/03_webdriver.jsp
[chromedriver]: https://sites.google.com/a/chromium.org/chromedriver/
[xpath]: https://www.w3schools.com/xml/xpath_syntax.asp