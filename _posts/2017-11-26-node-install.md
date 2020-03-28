---
layout: post
title: "Node 설치하기"
---
Node.js의 npm을 사용하기 위해 설치한다.  

# [NodeJS 홈페이지] 에서 각 OS에 맞는 인스톨 버전을 받아 설치  
이것이 가장 쉬운방법이지만 이렇게 하면 관리자 권한 문제와 Node 버전 관리가 힘들다.

### 관리자 권한 문제  
인스톨 버전을 설치하면 /usr/local 에 설치가 되는데 이렇게 설치가 되면 관리자 권한 없이는 제대로 실행되지 않는다.  
-g (global) 옵션을 사용하는 경우 항상 sudo를 해줘야 한다.  

### 버전 관리
[NodeJS 홈페이지] 에 가보면 깨닫겠지만 nodejs는 단일 버전으로 관리되지 않는다.  
프로젝트 별로 버전을 다른 것을 써야 할 경우에는 인스톨 버전을 설치하면 대응 방법이 없다.  

# zsh에서 nvm(node version manager) 을 사용하여 설치
https://github.com/lukechilds/zsh-nvm 에서 확인할 수 있다.  

1. zsh-nvm을 zsh에 plugin으로 설치한다.  
   git clone https://github.com/lukechilds/zsh-nvm ~/.oh-my-zsh/custom/plugins/zsh-nvm
2. ~/.zshrc 에서 plugins 항목을 찾아 zsh-nvm 을 추가한다.  
   plugins=(git zsh-nvm)
3. 쉘을 다시 시작하면 nvm이 설치된다.  
4. node를 설치한다.  
   $ nvm install --lts  
   이렇게 하면  LTS(Long Term Support) 버전을 설치한다.  
   v6.x.x 대 최신버전을 설치하면 지원하지 않는 모듈이 있어서 대부분의 사용자들에게 권장하는 LTS버전을 설치한다.  
5. npm으로 bower 설치해 보기  
   npm install -g bower

[NodeJS 홈페이지]: http://nodejs.org