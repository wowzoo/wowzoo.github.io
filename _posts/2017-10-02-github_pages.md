---
title: "GitHub Pages와 Jekyll로 블로그 하기"
layout: post
date:   2017-10-02 08:23
categories: etc
---
Markdown으로 글을 쓰면서 블로그를 하고 싶다는 생각을 하고 있었는데 github pages와 jekyll을 이용하면 될 것 같아 만들어 보았다. 
이 HOW-TO는 Mac에서 Jekyll를 로컬로 세팅하고 post를 작성해서 github에 올리는 것에 대한 내용이다. 
git 계정은 이미 가지고 있다고 가정한다.     

## repository 만들기  
markdown으로 작성한 post를 올려놓을 repository를 만든다. 
*username*.github.io 와 같은 이름으로 repository 이름을 만든다.
**username 은 github의 계정이름과 같아야 한다.**  
[github pages]를 참고해서 만든다. git client는 terminal을 이용한다.

## ruby 버전 확인 및 설치  
Jekyll은 ruby 에서 실행되는데 ruby 버전이 2.1.0 이상이어야 한다.  
Jekyll 설치 전 먼저 ruby 버전을 확인해 보고 낮으면 업데이트 해준다.  
  
* ruby 버전을 확인한다.  
  ```shell
  $ ruby -v
  ruby 2.0.0p247 (2013-06-27 revision 41674) [universal.x86_64-darwin13]
  ```
  mac에 기본으로 설치된 ruby가 2.1.0 보다 낮으면 최신버전으로 업데이트 한다.    
* ruby 버전 업데이트  
  ```shell
  $ brew update
  $ brew install rbenv ruby-build
  ```  
  설치가 끝나면 아래 명령으로 rbenv를 실행한다.  
  ```shell
  $ eval "$(rbenv init -)"
  ```
  shell 환경에 ruby 초기화 명령을 넣어주면 터미널을 실행할 때 rbenv도 같이 실행되게 된다.
  기본 shell을 사용한다면 .bashrc, 다른 shell을 사용한다면 그 shell의 .xxxrc 파일에 아래 명령을 추가한다.
  ```shell
  if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
  ``` 
  ruby 최신 버전을 확인하고 설치해 준다.
  ```shell
  $ rbenv install --list
  Available versions:
    1.8.5-p52
    1.8.5-p113
    1.8.5-p114
    1.8.5-p115
    1.8.5-p231
    1.8.6
    ...
    2.4.2
    2.5.0-dev
    ...
  $ rbenv install 2.4.2
  $ rbenv rehash
  $ rbenv global 2.4.2
  ```
  다시 버전을 확인해서 2.4.2 가 지정되었는지 확인한다.
  ```shell
  $ ruby -v
  ruby 2.4.2p198 (2017-09-14 revision 59899) [x86_64-darwin16]
  ```

## Jekyll 설치하기  
Jekyll을 Mac에 설치하는 방법은 다양하지만 여기서는 [github의 가이드]를 따른다.  
요구사항의 ruby 버전은 위에서 확인했으니 bundler를 설치한다.  

```shell
$ gem install bundler
``` 

[github의 가이드]에는 jekyll local site를 위한 git repository를 만들라고 되어있는데 *username*.github.io를 local site로 이용하면서
.gitignore에 _site 를 추가하면 굳이 따로 local site를 만들 필요가 없을 것 같아 다른 방법으로 설치했다.  

1. 나중에 지울것이니 적당한 디렉토리에 temp 디렉토리를 만들고 디렉토리로 이동한다.
   ```shell
   $ mkdir temp
   $ cd temp
   ```
2. Gemfile을 아래와 내용으로 만든다.
   ```ruby
   source 'https://rubygems.org'
   gem 'github-pages', group: :jekyll_plugins
   ```   
3. bundle 을 이용해서 jekyll을 설치한다.
   ```shell
   $ bundle install
   Fetching gem metadata from https://rubygems.org/...........
   Fetching version metadata from https://rubygems.org/..
   Fetching dependency metadata from https://rubygems.org/.
   Resolving dependencies...
   Using i18n 0.8.6
   Using minitest 5.10.3
   ...
   ```
4. Jekyll template site를 temp 디렉토리에 만든다.  
   ```shell
   $ bundle exec jekyll new myblog
   ```
5. myblog 의 내용을 *username*.github.io 에 복사한다.  
   앞서 만들어 놓은 *username*.github.io 디렉토리로 가서 파일들을 지운다.
   ```shell
   $ cd *username*.github.io
   $ rm -rf *
   ```
   myblog로 이동하여 모든 파일을 *username*.github.io 에 복사한다.  
   
   다시 *username*.github.io로 이동하여 Gemfile을 연다.
   Gemfile의 내용 중 
   ```shell
   # This will help ensure the proper Jekyll version is running.
   # Happy Jekylling!
   gem "jekyll", "3.5.2"
   ```
   의 `gem "jekyll", "3.5.2"` 를 지운다.  
   ```shell
   # This will help ensure the proper Jekyll version is running.
   # Happy Jekylling!
   ```   
   그리고
   ```shell
   # If you want to use GitHub Pages, remove the "gem "jekyll"" above and
   # uncomment the line below. To upgrade, run `bundle update github-pages`.
   # gem "github-pages", group: :jekyll_plugins
   ```
   에서 `# gem "github-pages", group: :jekyll_plugins`의 주석을 푼다.
   
   ```shell
   # If you want to use GitHub Pages, remove the "gem "jekyll"" above and
   # uncomment the line below. To upgrade, run `bundle update github-pages`.
   gem "github-pages", group: :jekyll_plugins
   ```
6. .gitignore 파일 편집하기  
   *username*.github.io 디렉토리에서 .gitignore 파일을 연다. 없으면 myblog에서 복사해 온다.  
   내용이 아래와 같으면 된다.
   ```shell
   _site
   .sass-cache
   .jekyll-metadata
   .DS_Store
   Gemfile.lock
   ```   
7. temp 디렉토리 지우기  
   jekyll은 temp 디렉토리에 설치되는 것이 아니라 rbenv 가 관리하는 디렉토리에 설치된다.  
   ```shell
   예) /Users/username/.rbenv/versions/2.4.2/lib/ruby/gems/2.4.0/gems/minima-2.1.1
   ```
   그래서 jekyll을 설치하고 *username*.github.io 에 Jekyll template site의 내용을 복사했으면 
   더 이상 필요가 없다.  
   
## git repository와 연결하기  
*username*.github.io 로 이동해서 git repository를 커밋한다.
```shell
$ cd *username*.github.io
$ git add .
$ git commit -m "updated site"
$ git push -u origin master
```

## post 만들기  
Jekyll template site를 성공적으로 *username*.github.io 디렉토리에 복사하였다면 
_posts 디렉토리가 있을 것이다. *username*.github.io에 올라가는 post는 _posts 디렉토리에 위치해야 한다. 
이미 예제 파일이 하나 있을 텐데 새롭게 하나 만들어 본다. 
파일이름은 2017-10-02-hello_word.md 로 한다.
파일이름 규칙은 [post 작성하기] 를 읽어본다.

```markdown
---
title: "Hello World"
layout: post
date:   2017-10-02 08:23
categories: etc
---
# Hello World  
이것은 테스트 입니다.
```
시작부분의 title, layout, date, categories 같은 머리말들은 [YAML 머리말]를 참고한다.


[github pages]: https://pages.github.com
[github의 가이드]: https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/
[post 작성하기]: http://jekyllrb-ko.github.io/docs/posts/
[YAML 머리말]: http://jekyllrb-ko.github.io/docs/frontmatter/