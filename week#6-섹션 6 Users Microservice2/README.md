# Secotion 6. Users Microservice 2

## 인증과 권한 기능 개요

![22](https://user-images.githubusercontent.com/52458791/142789342-54dea868-0277-42bf-809b-c152fcca65ee.png)
- 이번 섹션에서는 회원 로그인 기능을 개발
- 회원 정보 수정/삭제는 알아서 실습

![222](https://user-images.githubusercontent.com/52458791/142789365-ae15b598-7a51-4a34-a382-1e0dfc200104.png)
- 로그인을 위한 API 생성
- apigateway를 통해 API를 호출
- 스프링 시큐리티를 사용해 비밀번호를 암호화함
- userId는 UUID로 바꾸고 JWT를 생성해 사용자에게 반환

![1](https://user-images.githubusercontent.com/52458791/142789066-533aefae-06d6-49d7-b7f6-2af0458e3a0b.png)
- 사용자 로그인 정보 저장을 위한 클래스 생성

![11](https://user-images.githubusercontent.com/52458791/142789099-ef49dfe6-0605-4140-abe9-47b6e3f41d9e.png)
- 스프링 시큐리티를 이용해 로그인 요청 발생시 작업 처리를 위한 커스텀 필터
- UsernamePasswordAuthenticationFilter 상속을 받아 구현
- attemptAuthentication()은 전달받은 데이터를 requestlogin으로 변환함
- 전달받은 email과 pwd값을 추출하고 인증처리에 사용
- successfulAuthentication()에 인증 성공 후 권한 등 기능을 추가

![111](https://user-images.githubusercontent.com/52458791/142789110-1f539d4c-e74c-4bd5-bfc0-3c7962b6c276.png)
- 권한 관련한 메소드 기능 구현
- 모든 사용자를 허용해주었던 것을 수정하여 IP를 제한하고 모든 요청에 대한 인증처리 필터 추가

![1111](https://user-images.githubusercontent.com/52458791/142789118-e0566a48-0428-49de-bc9c-703c49d9587c.png)
- configure 메소드에서 인증 관련 작업 처리, 비밀번호 plain text를 암호화

![11111](https://user-images.githubusercontent.com/52458791/142789134-43e267aa-7cb5-46bf-985e-68e360d121a0.png)
- 기존의 userService는 보통의 인터페이스 이기에 UserDetailService를 상속받아 새로 구현
- username 즉 email를 사용해 찾아오는 메소드 구현
- find계열 메소드는 select로 db내용을 가져옴(select pwd from users where username = ?)
- email이 존재하지 않으면 exception 발생, 성공시 email, pwd 반환

![111111](https://user-images.githubusercontent.com/52458791/142789159-3c606cfd-db7e-4a12-9d3a-36c8ad36b087.png)
- 기존의 루트 정보를 3개로 구분 -> 로그인, 회원가입, 나머지
- 직접 msa UserService 호출시 굳이 서비스이름을 붙이지 않아도 되도록 수정
- 정규표현식 사용, user-service글자를 뒤에 패턴으로 치환

![2](https://user-images.githubusercontent.com/52458791/142789186-c5307748-8f1c-4112-8747-60da9e8ec85e.png)
- Postman을 사용해 로그인 서비스 테스트

## AuthenticationFilter 추가

RequestLogin.java
```java
@Data
public class RequestLogin {
    @NotNull(message = "Email cannot be null")
    @Size(min = 2, message = "Email not be less than two characters")
    @Email
    private String email;

    @NotNull(message = "Password cannot be null")
    @Size(min = 8, message = "Password must be equals or greater than 8 characters")
    private String password;
}
```
- 로그인 정보를 처리하는 모델 클래스 생성

AuthenticationFilter.java - attemptAuthentication 메소드
```java
@Slf4j
public class AuthenticationFilter extends UsernamePasswordAuthenticationFilter {
    private UserService userService;
    private Environment env;

    public AuthenticationFilter(AuthenticationManager authenticationManager,
                                UserService userService,
                                Environment env) {
        super(authenticationManager);
        this.userService = userService;
        this.env = env;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException {
        try{
            RequestLogin creds = new ObjectMapper().readValue(request.getInputStream(), RequestLogin.class);

            return getAuthenticationManager().authenticate(
                    new UsernamePasswordAuthenticationToken(
                            creds.getEmail(),
                            creds.getPassword(),
                            new ArrayList<>())
            );

        }catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {
}
```
- UsernamePasswordAuthenticationFilter 상속하고 두가지 메소드 오버라이딩
- attemptAuthentication메소드에서 inputstream으로 들어온 사용자 정보를 ObjectMapper를 통해 자바 클래스로 변경
- 사용자 정보를 스프링 시큐리티에서 사용할 수 있게 이메일과 비밀번호를 UsernamePasswordAuthenticationToken의 형태로 데이터 변경
- 최종적으로 이 데이터 값을 인증 처리 매니저가 비교

WebSecurity.java
```java
@Configuration
@EnableWebSecurity
public class WebSecurity extends WebSecurityConfigurerAdapter {
    private UserService userService;
    private BCryptPasswordEncoder bCryptPasswordEncoder;
    private Environment env;

    public WebSecurity(Environment env, UserService userService, BCryptPasswordEncoder bCryptPasswordEncoder){
        this.env = env;
        this.userService = userService;
        this.bCryptPasswordEncoder = bCryptPasswordEncoder;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
//        http.authorizeRequests().antMatchers("/users/**").permitAll();
        http.authorizeRequests().antMatchers("/**")
                .hasIpAddress("10.110.120.241") // IP 변경
                .and()
                .addFilter(getAuthenticationFilter());

        http.headers().frameOptions().disable();
    }

    private AuthenticationFilter getAuthenticationFilter() throws Exception {
        AuthenticationFilter authenticationFilter =
                new AuthenticationFilter(authenticationManager(), userService, env);
//        authenticationFilter.setAuthenticationManager(authenticationManager());

        return authenticationFilter;
    }

    //select pwd from users where email=?
    // db_pwd(encrypted) == input_pwd(encrypted)
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService).passwordEncoder(bCryptPasswordEncoder);
    }
}
```
- 첫번째 configure()메소드의 permitAll()로직을 주석처리하고 통과시키고 싶은 IP 주소를 삽입, 추가적으로 필터를 추가해줌(권한 설정)
- getAuthenticationFilter() 메소드를 생성해 인스턴스를 반환해줌
- 쿼리없이 스프링 시큐리티의 로그인 기능을 활용하기 위해 authenticationManager를 사용해 인증 처리

## loadUserByUsername() 구현

WebSecurity.java - configure override
```java
    //select pwd from users where email=?
    // db_pwd(encrypted) == input_pwd(encrypted)
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService).passwordEncoder(bCryptPasswordEncoder);
    }
```
- userDetailService로 사용자 정보를 가져와 input 비밀번호를 암호화하고 이 데이터와 db 비밀번호를 비교함
- 생성자 주입을 통해 userService, bCryptPasswordEncoder 빈으로 사용


UserServiceImpl.java - loadUserByUsername() 메소드
```java
@Service
public class UserServiceImpl implements UserService{
    UserRepository userRepository;
    BCryptPasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserEntity userEntity = userRepository.findByEmail(username);

        if(userEntity == null){
            throw new UsernameNotFoundException(username);
        }

        return new User(userEntity.getEmail(), userEntity.getEncryptedPwd(),
                true, true, true, true,
                new ArrayList<>());
    }
}
```
- UserService 인터페이스에 UserDetailsService를 상속받는 코드를 추가 후 UserServiceImpl에 관련 메소드 코드를 구현한다.
- username 즉 email로 사용자 정보를 가져오고 사용자 정보를 담는 User객체에 그 내용을 담아 반환시켜준다.

## Routes 정보 변경

application.yml(apigateway-service)
```yml
routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user-service/login
            - Method=POST
          filters:
            - RemoveRequestHeader=Cookie
            - RewritePath=/user-service/(?<segment>.*), /$\{segment}
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user-service/users
            - Method=POST
          filters:
            - RemoveRequestHeader=Cookie
            - RewritePath=/user-service/(?<segment>.*), /$\{segment}
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user-service/**
            - Method=GET
          filters:
            - RemoveRequestHeader=Cookie
            - RewritePath=/user-service/(?<segment>.*), /$\{segment}
            - AuthorizationHeaderFilter
```
- apigateway-service의 yml파일 루트 설정 중 유저 서비스 부분을 로그인, 회원가입, 나머지로 나누어준다.
- 정규표현식으로 user-service의 값을 치환해준다. 따라서 RequestMaaping의 url정보를 삭제한다.


## 로그인 처리 및 성공 처리 과정

![0](https://user-images.githubusercontent.com/52458791/142808026-40df88a1-91f5-4135-a567-f23180e157e2.png)

![00](https://user-images.githubusercontent.com/52458791/142808030-b4b77188-74f8-47ef-8a81-185a90252ac1.png)

![000](https://user-images.githubusercontent.com/52458791/142808032-8ca057ce-3b00-4ac1-83be-f445a99536bf.png)

- 위와 같이 디버그 모드 및 포스트맨을 활용해 데이터가 어떻게 들어가는지 테스트 한다.

![0000](https://user-images.githubusercontent.com/52458791/142808034-7125574d-6649-4e60-90dd-57aacd1e5d43.png)
- 사용자가 보낸 유저 정보가 어떤 흐름을 타는지 확인할 수 있다.



![3](https://user-images.githubusercontent.com/52458791/142806027-4bc3d4c4-2810-4cd2-bfd0-084f7f8381af.png)

AuthenticationFilter.java - successfulAuthentication 메소드
```java
@Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {
        String username = ((User)authResult.getPrincipal()).getUsername();
        UserDto userDetails = userService.getUserDetailsByEmail(username);
}
```
- userDto에 유저 정보가 제대로 들어오는지 테스트

![33](https://user-images.githubusercontent.com/52458791/142808035-4d444138-c402-45c5-8146-84486635e450.png)

![333](https://user-images.githubusercontent.com/52458791/142808037-fd2b3500-fa0e-4110-871d-580c6af56319.png)
- 생성자 생성, userDto를 반환하기 위한 메소드 생성 등 추가적인 코드를 추가한다.


## JWT 생성 및 처리 과정

![4](https://user-images.githubusercontent.com/52458791/142809179-e5defbed-67cb-49be-ad82-7f387337ce76.png)
- 사용자객체의 아이디를 사용해 jwt를 생성한다.
- 어떤 내용을 가지는지, 유효기간(yml에 property 설정), 암호화 방식을 설정하여 토큰을 생성해준다.
- 위변조 방지를 위해 내가 가진 토큰 값과 인풋 토큰 값을 비교해준다.

AuthenticationFilter.java - successfulAuthentication() 구현
```java
@Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {
        log.debug(((User)authResult.getPrincipal()).getUsername());
        String username = ((User)authResult.getPrincipal()).getUsername();
        UserDto userDetails = userService.getUserDetailsByEmail(username);

        String token = Jwts.builder()
                .setSubject(userDetails.getUserId())
                .setExpiration(new Date(System.currentTimeMillis() + Long.parseLong(env.getProperty("token.expiration_time"))))
                .signWith(SignatureAlgorithm.HS512, env.getProperty("token.secret"))
                .compact();

        response.addHeader("token", token);
        response.addHeader("userId", userDetails.getUserId());
    }
```
- Jwts 메소드를 사용해 스트링 토큰값을 생성한다.
- userId를 subject로 설정, 토큰 만료기한(현재시간 + 1일, 밀리세컨드 단위), 문자열은 token.secret를 조합키로 사용하고 HS512알고리즘으로 암호화

![1](https://user-images.githubusercontent.com/52458791/142810287-5d0aede0-552e-4bfe-884b-f52176387f17.png)
- 세션과 쿠키는 이기종간에 공유가 안됨(ex. 모바일 디바이스), 모바일 디바이스는 다른 언어로 개발되기도 하고 클라이언트가 자바가 아닌 언어로 개발되면 공유가 안됨
- 모바일 어플리케이션은 HTML이 아닌 다른 포맷 필요함

![11](https://user-images.githubusercontent.com/52458791/142815959-b4246ce7-e9e9-44bf-be42-4b7fac3a393b.png)

- 이전과는 다르게 JWT토큰을 발급해준다.
- 토큰을 받은 클라이언트가 이를 이용해 다른 기능을 사용한다.

### JWT란
- 인증 헤더 내에서 사용되는 토큰 포맷
- 두 개의 시스템끼리 안전한 방법으로 통신 가능
- https://jwt.io/ 사이트에서 토큰 값 변환 및 확인 가능
- 다양한 언어 지원(Ruby, .NET, JS 등등)
- 장점
![111](https://user-images.githubusercontent.com/52458791/142815965-4a811c00-ff8e-47ef-b952-8be800bc8bb1.png)



## AuthorizationFilter 추가

apigateway-service > AuthorizationHeaderFilter.java
```java
@Component
@Slf4j
public class AuthorizationHeaderFilter extends AbstractGatewayFilterFactory<AuthorizationHeaderFilter.Config> {
    Environment env;

    public AuthorizationHeaderFilter(Environment env){
        super(Config.class);
        this.env = env;
    }

    public static class Config{

    }

    // login -> token -> users (with token) -> header(include token)
    @Override
    public GatewayFilter apply(Config config) {
        return ((exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();

            if(!request.getHeaders().containsKey(HttpHeaders.AUTHORIZATION)) {
                return onError(exchange, "no authorization header", HttpStatus.UNAUTHORIZED);
            }

            String authorizationHeader = request.getHeaders().get(org.springframework.http.HttpHeaders.AUTHORIZATION).get(0);
            String jwt = authorizationHeader.replace("Bearer", "");

            if(!isJwtValid(jwt)) {
                return onError(exchange, "JWT token is not valid", HttpStatus.UNAUTHORIZED);
            }

            return chain.filter(exchange);
            });
    }

    private boolean isJwtValid(String jwt) {
        boolean returnValue = true;

        String subject = null;

        try{
            subject = Jwts.parser().setSigningKey(env.getProperty("token.secret"))
                    .parseClaimsJws(jwt).getBody()
                    .getSubject();
        }catch (Exception ex){
            returnValue = false;
        }

        if(subject == null || subject.isEmpty()){
            returnValue = false;
        }

        return returnValue;
    }


    // Mono, Flux -> Spring WebFlux
    private Mono<Void> onError(ServerWebExchange exchange, String err, HttpStatus httpStatus) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(httpStatus);

        log.error(err);
        return response.setComplete();
    }
}
```
- apply 메소드를 오버라이드하여 구현, 인증이 되었는지 확인하고 관련 로직 생성
- 로그인하고 반환받은 토큰을 받고 이를 가지고 유저 정보를 가져옴
- onError 메소드를 구현하여 에러 발생시 해당하는 에러 메시지를 반환해줌
- bearer 글자를 삭제해주고 유효성 검사 메소드를 만들어줌
- jwt안의 subject를 추출하고 이를 확인해 값이 유효한지 확인한다.
- Spring Webflux의 타입을 사용함(Mono -> 0~1개의 값)


## 참고

- SpringWebflux, Mono, Flux - https://devuna.tistory.com/108
- 세션과 쿠키, 토큰 - https://tofusand-dev.tistory.com/89
- Jwt란? - https://mangkyu.tistory.com/56
- token 기반 인증 - https://behonestar.tistory.com/37