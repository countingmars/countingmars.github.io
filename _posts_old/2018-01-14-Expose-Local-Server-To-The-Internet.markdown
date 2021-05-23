---
layout: post
title:  "Ngrok: Expose Localhost To The Internet"
date:   2018-01-14 18:25:01
author: Mars
categories: tools
---


### Ngrok: Expose Localhost To The Internet
<https://ngrok.com/>

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

`localhost:4040` 으로 접속하면 inspection 기능이나 replay 기능도 활용할 수 있다.
나는 아직 불필요하지만 ...

아쉽게도 도메인을 지정하는 기능은 유료 회원되어야 가능하다.
난 저렴하게 가야지.


### Problems with Ngrok
몇 가지 문제가 발생했다.

#### Invalid Host Header
나의 로컬 서버에 ngrok 을 사용해서 접속했더니, 짠~ 하고 `Invalid Host Header` 라는 오류 메세지가 떴다.

구글링을 해보니 몇 가지 솔루션이 나오는데 다음과 같았다.
1. ngrok http 8080 -host-header="localhost:8080"
2. ngrok http --host-header=rewrite 8080
3. devServer.disableHostCheck = true

나는 webpack.config.js 에서 `devServer { disableHostCheck: true }` 를 사용해서 해결했다.

#### Too Slow Response
첫 요청에서 너무 느려서 좀 빠르게 써볼려고 다음과 같이 시도해봤다.
1. `-inspect=false` 옵션을 적용해봤는데도 여전히 느리네 ...
2. `-region=ap` 옵션을 적용해봤는데 그다지 빠르단 느낌은 없네 ...

그마나 두번째 요청부턴 좀 빨라지니 참고 써야겠지?



