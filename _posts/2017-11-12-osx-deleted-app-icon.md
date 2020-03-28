---
title: "MAC에서 삭제한 앱의 아이콘이 계속 보일때"
layout: post
date: 2017-11-12 09:03
categories: mac
---
*5KPlayer* 란 프로그램을 깔았는데 이걸 지우고 나서도 이것과 연결되었던 파일들의 아이콘이 계속 *5KPlayer*의 것으로 나왔다.  
정보보기에서 연결프로그램을 바꿔도 안되서 정보를 검색하던 중 **_아래 명령어로 아이콘 캐시를 지우면 된다_**고 해서 해봤는데 드디어  
아이콘이 제대로 나온다.

```shell
$ sudo find /private/var/folders/ \
  -name com.apple.dock.iconcache -exec rm {} \;
$ sudo find /private/var/folders/ \
  -name com.apple.iconservices -exec rm -rf {} \;
$ sudo rm -rf /Library/Caches/com.apple.iconservices.store
```

이렇게 하고 **_재부팅_** 해줘야 한다.  

