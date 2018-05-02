---
layout: post
title:  "Spring Boot, Netflix OSS, Kafka 마이크로서비스 구축하기 - Part 2: Message Broker and User Service"
date:   2018-04-20 23:25:01
author: Mars
categories: tools
---

원본 <https://dreamix.eu/blog/java/building-microservices-with-netflix-oss-apache-kafka-and-spring-boot-part-2>  



### Message Broker (Kafka & ZooKeeper)
Kafka는 분산 시스템이고 Zookeeper를 이용해서 클러스터 노드, topic, partition의 상태를 추적한다.
하지만 이 예제에서는 Kafka의 분산 기능을 사용하지는 않을 예정이다.  

Kafka를 사용하기 전에 Zookeeper를 설치하는 것은 필수이다. 아래 명령은 우분투 16.04에 Zookeeper를 설치하는 명령이다.


##### Install Zookeeper 
```
$ sudo apt-get install zookeeperd
```
Zookeeper가 설치되면 자동으로 데몬으로 시작되고 2181 포트를 리스닝할거다.

##### Ask zookeeper if it is ok

```
$ telnet localhost 2181
```
`ruok` 를 입력하고 엔터를 누르면 이런 응답을 봐야한다: imok Connection closed by foreign host.

##### Download the latest Kafka
https://kafka.apache.org/downloads 로 가서 최신 버전의 바이너리(현재는 Kafka_2.12-0.11.0.1.tgz)를 다운로드 받자. 

```
$ mkdir ~/kafka
$ wget http://apache.cbox.biz/kafka/0.11.0.1/kafka_2.11-0.11.0.1.tgz
$ tar -xvzf kafka_2.11-0.11.0.1.tgz ~/kafka
```

##### Configure the Kafka Server
server.properties 파일을 업데이트하자. 디폴트로 deleting topics 은 disabled 된 상태인데 enable 시키는게 좋다.
파일의 마지막에 delete.topic.enable 를 추가하자.

```
$ nano ~/kafka/config/server.properties
delete.topic.enable = true
```


##### Run the Kafka Server as a background process
```
$ nohup ~/kafka/bin/kafka-server-start.sh ~/kafka/config/server.properties > ~/kafka/kafka.log 2>&1 &
```

##### Verify Kafka is running
```
$ echo dump | nc localhost 2181 | grep brokers
```



### User Service
이제 우리는 Kafka를 구동시켰으므로 User 마이크로서비스를 만들 수 있게 되었다. 
Part 1에서 이미 말했듯이 User 마이크로서비스는:
1. 자기 스스로를 Service Registry (Eureka)에 등록한다.
2. Config 서버(Spring Cloud Config)에서 설정을 가져온다. 
3. 2개의 endpoints를 가진다 
	- /member : POST 요청은 신규 사용자를 등록한다
	- /member : GET 요청은 등록된 모든 유저를 가져올 것이다.
4. 사용자를 새로 등록하게 되면, User 서비스는 “USER_REGISTERED” 메시지를 메시지 브로커(Kafka)에게 보낸다.
5. H2 메모리 데이터베이스에 사용자를 저장한다.


SPRING INITIALIZR를 이용해서 ms-user 프로젝트를 생성하자. 필요한 의존성은 이렇다: Eureka Discovery; JPA; H2; Kafka; Config Client;

##### /pom.xml
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

```

Config 서버와 마찬가지로, Discovery Client 를 활성화시키기 위해 Application.java에 @EnableEurekaClient를 추가하자.

##### /bootstrap.yml
설정파일에 프로젝트 이름과 포트를 지정하자. 

여기서 새로 추가되는 것은 cloud > config > discovery 이다. User 마이크로서비스는 'ms-config-server'라는 키 값으로 Service Registry로부터 Config 서버의 위치를 찾을 것이다. 
덕분에 url 혹은 port를 하드코딩할 필요가 없다. h2 메모리 데이터베이스에 대한 설정이나, Kafka에 대한 설정은 모두 Config 서버로 부터 읽어오게 된다. 

```
server:
  port: 8081
spring:
  application:
    name: ms-user
  cloud:
    config:
      discovery:
        enabled: true
        service-id: ms-config-server
