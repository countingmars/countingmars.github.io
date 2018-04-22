---
layout: post
title:  "Spring Boot, Netflix OSS, Kafka 마이크로서비스 구축하기 - Part 3:  Email Service and Gateway"
date:   2018-04-21 11:25:01
author: Mars
categories: tools
---

원본 https://dreamix.eu/blog/java/building-microservices-with-netflix-oss-apache-kafka-and-spring-boot-part-3-email-service  


### Email Service
이제는 email service 차례인데, 이 서비스는 USER_CREATED_TOPIC을 리스닝할거다. 
Kafka consumer를 설정하고 수신된 메시지를 UserDto로 변환할거다.


User 마이크로서비스와 유사하게 EmailService에 비즈니스 로직을 구현할 것이다. 
EmailService는 payload의 UserDto 사용하여 Mail 엔터티를 만들어서 데이터베이스에 저장하고 메일을 보낼 것이다.

새 프로젝트로 ms-mail 을 생성하고 Eureka Discovery; JPA; H2; Kafka; Config Client; 의존성을 추가하자.

##### pom.xml
``` 
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.kafka</groupId>
	<artifactId>spring-kafka</artifactId>
</dependency>
<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<scope>runtime</scope>
</dependency>
<dependency>
	<groupId>javax.mail</groupId>
	<artifactId>mail</artifactId>
</dependency>
```

##### /ms-mail.yml
메일 발송 설정은 gmail을 사용할거다. 
발신인으로 사용될 gmail 계정을 username과 password에 꼭 입력하자. 


```
mail:
    host: smtp.gmail.com
    port: 587
    username: username
    password: password
    properties.mail.smtp:
      auth: true
      starttls.enable: true
```


##### /ReceiverConfig.java
Kafka로부터 메시지를 읽기 위해서는 ConsumerFactory와 KafkaListenerContainerFactory가 필요하다. 
이 또한 ProducerFactory와 마찬가지로 설정을 해줘야한다.  

@EnableKafka는 @KafkaListener를 사용하기 위해서 필요하다.
```
@Configuration
@EnableKafka
public class ReceiverConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;
 
    @Bean
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "UserCreatedConsumer");
 
        return props;
    }
 
    @Bean
    public ConsumerFactory<String, UserDto> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs(), new StringDeserializer(),
                new JsonDeserializer<>(UserDto.class));
    }
 
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, UserDto> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, UserDto> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
 
        return factory;
    }
 
    @Bean
    public Receiver receiver() {
        return new Receiver();
    }
}
```


##### /Receiver.java
이제 @KafkaListener을 가진 메서드는 메시지가 수신되었을 때 호출될 것이다.
```
private CountDownLatch latch = new CountDownLatch(1);

@KafkaListener(topics = "${spring.kafka.topic.userCreated}")
public void receive(UserDto payload) {
   emailService.sendSimpleMessage(payload);
   latch.countDown();
}
```

##### /EmailServiceImpl.java
위에서 언급했듯이, EmailService는 인입된 payload를 변환하고 이메일을 발송하고 기록을 남기는 곳이다.

```
public class EmailServiceImpl implements EmailService {
    @Override
    public void sendSimpleMessage(UserDto input) {
        try {
            Mail newMail = new Mail();
            newMail.setTo(input.getUsername());
            newMail.setSubject("TestSubject");
            newMail.setText("TestText");

            SimpleMailMessage message = new SimpleMailMessage();
            message.setTo(newMail.getTo());
            message.setSubject(newMail.getSubject());
            message.setText(newMail.getText());

            mailRepository.save(newMail);
            emailSender.send(message);
        } catch (MailException exception) {
            exception.printStackTrace();
        }
    }
}
```


##### 빌드, 실행 그리고 테스트
1. Service Registry (Eureka)가 실행 중인지 확인하자 (http://localhost:8761)
2. Config 서버(Spring Cloud Config)가 실행 중이고 ms-user와 ms-email 설정이 존재하는지 확인해보자 (http://localhost:8888/ms-user/default)
3. 프로젝트 빌드하자 `mvn clean install`
4. `java-jar target/ms-mail-0.0.1-SNAPSHOT.jar`
5. 아래의 HTTP POST 요청으로 신규 사용자를 생성해보자:
```
POST http://localhost:8081/member 
{
"username": "email@example.com",
"password": "password"
}
```
6. 사용자가 생성되었고 이메일이 도착했는지 확인해보자.


### Gateway (Zuul)
만약 사용자를 등록하고 싶다면 서버의 IP/PORT 등을 알아야만 하는데, 이런 문제를 해결하기 위해 모든 마이크로서비스를 위한 입구가 될 새로운 서비스(Zuul)을 만들어보자.
클라이언트는 Zuul을 호출하고 Zuul은 적절한 곳으로 요청을 보내줄거다.

 
ms-gateway 프로젝트를 생성하고 Eureka Discovery; Config Client; Zuul 의존성을 추가하자.

##### /prom.xml
```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```

##### /bootstrap.yml 
Config 클라이언트를 설정하자. 
```
server:
  port: 8765
spring:
  application:
    name: ms-gateway
  cloud:
    config:
      discovery:
        enabled: true
        service-id: ms-config-server
```

Application.java 파일에 @EnableEurekaClient 어노테이션을 추가하고, Zuul의 기능을 활성화시키기 위해 @EnableZuulProxy를 추가하자.


##### /ms-gateway.yml
라우팅 설정이 필요하므로 ms-config-properties repository에 설정 파일을 추가하자.

```
zuul:
  routes:
    ms-user: /api/user/**
```

이 설정을 추가되면 8765:/api/user (8765는 gateway의 포트이다)으로 요청을 보내면 ms-user 마이크로서비스로 리다이렉트될거다.

그리고 Zuul은 필요한 IP와 PORT를 Eureka로부터 가져올거다.

 
##### 빌드, 실행 및 테스트
 
1. 신규 사용자 생성
```
POST http://localhost:8765/api/user/member
{
"username": "email@example.com",
"password": "password"
}
```
2. 사용자 생성 확인하기 
```
GET http://localhost:8765/member
```
3. 이메일 발송되었는지 확인하기 

