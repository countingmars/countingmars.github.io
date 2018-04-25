---
layout: post
title:  "Spring Cloud Sleuth For Logging"
date:   2018-04-24 11:25:01
author: Mars
categories: tools
---

원본 http://www.baeldung.com/spring-cloud-sleuth-single-application


### 1. Overview
Spring Cloud Sleuth는 복잡한 서비스의 로그를 향상시키는 도구이다.  
이 예제에서는 마이크로서비스가 아닌 monolith 서비스에서 어떻게 Sleuth를 사용할지 보여주고자 한다.

 
우리는 복잡한 상황(스케쥴된 작업, 멀티 쓰레드, 혹은 복잡한 웹 요청)에서
문제를 분석하는 기쁘지 않은 경험을 가지고 있다.
 
이런 경우에 종종 어떤 로그가 하나의 요청과 관련된 것인지 식별해내기 어렵다.
따라서 분석작업은 더 어려우지거나 심지어 불가능해진다. 


결국 해결책은 요청에 고유한 ID를 전달해서 로그의 고유성을 식별해야 하는데, 
Sleuth는 로그가 어떠한 Job, 쓰레드 혹은 요청에 속한 것인지 식별하는 것을 가능케 해준다.
또한 특별한 수고없이 기존의 로깅 프레임웍과 통합된다.  



### 2. Setup
Spring Boot 프로젝트를 하나 생성하자. 의존성은 아래와 같다:
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

Sleuth의 마지막 버전이 뭔지 알고 싶다면 여기를 참고하자.
* https://repo.spring.io/libs-milestone-local/org/springframework/cloud/spring-cloud-sleuth/

전체 프로젝트 소스코드는 아래에서 볼 수 있다.
* https://github.com/eugenp/tutorials/tree/master/spring-sleuth


추가적으로, 어플리케이션 이름을 추가해서 Sleuth가 로그를 식별할 수 있도록하자.

application.properties:
```
spring.application.name=Baeldung Sleuth Tutorial
```

### 3. Sleuth Configurations
Spring Cloud Sleuth는 Brave를 사용해서 인입된 요청에 고유한 ID를 추가한다.
게다가 Spring 팀은 이 추적용 ID를 쓰레드 경계에도 공유하는 기능을 추가했다.


추적(Traces)은 하나의 요청이나 어플리케이션 내부에서 시작된 Job으로 간주될 수 있다.
이 요청에서의 다양한 다음 스텝들은 동일한 traceId를 가지게 될 것이다.

확장(Spans)은 하나의 Job이나 요청의 섹션으로 간주될 수 있다.
하나의 Trace는 여러개의 Span으로 구성될 수 있으며, 
trace id와 span ids를 사용한다면, 우리의 서비스가 요청을 처리하기 위해 언제 어디에 있었는지 정확하게 알아낼 수 있으며
로그를 훨씬 더 쉽게 읽을 수 있게 해준다.  
 

이제 예제 좀 보자.  


### 3.1. Simple Web Request
우선 컨트롤러 클래스 하나 만들자:
```
@RestController
public class SleuthController {
 
    @GetMapping("/")
    public String helloSleuth() {
        logger.info("Hello Sleuth");
        return "success";
    }
}
```
이제 “http://localhost:8080”를 호출하고 로그를 보자:
```
2017-01-10 22:36:38.254  INFO 
  [Baeldung Sleuth Tutorial,4e30f7340b3fb631,4e30f7340b3fb631,false] 12516 
  --- [nio-8080-exec-1] c.b.spring.session.SleuthController : Hello Sleuth
```
평범한 로그처럼 보이겠지만, 대괄호([, ]) 사이의 정보가Spring Sleuth가 추가한 핵심 정보이다.
이 데이터는 아래의 형식을 따른다:  

`[application name, traceId, spanId, export]`

- Application name: 앱 이름으로 여러 장비의 로그를 수집할 때 사용한다.
- TraceId: 하나의 요청에 대한 ID로써, 하나의 Request, Job 혹은 Action에 할당된다.
- SpanId: 작업 단위를 추적한다. 여러 단계로 처리될 request의 경우이다.
- Export: 이 속성은 boolean 값인데 Zipkin과 같은 Aggregator에게 export될지를 가리킨다. Zipkin은 이 글의 범위를 벗어나지만 굉장히 중요한 역할을 담당한다.

