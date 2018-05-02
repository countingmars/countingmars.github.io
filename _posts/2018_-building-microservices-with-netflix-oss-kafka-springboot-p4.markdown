---
layout: post
title:  "Spring Boot, Netflix OSS, Kafka 마이크로서비스 구축하기 - Part 4: Security"
date:   2018-04-21 11:25:01
author: Mars
categories: tools
---

원본 <https://dreamix.eu/blog/java/building-microservices-with-netflix-oss-apache-kafka-and-spring-boot-part-4-security>  


마이크로서비스의 그룹을 만들었으므로, 다음 단계는 이 시스템에 보안을 적용하는 거다.
Dreamix에서의 내 경험에 의하면, Spring security와 JWT 토큰은 성공적인 레시피였다. 
 
나는 이 부분을 내 GIT 저장소의 microservices/auth로 분리했으며, 프로젝트의 microservices_insomnia.json 파일에서 insomnia(HTTP 기반 API 테스트 툴)에서 가져온 테스트 요청을 볼 수 있다:


Security를 적용하기 위해 파트3에서 변경한 내용들이다:
1. Auth 서버 추가
	– Signed JWT 발행 
	– 신규 사용자가 등록되었을 때, Auth 서버에 저장
2. Gateway 서비스 개선:
	– JWT 토큰을 저장할 쿠키 관리
	– secret과 몇몇 데이터를 request에 추가
3. User 서비스 개선:
	– 토큰 검증
	– 어드민 권한(ROLE_ADMIN)을 가진 사용자에게 “get all users” 기능에 접근 설정
	– 사용자 권한(ROLE_USER)을 가진 사용자에게 “find by id” 기능에 접근 설정


### Auth-server
우선 certificate 을 생성하자. 
이것은 Auth 서버가 토큰에 사인을 할 때와 리소스 서버(ms-user)에서 검증할 때 사용된다.

##### Private key
```
keytool -genkeypair -alias ms-auth -keyalg RSA -keypass ms-auth-pass -keystore ms-auth.jks
 -storepass ms-auth-pass
```

##### Public key
```
keytool -list -rfc --keystore ms-auth.jks | openssl x509 -inform pem -pubkey
```

##### /pom.xml
이제 authorization 서버를 만들자. oauth2 의존성이 필요하며 그 외에는 이전에 사용했던 의존성들이 필요하다: eureka discovery, config client, jpa, web, H2 and Kafka.
  
```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
```


##### /AuthorizationConfig.java
@EnableAuthorizationServer 어노테이션과 AuthorizationServerConfigurer를 상속받아 OAuth 2.0 Authorization Server mechanism을 설정할 수 있다.

Spring은 AuthorizationServerConfigurerAdapter 구현체를 제공해주고 우리는 configure() 메서드를 오버라이드하면 된다. 
디폴트로 Spring Security는 access_token과 refresh_token을 UUID 포멧으로 제공해주며 이것은 ms-user서버에서 Auth 서버의 API로 검증되어야 한다. 

이것은 만약 엄청난 요청이 몰린다면, Auth 서버가 병목이 될 수 있음을 의미한다. 
하지만 signed JWT를 발생하면 토큰만으로 검증을 끝낼 수 있기에 이에 대한 해결책이 될 수 있다. 

이러한 목적을 달성하기 위해 JwtTokenStore, JwtAccessTokenConverter, DefaultTokenServices 가 필요하다.  


```
@Configuration
@EnableAuthorizationServer
public class AuthorizationConfig extends AuthorizationServerConfigurerAdapter {
    @Autowired
    @Qualifier("dataSource")
    private DataSource dataSource;

    @Autowired
    private AuthenticationManager authenticationManager;

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                .authenticationManager(this.authenticationManager)
                .tokenServices(tokenServices())
                .tokenStore(tokenStore())
                .accessTokenConverter(accessTokenConverter());
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.jdbc(dataSource);
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        KeyStoreKeyFactory keyStoreKeyFactory =
                new KeyStoreKeyFactory(
                        new ClassPathResource("ms-auth.jks"),
                        "ms-auth-pass".toCharArray());
        converter.setKeyPair(keyStoreKeyFactory.getKeyPair("ms-auth"));
        return converter;
    }

    @Bean
    @Primary
    public DefaultTokenServices tokenServices() {
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setTokenStore(tokenStore());
        defaultTokenServices.setSupportRefreshToken(true);
        defaultTokenServices.setTokenEnhancer(accessTokenConverter());
        return defaultTokenServices;
    }
}
```

##### /Account.java
Auth 서버가 인증(authentication, 누구인지 식별하는 단계) 시에 데이터베이스를 사용하도록 할려면 UserDetails, UserDetailsService, DaoAuthenticationProvider가 필요하다.


```
@Entity
public class Account implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;
    @Column(unique = true)
    private String username;
    @JsonIgnore
    private String password;
    @Enumerated(EnumType.STRING)
    @ElementCollection(fetch = FetchType.EAGER)
    private List<Role> roles;
    private boolean accountNonExpired, accountNonLocked, credentialsNonExpired, enabled;
}
```


```
@Service
public class AccountService implements UserDetailsService {
	@Autowired
	private AccountRepository accountRepository;

	@Override
	public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
			Optional<Account> account = accountRepository.findByUsername(s);
			if (account.isPresent()) {
					return account.get();
			} else {
					throw new UsernameNotFoundException(String.format("Username: %s not found", s));
			}
	}
}
```

##### /SecurityConfig.java

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setPasswordEncoder(passwordEncoder);
        provider.setUserDetailsService(userDetailsService());
        return provider;
    }

    @Bean
    public UserDetailsService userDetailsService() {
        return new AccountService();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .userDetailsService(userDetailsService())
                .passwordEncoder(passwordEncoder);
    }
}
```

Spring은 security 데이터를 데이터베이스에 저장하기 위해 6개의 테이블을 필요로한다.
Spring Boot에서는 쿼리를 schema.sql 파일에 저장하면 된다. 
There will also add an insert of the client_id that will use for our requests.



Loves and enjoys designing, architecting and implementing softwares to achieve sustaining productivity with timeboxing.
Communicates with co-workers and organization to motivate in comprehensive and direct manner.
Learns everything fast by researching, discussing, and experiencing issues with solid background in IT field.
