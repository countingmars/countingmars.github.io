---
layout: post
title:  "Spring Cloud Sleuth And Zipkin"
date:   2018-04-24 13:25:01
author: Mars
categories: tools
---

원본 https://dzone.com/articles/tracing-in-microservices-with-spring-cloud-sleuth



### Sleuth
* 이전 글을 통해 Sleuth에 대해서는 어느정도 알게 되었을테니, 추가적으로 알아야 할 사항만 설명해보겠다.
로그에 trace 정보를 추가하는거 외에도, Sleuth는 다른 마이크로서비스를 호출할 때에도 혜택을 준다.

하나의 마이크로서비스에서 로그를 식별하는게 문제라기 보다는 여러 마이크로서비스의 호출 체인을 추적하는 것이 어려운 것이다.
그리고 마이크로서비스는 REST API 혹은 message hub로 상호작용하는데, Sleuth는 이러한 경우에도 추적 정보를 제공해준다. 
이 예제에서는 REST API 예제만 살펴보겠다. 

일단 이번 예제는 자기자신을 호출하는 RestTemplate 예제이다.

```
@Autowired private RestTemplate restTemplate; 
public static void main(String[] args) { 
  SpringApplication.run(SleuthSampleApplication.class, args); 
} 
@Bean public RestTemplate getRestTemplate() { 
  return new RestTemplate(); 
} 
@RequestMapping("/") public String home() { 
  LOG.log(Level.INFO, "you called home"); 
  return "Hello World"; 
} 
@RequestMapping("/callhome") public String callHome() { 
  LOG.log(Level.INFO, "calling home"); 
  return restTemplate.getForObject("http://localhost:8080", String.class); 
}
```

위 코드를 살펴보면 왜 굳이 RestTemplate을 bean으로 만들었는지 궁금할거다.
Spring Cloud Sleuth가 요청 헤더에 추적 정보를 추가하기 위해서는 RestTemplate을 반드시 스프링 빈으로 만들어야 한다.
그래야만 Spring Cloud Sleuth가 의존성 주입을 통해 RestTemplate 객체의 헤더에 손을 댈 수 있다.

`/callhome`을 호출하면 아래와 같은 로그를 확인할 수 있을거다.
```
2016-06-17 16:12:36.902 INFO [slueth-sample,432943172b958030,432943172b958030,false] 12157 
	--- [nio-8080-exec-2] com.example.SleuthSampleApplication : calling home 
2016-06-17 16:12:36.940 INFO [slueth-sample,432943172b958030,b4d88156bc6a49ec,false] 12157 
	--- [nio-8080-exec-3] com.example.SleuthSampleApplication : you called home
```
trace id가 동일하다는 것을 확인할 수 있다! 


만약 위의 주소를 브라우저 디버깅 도구로 호출해서 살펴보면 응답 헤더에서 아래의 값을 확인할 수도 있을 것이다.
`X-B3-SpanId: fbf39ca6e571f294 X-B3-TraceId: fbf39ca6e571f294`


RestTemplate의 대안으로 Spring Cloud Netflix Feign을 사용해도 추적 정보가 요청에 포함된다. 
Spring Cloud Netflix Zuul을 사용하면 다른 서비스로 포워딩할 때 추적정보를 함께 넘겨준다.

### Zipkin
만약 하나의 요청이 완료될 때까지 얼마의 시간이 걸렸는지 확인하고자 한다면 어떨까?
spring-cloud-sleuth-zipkin 의존성을 추가하면 Sleuth가 추적 정보를 Zipkin 서버에게 보내준다.
디폴트로 Sleuth는 Zipkin 서버를 `http://localhost:9411`로 가정한다.
어플리케이션 설정파일의 `spring.zipkin.baseUrl`에서 지정할 수 있다.

위 예제에서 봤듯이 우리는 Zipkin을 사용해서 추적 정보를 쉽게 수집할 수 있다.


새로운 프로젝트를 만들어서 Zipkin 서버를 구축해보자.
우선 위의 서비스에서 Zipkin 서버로 추적 정보를 전달하도록 수정해보자. 
우선 의존성을 추가하자: 
```
<dependency> 
  <groupId>org.springframework.cloud</groupId> 
  <artifactId>spring-cloud-sleuth-zipkin</artifactId> 
</dependency>
```
다음으로 얼마나 자주 Zipkin 서버에 로그를 전달할지 설정하자.

일단 모든 요청을 전달하도록 설정하자.
AlwaysSampler 빈을 추가하면 해결된다.
``` 
@Bean 
public AlwaysSampler defaultSampler() { 
  return new AlwaysSampler(); 
}
```
샘플러 빈을 추가한 후에 서버를 재시작하고 `http://localhost:8080/callhome`을 호출하고 로그를 확인해보면 로그의 export flag 값이 false에서 true로 변경된 것을 확인할 수 있다.
 
```
2016-06-20 09:03:44.939 INFO [slueth-sample,380c24fd1e5f89df,380c24fd1e5f89df,true] 19263 
	--- [nio-8080-exec-1] com.example.SleuthSampleApplication : calling home 
2016-06-20 09:03:44.966 INFO [slueth-sample,380c24fd1e5f89df,fc50a65582b7b845,true] 19263 
	--- [nio-8080-exec-2] com.example.SleuthSampleApplication : you called home
```

즉 추적 정보가 Zipkin 서버로 전달되었음을 의미한다.
`http://localhost:9411` 페이지를 브라우저로 열어보면 Zipkin UI를 확인할 수 있을 것이다. 