이제 이 라이브러리의 힘을 이해해야 할 때인거 같다. 
다른 예제를 살펴보고 이 라이브러리가 어떻게 로깅에 통합되어야 할지 알아보자.

### 3.2. Simple Web Request with Service Access
서비스 하나 추가하자:
```
@Service
public class SleuthService {
	public void doSomeWorkSameSpan() {
		Thread.sleep(1000L);
		logger.info("Doing some work");
	}
}
```
이제 컨트롤러에 서비스를 적용하자.
@Autowired
private SleuthService sleuthService;
```     
	@Autowired
	private SleuthService sleuthService;
     
	@GetMapping("/same-span")
	public String helloSleuthSameSpan() throws InterruptedException {
		logger.info("Same Span");
		sleuthService.doSomeWorkSameSpan();
		return "success";
	}
```
마지막으로 앱을 재시작하고 “http://localhost:8080/same-span” 호출한 후  
로그를 보자:
```
2017-01-10 22:51:47.664  INFO 
  [Baeldung Sleuth Tutorial,b77a5ea79036d5b9,b77a5ea79036d5b9,false] 12516 
  --- [nio-8080-exec-3] c.b.spring.session.SleuthController      : Same Span
2017-01-10 22:51:48.664  INFO 
  [Baeldung Sleuth Tutorial,b77a5ea79036d5b9,b77a5ea79036d5b9,false] 12516 
  --- [nio-8080-exec-3] c.baeldung.spring.session.SleuthService  : Doing some work
```
주목해야 할 점은 두 줄의 로그가 동일한 trace id와 span id를 사용한다는 점이다.
심지어 로그 메시지가 다른 클래스로부터 생성되었음에도.

이것은 trace id로 로그를 식별하는 일이 쉬운 일로 만들어준다.

이것은 디폴트 동작이고, 하나의 요청은 하나의 trace id와 span id를 사용하게 되지만 
원한다면 span을 새로 추가할 수도 있다.

### 3.3. Manually Adding a Span
새로운 컨트롤러를 추가하자.
```
@GetMapping("/new-span")
public String helloSleuthNewSpan() {
	logger.info("New Span");
	sleuthService.doSomeWorkNewSpan();
	return "success";
}
```
이제 서비스에 메서드를 추가해보자.
```
@Autowired
private Tracer tracer;
// ...
public void doSomeWorkNewSpan() throws InterruptedException {
    logger.info("I'm in the original span");
 
    Span newSpan = tracer.newTrace().name("newSpan").start();
    try (SpanInScope ws = tracer.withSpanInScope(newSpan.start())) {
        Thread.sleep(1000L);
        logger.info("I'm in the new span doing some cool work that needs its own span");
    } finally {
        newSpan.finish();
    }
 
    logger.info("I'm in the original span");
}
```

주목해야 할 점은 우리가 새로운 객체인 Tracer를 추가했다는 점이다. 
Spring Sleuth가 startup 시점에 생성된 tracer 인스턴스이고 의존성 주입으로 쉽게 사용할 수 있다.

Trace는 반드시 시작과 중지를 지정해줘야 한다.
따라서 try-finally를 사용하는 것은 필수이며 

그리고 새로운 span이 scope 안에 위치해야 한다는 점도 주목하자.

