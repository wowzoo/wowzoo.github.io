---
title: "Crawling에 Headless Chrome 사용하기"
layout: post
date: 2017-10-29 11:25
categories: etc
---
## Headless Chrome
이전 포스팅에서 [selenium 으로 crawling 하는 법]에 대해서 알아보았다.  
[webdriver] 는 웹 브라우저를 실제 구동하는 것이기 때문에 chrome 이나 firefox, ie 등을 쓰면  
화면상에 브라우저가 떠서 사이트를 이동하고 값이 자동으로 입력되는 모습들이 보일 것이다.  

웹 사이트의 자동화 테스트가 목적이라면 테스트 되고 있는 모습이 보이는 것도 좋겠으나 crawling 을 할 때는 굳이 화면이  
보일 필요는 없다. 더불어 linux 같이 GUI 보다는 terminal 에서 많은 작업이 이루어 진다거나 실행 속도를 조금 더 빨리하고  
싶다면 CLI 환경에서 crawling 이 이루어져야 할 것이다. 이러한 목적에 부합하는 것으로 [headless browser] 가 있다.

[headless browser] 는 GUI 가 없이 동작하는 브라우저를 말하는데 대표적인 것으로 [phantomjs] 가 있다.  
여기서는 가장 많이 쓰는 브라우저 중 하나인 chrome 의 headless 버전인 [headless chrome] 으로 이전 포스팅의  
코드를 업데이트 해보겠다.

## chrome 옵션 설정하기  
이미 chrome 으로 동작하도록 되어 있기 때문에 chrome 구동 옵션을 headless 로 바꿔주면  
다른 코드를 바꾸지 않고도 동작한다.  

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

class GmarketAttendance:
    def __init__(self):
        chrome_driver = ''chromedriver 가 있는 디렉토리 경로''
        chrome_options = webdriver.ChromeOptions()
        chrome_options.add_argument('--headless')
        chrome_options.add_argument('--disable-gpu')
        self._driver = webdriver.Chrome(
            executable_path=chrome_driver,
            chrome_options=chrome_options
        )
``` 

chrome 구동 시 [chromedriver] 경로 외에 추가로 옵션을 설정해 준다.  
> `--headless` chrome 을 headless 모드로 동작하게 해 준다.  
> `--disable-gpu` Mesa library 가 없는 경우 발생하는 에러를 피하기 위해 임시로 설정된 옵션인데 추후에는 필요없다고 한다. 

이렇게 설정하면 된다. 코드의 다른 부분은 수정할 필요가 없다.  
headless chrome 의 다른 옵션이나 사용법에 대해서 더 자세히 알고 싶으면 [Getting Started with Headless Chrome] 을 참고한다.
 

[selenium 으로 crawling 하는 법]: https://wowzoo.github.io/etc/2017/10/22/selenium-crawler.html
[selenium]: http://www.seleniumhq.org
[webdriver]: http://www.seleniumhq.org/docs/03_webdriver.jsp
[chromedriver]: https://sites.google.com/a/chromium.org/chromedriver/
[headless browser]: https://en.wikipedia.org/wiki/Headless_browser
[phantomjs]: http://phantomjs.org
[headless chrome]: https://chromium.googlesource.com/chromium/src/+/lkgr/headless/README.md
[Getting Started with Headless Chrome]: https://developers.google.com/web/updates/2017/04/headless-chrome