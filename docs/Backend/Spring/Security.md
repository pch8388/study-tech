# Spring Security

- Spring Security 의존성만 추가해도 기본 인증 폼을 보내준다. ( 스프링 부트 사용시 설정이 자동 설정 되는 부분이 있음 ) ⇒ 자동 설정으로 id : user, password : 콘솔확인 으로 생성한다
- 시큐리티 설정

    ```java
    // 시큐리티 설정 파일임을 나타냄
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            // 특정 요청에 대해 설정
            // 인가
            http.authorizeRequests()
                    .mvcMatchers("/", "/info").permitAll()  // 인증없이 접근 가능
                    .mvcMatchers("/admin").hasRole("ADMIN")  // role 에 따라 구분
                    .anyRequest().authenticated();   // 기타 요청은 인증을 하기만 하면 됨

            // 인증
            http.formLogin();  // form login 을 사용
            http.httpBasic();  // http basic authentication 사용
        }
    }
    ```

![기본정보생성](../../images/security1.png)

UserDetailsServiceAutoConfigration 에서 기본 유저 정보를 생성한다

- UserDetailsServiceAutoConfiguration 안에서 인메모리 방식으로 유저를 생성해주는 것을 알 수 있다

```java
@Bean
	@ConditionalOnMissingBean(
			type = "org.springframework.security.oauth2.client.registration.ClientRegistrationRepository")
	@Lazy
	public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties,
			ObjectProvider<PasswordEncoder> passwordEncoder) {
    
    // 시큐리티 프로퍼티의 유저 타입인 유저를 가져와 세팅하는 것을 확인
		SecurityProperties.User user = properties.getUser(); 
		List<String> roles = user.getRoles();
		return new InMemoryUserDetailsManager(
				User.withUsername(user.getName()).password(getOrDeducePassword(user, passwordEncoder.getIfAvailable()))
						.roles(StringUtils.toStringArray(roles)).build());
	}
```

![기본정보생성코드](../../images/security2.png)

- 따라서 설정 파일을 세팅하면 특정유저를 만들 수 있다 ⇒ 그냥 참고용으로만 ..

```java
// application.properties
spring.security.user.name=admin
spring.security.user.password=123
spring.security.user.roles=ADMIN
```

### 인메모리 유저 설정

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()       // 인메모리 authentication 설정
        .withUser("sc").password("{noop}123").roles("USER").and()
        .withUser("admin").password("{noop}!@#").roles("ADMIN");
}
```

- AuthenticationManagerBuilder 에 이런 인증 방식을 사용하겠다고 알려주는 것(auth.inMemoryAuthentication())
- {noop} : Spring 5 부터는 시큐리티 암호는 무조건 암호화를 해야한다. 그리고 암호화 한 방법에 대해 알려줘야 하는 데, 그것을 괄호안에 정의한다. noop 은 암호화를 하지 않겠다는 의미

### JPA 연동

1. 의존성 추가
2. UserDetailsService : 스프링 시큐리티에서 DAO 인터페이스를 통해서 데이터 베이스에 있는 유저 정보를 가지고 인증을 하는 서비스 인터페이스
    - loadUserByUsername 메서드 : user 정보를 가져와서 UserDetails 를 리턴해줘야 함
    - User 의 빌더를 이용하여 간단하게 UserDetails 타입의 클래스를 생성해줄 수 있다
    - UserDetailsService 타입의 빈이 등록만 되어 있으면 config file 의 configure 메서드의 AuthenticationManagerBuilder 에 설정하지 않아도 된다.

### PasswordEncoder

- 패스워드 인코딩 방식을 결정해주는 빈
- NoOpPasswordEncoder 가 5.0 이전 기본 전략 - 현재 deprecated
    - 현재는 bcrypt 가 기본전략
    - PasswordEncoderFactories.createDelegatingPasswordEncoder() 를 반환해주면 기본전략과 특정 아이디로 인코더를 지정한 방법을 사용할 수 있다
    