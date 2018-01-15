---
layout: post
title:  "Ngrok: Expose Localhost To The Internet"
date:   2018-01-14 18:25:01
author: Mars
categories: tools
---


### Ngrok: Expose Localhost To The Internet
https://ngrok.com/

간혹 모바일 기기로 로컬호스트에 접속하거나, 배포없이 다른 사람에게 보여주고 싶을 때가 있다.
이럴 때, Localhost 서버를 Tunneling 해주는 ngrok을 사용해보자.

다운로드 후에 `ngrok http PORT` 명령만 실행하면 접근 URL을 얻을 수 있다!
절차는 아래와 같다:
1. https://ngrok.com 에서 파일을 다운로드한다.
2. 압축을 푼다.
3. `ngrok http PORT` 명령어 실행

실행 결과로 아래와 같은 정보를 보여준다.
```
http://?.ngrok.io -> localhost:8082
https://?.ngrok.io -> localhost:8082
```
이제 ?.ngrok.io로 접속할 수 있다!


### Inspection, Replay, Subdomain ...
그 외에도 다양한 옵션들을 제공해준다.
* https://ngrok.com/docs/2

`localhost:4040`으로 접속하면 inspection 기능이나 replay 기능도 활용할 수 있다.
나는 아직 불필요하지만 ...

아쉽게도 도메인을 지정하는 기능은 유료 회원되어야 가능하다.
난 저렴하게 가야지.