이제 앱을 재시작하고 “http://localhost:8080/new-span”을 조회한 후에 로그를 보자.
```
2017-01-11 21:07:54.924 
  INFO [Baeldung Sleuth Tutorial,9cdebbffe8bbbade,9cdebbffe8bbbade,false] 12516
  --- [nio-8080-exec-6] c.b.spring.session.SleuthController      : New Span
2017-01-11 21:07:54.924 
  INFO [Baeldung Sleuth Tutorial,9cdebbffe8bbbade,9cdebbffe8bbbade,false] 12516
  --- [nio-8080-exec-6] c.baeldung.spring.session.SleuthService  : 
  I'm in the original span
2017-01-11 21:07:55.924 
  INFO [Baeldung Sleuth Tutorial,9cdebbffe8bbbade,1e706f252a0ee9c2,false] 12516
  --- [nio-8080-exec-6] c.baeldung.spring.session.SleuthService  : 
  I'm in the new span doing some cool work that needs its own span
2017-01-11 21:07:55.924 
  INFO [Baeldung Sleuth Tutorial,9cdebbffe8bbbade,9cdebbffe8bbbade,false] 12516
  --- [nio-8080-exec-6] c.baeldung.spring.session.SleuthService  : 
  I'm in the original span
```
세번째 로그의 span id만 다르다는 것을 확인할 수 있다.
이것을 통해 하나의 요청에 대한 세부적인 추적을 할 수 있다.

이제 Sleuth의 쓰레드 서포트를 살펴보자.

### 3.4. Spanning Runnables
Sleuth의 쓰레드 수용성을 확인하기 위해 thread pool의 setup 클래스를 추가하자.
```
@Configuration
public class ThreadConfig {
    @Autowired
    private BeanFactory beanFactory;
    @Bean
    public Executor executor() {
        ThreadPoolTaskExecutor threadPoolTaskExecutor
         = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.setCorePoolSize(1);
        threadPoolTaskExecutor.setMaxPoolSize(1);
        threadPoolTaskExecutor.initialize();
 
        return new LazyTraceExecutor(beanFactory, threadPoolTaskExecutor);
    }
}
```
LazyTraceExecutor는 Sleuth에서 제공하는 것으로 trace id를 쓰레드에 전달해주고 쓰레드별로 새로운 span id를 생성해준다.
이제 executor를 컨트롤러에서 사용해보자.
```
	@Autowired
	private Executor executor;
    
	@GetMapping("/new-thread")
	public String helloSleuthNewThread() {
		logger.info("New Thread");
		Runnable runnable = () -> {
				try {
						Thread.sleep(1000L);
				} catch (InterruptedException e) {
						e.printStackTrace();
				}
				logger.info("I'm inside the new thread - with a new span");
		};
		executor.execute(runnable);

		logger.info("I'm done - with the original span");
		return "success";
	}
```
앱을 재시작하고 “http://localhost:8080/new-thread” 페이지를 조회한 후 로그를 보자.
```
2017-01-11 21:18:15.949  
  INFO [Baeldung Sleuth Tutorial,96076a78343c364d,96076a78343c364d,false] 12516 
  --- [nio-8080-exec-9] c.b.spring.session.SleuthController      : New Thread
2017-01-11 21:18:15.950  
  INFO [Baeldung Sleuth Tutorial,96076a78343c364d,96076a78343c364d,false] 12516 
  --- [nio-8080-exec-9] c.b.spring.session.SleuthController      : 
  I'm done - with the original span
2017-01-11 21:18:16.953  
  INFO [Baeldung Sleuth Tutorial,96076a78343c364d,e3b6a68013ddfeea,false] 12516 
  --- [lTaskExecutor-1] c.b.spring.session.SleuthController      : 
  I'm inside the new thread - with a new span
```
모든 로그가 trace id를 공유하고 있고, runnable에서 남긴 로그만 다른 span을 가지고 있음을 확인할 수 있다.

이 모든 것은 LazyTraceExecutor에 의해서 발생한 것임을 기억하자. 

이제 Sleuth의 @Async methods 지원을 알아보자.


### 3.5. @Async Support
Async 지원을 위해 ThreadConfig 클래스를 좀 수정하자:
```
@Configuration
@EnableAsync
public class ThreadConfig extends AsyncConfigurerSupport {
	//...
	@Override
	public Executor getAsyncExecutor() {
		ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
		threadPoolTaskExecutor.setCorePoolSize(1);
		threadPoolTaskExecutor.setMaxPoolSize(1);
		threadPoolTaskExecutor.initialize();

		return new LazyTraceExecutor(beanFactory, threadPoolTaskExecutor);
	}
}
```
AsyncConfigurerSupport를 상속받았고 LazyTraceExecutor를 사용했으며, @EnableAsync를 추가했다.

