---
layout: post
title:  "Spring Cloud Zuul And Ribbon"
date:   2018-05-02 13:25:01
author: Mars
categories: tools
---

원본 <https://spring.io/guides/gs/routing-and-filtering/>


### Routing and Filtering
Netflix Zuul과 Ribbon을 사용해서 request를 라우팅하고 필터링해보자. 

### What you’ll build
우선 간단한 마이크로서비스를 하나 만들고, 
Netflix Zuul로 reverse proxy 앱을 만들어서 받은 요청을 마이크로서비스로 보내보자.
  
### Set up a microservice
아주 간단한 Book 서비스를 만들자.

book/build.gradle을 만들어주자.
핵심은 spring-boot-starter-web 의존성을 추가하는 것이다.

```
dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
``` 

이제 스프링 앱을 개발하자.
BookApplication.java 파일은 이런 형태일 것이다: 

```
// book/src/main/java/hello/BookApplication.java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class BookApplication {

  @RequestMapping(value = "/available")
  public String available() {
    return "Spring in Action";
  }

  @RequestMapping(value = "/checked-out")
  public String checkedOut() {
    return "Spring Boot in Action";
  }

  public static void main(String[] args) {
    SpringApplication.run(BookApplication.class, args);
  }
}
```
BookApplication 클래스는 REST controller이고, 
@RequestMapping을 통해 요청에 대해 적절히 응답을 줄 것이다.

@RequestMapping은 `available()`, `checkedOut()` 메서드에 적용되어 있고, `/available`와 `/checked-out` 경로에 대해
간단한 문자열을 반환한다.


설정 파일에 어플리케이션 이름을 book이라고 지정해주자. 
```
# src/main/resources/application.properties.
spring.application.name=book
server.port=8090
```
서버 포트를 8090으로 설정해서 뒤에 구현할 edge 서비스와의 포트 충돌을 피했다.

### Create an edge service
이제 edge 서비스를 개발하자.
Spring Cloud Netflix는 embedded Zuul proxy을 포함하고 있다.

gateway/build.gradle 파일의 핵심은 아래의 의존성 설정이다.
```
compile('org.springframework.cloud:spring-cloud-starter-zuul')
compile('org.springframework.boot:spring-boot-starter-web')
```
	 
@EnableZuulProxy 어노테이션을 사용하면 어플리케이션을 reverse proxy로 만들어주며, 요청을 다른 서비스로 포워딩해준다.

GatewayApplication.java는 아래의 형태이다.
``` 
// gateway/src/main/java/hello/GatewayApplication.java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {
  public static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
  }
}
```
요청을 포워딩할려면, Zuul에게 route(요청을 포워딩하는 맵핑 설정)에 대해 알려줘야 한다.
application.properties의 zuul.routes 속성에 설정할 수 있다.

각각의 마이크로서비스는 자신의 route 엔트리로 `zuul.routes.어플리케이션명`을 가진다.
```
# gateway/src/main/resources/application.properties
zuul.routes.books.url=http://localhost:8090
ribbon.eureka.enabled=false
server.port=8080
```
Spring Cloud Zuul은 자동으로 마이크로서비스 이름(application name)에 대한 경로를 설정하지만, 
이 예제에서는 `zuul.routes.books.url`에 대한 URL을 지정했으므로 
`/books`에 대한 요청은 http://localhost:8090으로 전달될거다.

설정파일의 2번째 속성은 Zuul이 Netflix Ribbon을 사용해서 
클라이언트 사이드 로드 밸런싱을 수행하겠다는 것을 의미한다.

그리고 Ribbon은 디폴트로 Netflix Eureka(service discovery)를 사용하는데
이 예제에서는 Eureka를 설정하지 않았으므로, 설정파일의 3번째 속성 `ribbon.eureka.enabled`를 false로 지정해서
eureka를 사용하지 않도록 했다. 따라서 `url` 설정은 필수인 것이다.


### Add a filter
이제 요청을 필터링해보자. 
Zuul은 4개의 표준 필터 타입을 가지고 있다:
- pre filters : routing하기 전에 수행된다.
- route filters : 실제 routing 행동을 수행한다.
- post filters : routing된 후에 수행한다.
- error filters : 에러가 발생하면 수행된다.

pre 필터만 만들어보자. 
@Bean이 설정된 ZuulFilter 하위 클래스는 Zuul의 필터로 등록된다.
gateway/src/main/java/hello/filters/pre 경로에 `SimpleFilter.java`를 추가해보자:
```
package hello.filters.pre;

import javax.servlet.http.HttpServletRequest;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.ZuulFilter;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SimpleFilter extends ZuulFilter {

  private static Logger log = LoggerFactory.getLogger(SimpleFilter.class);

  @Override
  public String filterType() {
    return "pre";
  }

  @Override
  public int filterOrder() {
    return 1;
  }

  @Override
  public boolean shouldFilter() {
    return true;
  }

  @Override
  public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    HttpServletRequest request = ctx.getRequest();

    log.info(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));

    return null;
  }

}
```
4개의 메서드를 구현했다:
- filterType() : 필터 타입이 무엇인지 문자열로 반환한다. "pre"로 지정했다.
- filterOrder() : 다른 필터와의 호출 순서를 지정했다. 
- shouldFilter() : 이 필터가 언제 실행되어야 하는지 지정하는 로직이다. 항상 실행되어야 하기에 true를 반환한다.
- run() : 필터의 로직이다. 이 예제에서는 로그만 남긴다.

Zuul 필터는 request를 zuul.context.RequestContext에 저장한다. 
RequestContext에서 HttpServletRequest를 가져오고 HTTP 메서드와 URL을 로깅한다.

GatewayApplication 클래스는 위에서 구현한 SimpleFilter를 @Bean 어노테이션을 사용해서 등록한다.
gateway/src/main/java/hello/GatewayApplication.java:

```
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
import org.springframework.context.annotation.Bean;
import hello.filters.pre.SimpleFilter;

@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {
  public static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
  }

  @Bean
  public SimpleFilter simpleFilter() {
    return new SimpleFilter();
  }
}
```

### Trying it out
우선 두 어플리케이션 모두 실행한 후에 브라우저에서 `localhost:8080/books/available`를 호출해보자.


이제 로그를 확인해보자. 아래와 같은 로그가 나와야 한다:
```
2016-01-19 16:51:14.672  INFO 58807 --- [nio-8080-exec-6] hello.filters.pre.SimpleFilter           : GET request to http://localhost:8080/books/available
2016-01-19 16:51:14.672  INFO 58807 --- [nio-8080-exec-6] o.s.c.n.zuul.filters.ProxyRouteLocator   : Finding route for path: /books/available
``` 
