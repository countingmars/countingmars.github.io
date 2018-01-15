---
layout: post
title:  "Spring Boot Server Tests"
date:   2018-01-14 18:25:01
author: Mars
categories: java
---


### Spring Boot Server Tests
https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html

#### A Full Running Server For Tests
서버를 띄워서(a full running server) 테스트할 때는:
1. `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`: 서버를 랜덤 포트로 띄우고.
2. `TestRestTemplate`: 서버에 상대 경로로 접근할 수 있다.


```
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class RandomPortExampleTests {

	@Autowired
	private TestRestTemplate restTemplate;

	@Test
	public void exampleTest() {
		String body = this.restTemplate.getForObject("/", String.class);
		assertThat(body).isEqualTo("Hello World");
	}

}
```

예제 코드는 여기서 볼 수 있다.
- https://github.com/countingmars/spring-boot-server-tests/