```

 


##### /ms-user.yml
그러니 ms-config-properties/ms-user 에는 이러한 설정이 필요하다.
```
spring:
  h2:
    console:
      enabled: true
      path: /h2-console
  datasource:
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    username: sa
    password:
  kafka:
    bootstrap-servers: localhost:9092
    topic:
      userCreated: USER_CREATED_TOPIC
security:
  basic:
    enabled: false
```

##### /User. java
우선 우리는 간단한 Spring 웹 프로젝트를 생성할거다. User 엔티티와 UserRepository, UserService and UserController를 가진다. 

User 엔티티는 username과 password를 가지는데 username에 email을 저장할 예정이다.


```
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

@NotNull
private String username;

@NotNull
private String password;
```



##### /UserRepository.java
```
public interface UserRepository extends CrudRepository<User, Long> {
}
```


##### /UserService.java 
UserService는 신규 사용자 등록 메서드와 모든 사용자 조회 메서드를 가진다.

```
public interface UserService {
    User registerUser(User input);
    Iterable<User> findAll();
}
```


##### UserController
UserController 에는 GET /member 과 POST /member 두 개의 endpoints를 추가할거다. 

구현은 간단하다.
```
@RequestMapping(method = RequestMethod.GET, path = "/member")
public ResponseEntity<Iterable<User>> getAll() {
    Iterable<User> all = userService.findAll();
    return new ResponseEntity<>(all, HttpStatus.OK);
}

@RequestMapping(method = RequestMethod.POST, path = "/member")
public ResponseEntity<User> register(@RequestBody User input) {
    User result = userService.registerUser(input);
    return new ResponseEntity<>(result, HttpStatus.OK);
}
```

##### SenderConfig.java 
이제 sender 설정을 자세히 살펴보자. 
Kafka 토픽에 메시지 생산을 할려면 KafkaTemplate 이 필요하다. 
KafkaTemplate은 ProducerFactory를 필요로 하는데, Producer 생성 전략(strategy)를 설정해야 한다.

ProducerFactory는 이러한 전략을 맵으로 설정한다. 
전략을 설정하기 위한 속성 중에 가장 중요한 것은 BOOTSTRAP_SERVERS_CONFIG, KEY_SERIALIZER_CLASS_CONFIG, VALUE_SERIALIZER_CLASS_CONFIG 이다.

```
public class SenderConfig {
	@Value("${spring.kafka.bootstrap-servers}")
	private String bootstrapServers;
	
	@Bean
	public Map<String, Object> producerConfigs() {
			Map<String, Object> props = new HashMap<>();
			props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
			props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
			props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
	
			return props;
	}
	
	@Bean
	public ProducerFactory<String, User> producerFactory() {
			return new DefaultKafkaProducerFactory<>(producerConfigs());
	}
	
	@Bean
	public KafkaTemplate<String, User> kafkaTemplate() {
			return new KafkaTemplate<>(producerFactory());
	}
	
	@Bean
	public Sender sender() {
			return new Sender();
	}
}
```
메시지를 발송하기 위한 Kafka 서버 `@Value("${spring.kafka.bootstrap-servers}")`는 localhost:9092로 설정되는데 이것은 Config 서버로부터 가져온다.


##### /Sender.java

전송될 메시지(payload)는 User 객체를 JsonSerializer를 이용해서 생성한다. 
KafkaTemplate을 이용해서 메시지를 전송할 Sender를 구현해보자.

```
public class Sender {
	@Autowired
	private KafkaTemplate<String, User> kafkaTemplate;

	public void send(String topic, User payload) {
			kafkaTemplate.send(topic, payload);
	}
}
```

##### /UserServiceImpl.java
비즈니스 로직이 구현될 클래스는 UserServiceImpl 이다.

``` 
public class UserServiceImpl implements UserService {
	@Value("${spring.kafka.topic.userCreated}")
	private static String USER_CREATED_TOPIC;
	 
	private UserRepository userRepository;
	private Sender sender;
	 
	@Override
	public User registerUser(User input) {
			User createdUser = userRepository.save(input);
			sender.send(USER_CREATED_TOPIC, createdUser);
			return createdUser;
	}
}
```
