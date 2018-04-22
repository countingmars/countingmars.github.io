---
layout: post
title:  "Spring Boot, Netflix OSS, Kafka 마이크로서비스 구축하기 - Part 1: Eureka and Config Server"
date:   2018-04-20 22:25:01
author: Mars
categories: tools
---

원본 https://dzone.com/articles/building-microservices-with-netflix-oss-apache-kaf
소스 https://github.com/isilona/microservices


 
Netflix OSS, Kafka, Spring Boot 를 사용하여 신규 사용자에게 이메일을 발송하는 시스템을 만들어보자. 
지난 몇 년 동안 MSA(마이크로서비스 아키텍처)가 인기을 얻음으로써, 이제는 반드시 알아야 할 기술이 되었다. 
나는 지난 몇 달간의 리서치와 프로젝트 경험을 통해 얻은 지식을 여기에 정리하겠다. 




### The Solution

목표는 신규 사용자를 등록하고 확인 메일을 보내는 등록 시스템을 만드는 것이다. 아래의 컴포넌트들은 이 시스템의 구성원이다:
1. Service registry (Eureka) : 모든 서비스들은 스스로를 이곳에 등록할 것이다. 
2. Config server (Spring Cloud Config) : 모든 서비스들은 이곳에서 자신들의 설정을 가져올 것이다. 
3. Gateway (Zuul) : 모든 요청을 적절한 마이크로서비스로 리다이렉트 할 것이다.
4. User service : 새로운 유저를 등록하고, "USER_REGISTERED" 메시지를 메시지 브로커(Kafka) 에게 보낸다.   
5. Email service : "USER_REGISTERED" 메시지를 받으면, 확인 메일을 신규 사용자에게 발송한다.


### Service Registry (Eureka)
많은 서비스들이 다른 서비스들과 통신해야 하는 상황에서는, 서로의 네트웍 주소를 알아야만 한다. 
일감에는, 이러한 주소 정보를 설정 파일에 저장(keep)함으로써 해결할 수 있을 것 같다. 아마도 2 ~ 4 개의 서비스라면 할만할거다.


하지만 30개 이상, 심지어 100개 이상의 서비스라면 configuration hell 을 보게될텐데, Cloud-based 시스템이라면 사태는 더 심각해질거다.  


이럴 때 Service Registry 가 도움이 될 수 있다. 이것의 목적은 서비스 인스턴스의 위치를 저장하는 것이다.
Netflix Eureka 는 이러한 레지스트리의 좋은 전형이며, 등록된 서비스에 매 30초 마다 핑을 보내서 제대로 동작하는지 검증해준다.   


이러한 레지스트리를 Spring Boot, Netflix OSS 스택으로 구현하는 것은 쉬운 일이다.


##### /pom.xml


SPRING INITIALIZER 로 새로운 프로젝트(ms-discovery)를 만든 후에 "Eureka Server" 의존성을 추가하자.
pom.xml 에 자동으로 spring-cloud-starter-eureka-server 의존성이 추가되어야만 한다.



```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```
 

##### /MsDiscoveryApplication.java


그런 후에 그냥 @EnableEurekaServer 어노테이션을 Application.java 파일에 추가하자.
이 어노테이션은 Eureka Server 관련 설정을 활성화(activate)할 것이다.



```
@EnableEurekaServer
@SpringBootApplication
public class MsDiscoveryApplication {
 public static void main(String[] args) {
	SpringApplication.run(MsDiscoveryApplication.class, args);
 }
}
```



##### /bootstrap.yml
마지막으로, 설정 파일을 조정(tune)하자. bootstrap.yml 파일에 앱의 이름과 포트를 지정하자.


```
server:
  port: ${port:8761}
spring:
  application:
    name: ms-discovery
```



##### /application.yml
application.yml 에는 표준 앱 설정을 추가하자. 
register-with-eureka 와 fetch-registry 설정은 레지스트리가 리플리카 노드가 없다는 것에 불평하는 걸 막아준다. 
로컬 테스트 목적이라면 괜찮지만 프로덕션 환경에서는 바꾸는게 좋다.


```
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```


##### 빌드, 실행 및 확인
1. `mvn clean install`
2. `java –jar target/ms–discovery–0.0.1–SNAPSHOT.jar`
3. http://localhost:8761 에서 확인해보자.


### Config Server (Spring Cloud Config)
앱이 DEV, QA, PROD 환경에서 실행되어야 하는데 배포 전에 설정 파일을 수정해야 한다면 환영받지는 못할 것이다.
설정 정보를 외부로부터 읽어오는 Externalized configuration pattern 이라고 불리는 방법이 있다.

Spring Cloud config server는 피보탈(Pivotal)이 제공하는 Externalized Configuration Pattern 이다. 
Spring Cloud config server는 서비스를 위한 각각의 환경 설정 파일을 git repository에 저장하고, 서비스가 시작될 때 마다 설정 서버에게 적절한 설정 정보를 요청한다.



그런데 우리는 cloud config server 를 discovery service 에 등록할 예정이고, 
각각의 마이크로서비스는 config server 의 위치를 discovery service 로부터 가져올 것이다.

SPRING INITIALIZR 를 이용해서 ms-config-server 프로젝트를 생성하자. 
그리고 "Config Server" 와 "Eureka Discovery" 의존성을 추가하자.
  

ms-config-server 자기 자신을 제외한 나머지 서비스들을 위한 설정파일 repository가 필요하며, git 에 ms-config-properties repository를 추가하자. 
마이크로서비스와 그에 해당하는 환경에 따른 트리 구조의 설정 파일들로 구성될 것이다:
```
ms-config-properties
	ms-user
		default
			ms-user.yml
		dev
			ms-user.yml
		prod
			ms-user.yml
```

##### /pom.xml 
spring-cloud-starter-eureka 와 spring-cloud-config-server 의존성이 필요하다.
    
```
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
	</dependency>
```


Application.java에 @EnableEurekaClient를 추가해서 Eureka의 클라이언트로 설정하자.
디폴트로, Eureka 서버를 http://localhost:8761 에서 찾을 것이다. 그리고 @EnableConfigServer를 추가해서 config server를 활성화시켜야 한다. 

```
@EnableEurekaClient
@EnableConfigServer
@SpringBootApplication
public class MsConfigServerApplication {
 public static void main(String[] args) {
  SpringApplication.run(MsConfigServerApplication.class, args);
 }

```

##### bootstrap.yml
```
server:
  port: ${port:8888}
spring:
  application:
    name: ms-config-server
```

디폴트로, config server는 8888 포트로 구동된다.



##### /application.yml
application.yml 파일에 깃 저장소 위치와 설정 파일의 경로를 검색하는 패턴을 지정해줘야 한다:

```
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/isilona/microservices/
          search-paths:
            - "ms-config-properties/{application}/{profile}"
```

##### 빌드, 실행 및 확인
1. `mvn clean install`
2. `java –jar target/ms–config-server–0.0.1–SNAPSHOT.jar`
4. Config server 가 ms-config-properties 폴더에 존재하는 설정을 잘 반환하는지 확인해보자: http://localhost:8888/ms-service/dev 
* ms-service 폴더가 ms-config-properties 에 없는데 ... 확인해야겠군! 
5. Eureka 에 Config 서버가 등록되었는지 확인해보자.