이제 async 메서드를 추가해보자:
```
	@Async
	public void asyncMethod() {
		logger.info("Start Async Method");
		Thread.sleep(1000L);
		logger.info("End Async Method");
	}
```

이제 컨트럴러에서 사용해보자:
```
@GetMapping("/async")
public String helloSleuthAsync() {
	logger.info("Before Async Method Call");
	sleuthService.asyncMethod();
	logger.info("After Async Method Call");
	 
	return "success";
}
```
앱을 재시작 후에 “http://localhost:8080/async” 페이지를 조회하고 로그를 보자.
```
2017-01-11 21:30:40.621  
  INFO [Baeldung Sleuth Tutorial,c187f81915377fff,c187f81915377fff,false] 10072 
  --- [nio-8080-exec-2] c.b.spring.session.SleuthController      : 
  Before Async Method Call
2017-01-11 21:30:40.622  
  INFO [Baeldung Sleuth Tutorial,c187f81915377fff,c187f81915377fff,false] 10072 
  --- [nio-8080-exec-2] c.b.spring.session.SleuthController      : 
  After Async Method Call
2017-01-11 21:30:40.622  
  INFO [Baeldung Sleuth Tutorial,c187f81915377fff,8a9f3f097dca6a9e,false] 10072 
  --- [lTaskExecutor-1] c.baeldung.spring.session.SleuthService  : 
  Start Async Method
2017-01-11 21:30:41.622  
  INFO [Baeldung Sleuth Tutorial,c187f81915377fff,8a9f3f097dca6a9e,false] 10072 
  --- [lTaskExecutor-1] c.baeldung.spring.session.SleuthService  : 
  End Async Method
```
Async 메서드에서 남긴 로그에서만 span id가 변경되었음을 확인할 수 있다.

이제 scheduled tasks를 살펴보자.

### 3.6. @Scheduled Support
마지막으로 알아볼 것은 Sleuth가 @Scheduled 메서드를 지원하는가이다. 
일단 ThreadConfig 클래스를 좀 고치자:
```
@Configuration
@EnableAsync
@EnableScheduling
public class ThreadConfig extends AsyncConfigurerSupport
  implements SchedulingConfigurer {
	//...
	@Override
	public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
			scheduledTaskRegistrar.setScheduler(schedulingExecutor());
	}
	@Bean(destroyMethod = "shutdown")
	public Executor schedulingExecutor() {
			return Executors.newScheduledThreadPool(1);
	}
}
```
SchedulingConfigurer 인터페이스를 상속받았고 configureTasks 메서드를 오버라이드했으며 @EnableScheduling을 추가했다. 

다음은, 스케쥴 작업을 서비스에 추가하는 것이다:
```
@Service
public class SchedulingService {
	private Logger logger = LoggerFactory.getLogger(this.getClass());

	@Autowired
	private SleuthService sleuthService;

	@Scheduled(fixedDelay = 30000)
	public void scheduledWork() throws InterruptedException {
		logger.info("Start some work from the scheduled task");
		sleuthService.asyncMethod();
		logger.info("End work from scheduled task");
	}
}
```
30시 초마다 실행되는 스케쥴 작업을 추가했다.

이제 재시작하고 작업이 실행될 때까지 기다리고 로그를 보자.
```
2017-01-11 21:30:58.866 
  INFO [Baeldung Sleuth Tutorial,3605f5deaea28df2,3605f5deaea28df2,false] 10072
  --- [pool-1-thread-1] c.b.spring.session.SchedulingService     : 
  Start some work from the scheduled task
2017-01-11 21:30:58.866 
  INFO [Baeldung Sleuth Tutorial,3605f5deaea28df2,3605f5deaea28df2,false] 10072
  --- [pool-1-thread-1] c.b.spring.session.SchedulingService     : 
  End work from scheduled task
```

Sleuth가 작업을 위해 trace id와 span id를 새로 만들었음을 확인할 수 있다.
매 스케쥴 작업은 고유한 trace id와 span id를 가지게 될거다.


### 4. Conclusion
이제 Sleuth 사용법은 충분히 알았을테니 RestTemplate, messaging protocol, Zuul 등과 연동하는 방법이 궁금할거다.

