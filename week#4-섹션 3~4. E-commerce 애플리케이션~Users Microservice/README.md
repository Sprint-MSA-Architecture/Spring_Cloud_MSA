- ## E-commerce 애플리케이션

  1. ##### 전체 구조

     <img src="https://user-images.githubusercontent.com/64065318/138685784-badee362-3efa-4132-b13c-63e237d271a6.png">

  2. ##### 제약사항

     - 주문시 재고수량에서 -1, 재고수량이 0일 경우 주문 불가해야
     - Messaging 서비스 사용(메세지를 발행하는 쪽 / 메세지를 얻어가는쪽)
     - 사용자 주문 -> DB에 해당 데이터 저장 -> ORDER-SERVICE는 Kafka에 해당 내용 전달 ->  CATALOG-SERVICE는 구독을 했으므로 변경사항이 들어오면 받음

  3. ##### 전체 애플리케이션 구성

     <img src="https://user-images.githubusercontent.com/64065318/138687029-261b77a6-65df-4cf4-9343-54b8ab52450a.png">

     - Configuration Service에는 3개 서비스에 대한 설정 저장하고 동적으로 참조하여 사용(yml 파일 사용 지양)
     - 마이크로서비스 부하 분산/서비스 라우팅을 위해 Api Gateway Server 사용

  4. ##### 본래 서비스 구성도(참고사항)

     <img src="https://user-images.githubusercontent.com/64065318/138687659-e9d654ea-65f4-41d1-82ec-0c7738778845.png">

     - docker를 이용해 3가지 서비스 배포
     - 개인 git에서 push/commit하면 CI/CD Pipeline 통해 자동으로 배포되는 과정 통함
     - Client로 요청이 들어오면 NGINX를 통해 3가지 서비스와 연결



### Chapter 1. Users Microservice

1. ##### 전체 구성

   <img src="https://user-images.githubusercontent.com/64065318/138688395-ff9e4419-2a66-48e7-a3df-4f50d4a36b2f.png">

2. ##### API Gateway 사용 여부에 따르 API

   - O(ex. 8000포트 사용, 간접 호출) : /users-service/users
   - X(ex. 8081포트 사용, 직접 호출) : /users

3. ##### 구현

   - Application에 @EnableDiscoveryClient 추가

     ```java
     @RestController
     @RequestMapping("/") // ex. :8001/과 같이 입력시 호출
     public class UsersController {
       
       // @Autowired로 바로 주입 받아도 되지만 이 방법 지양
       private Environment env; // @Value도 사용 가능, 해당 방법은 하단 이미지 참고
       
       // 생성자를 통한 주입
       @Autowired
       public UsersController(Environment env) {
         this.env = env;
       }
       
       @GetMapping("/health_check") // 상태체크
       public String status() {
         return "Its working in User Service"
       }
       
       @GetMapping("/welcome") // 웰컴멘트
       public String welcome() {
         return env.getProperty("greeting.message"); // yml 파일의 메세지 출력
       }
     }
     ```

   <img src="https://user-images.githubusercontent.com/64065318/138689971-9120ded4-da1f-46d4-bbcb-dc538c7e2fd3.png">

   - yml 파일 greeting-message: 로 웰컴 멘트 설정 가능

   - Greeting 객체는 VO Package 내에 선언

   - 참고 : Lombok Plugin 주요 내용 정리

     ```java
     // 파라미터가 없는 기본 생성자를 생성
     @NoArgsConstructor
     // 모든 필드 값을 파라미터로 받는 생성자
     @RequiredArgsConstructor 
     // final이나 @NonNull인 필드 값만 파라미터로 받는 생성자
     @AllArgsConstructor 
     // toString 자동 생성,  exclude 속성을 사용시, 특정 필드를 toString() 결과에서 제외시킬 수 있음
     // 클래스명(필드1명=필드1값,필드2명=필드2값,...) 식으로 출력
     @ToString(exclude = "필드이름") 
     
     // @Getter, @Setter, @RequiredArgsConstructor, @ToString, @EqualsAndHashCode을 한꺼번에 설정
     @Data
     ```

     - @Data 지양해야하는 이유 :  객체가 서로를 참조할때 toString을 서로 계속 호출하면 무한 반복이 되기 때문. include/exclude 속성을 이용해서 toString() 작성 시에 포함하거나, 빼야 하지만 `@Data`는 설정이 불가능.

4. ##### H2 Database

   - 자바로 작성된 오픈소스 RDMS

   - Embedded, Server-Client 가능

   - 별도의 설치 없이 JPA 사용 가능

   - yml 파일 설정

     ```yml
     spring:
     	h2:
     		console:
     			enable: true # 데이터 테이블 생성됐는지 등 쿼리 사용 가능
     			settings:
     				web-allow-others: true
     			path: h2-console # 포트번호/path로 h2 연결
     		datasource: # 해당 코드가 있어야 JPA에서 테이블 자동 생성
     			driver-class-name: org.h2.Driver
     			url: jdbc:h2:mem:testdb
     			# 만약 유저 정보가 필요하다면 추가
     			username: sa
     			password: 1234
     ```

   - 서버가 기동됨과 동시에 디비 사용가능

5. ##### Users Microservice - 회원 가입

   <img src="https://user-images.githubusercontent.com/64065318/138695719-4c522a53-d918-484a-a1dc-24aac3588f4f.png">

   - `RequestUser`: Client로 부터 전달받는 객체

   - `UserDto`:  계층 간 데이터 교환을 위한 객체(Java Beans). DB에서 데이터를 얻어 Service나 Controller 등으로 보낼 때 사용하는 객체를 말한다.

   - `UserEntity`: 실제 DB의 테이블과 매칭될 클래스. 즉, JPA(Java 코드에서 쿼리를 다루지 않고 API를 통해 통신)를 위한 객체

   - CrudRepository 상속받아 사용하여 기본적인 CRUD문을 이용한 기능 구현

   - `ModelMapper`

     ###### < 참고사항> 

     [ModelMapper 참고링크]: https://baek.dev/post/15/	"찾기 힘든 버그를 유발하는 Java DTO 컨버팅 노가다, 리팩토링하기"

     

     * 특정 Entity는 다른 Entity들과 관계맵핑 되어있음 -> 해당 객체를 Client에 바로 response-> 관계가 있는 모든 Entity에 대한 쿼리문을 날리게됨

     * 이외에도 필드를 잘못 매핑하거나 신규 필드 추가 시 매핑 비용 부수적으로 추가되는 문제 발생

       ```java
       ModelMapper modelMapper = new ModelMapper();
       // 1 : Dto <-> Entity
       UserEntity userEntity = modelMapper.map(userDto, UserEntity.class); // userDto라는 객체를 userEntity로 바꿔줌
       
       // 2 : RequestUser <-> Dto
       UserDto userDto = modelMapper.map(requestUser, UserDto.class);
       ```

       

     * 자주 사용한다면 매번 선언하지말고 @Bean 으로 등록해놓고 @Autowired로 사용하는 방법도 있음

   - Validation Check

     ```java
     @NotNull(message=" null일 경우 메세지 ")
     @Size(min = 최소길이, message=" 최소 길이 지켜지지 않을 경우 메세지")
     ```

   - UserEntity Class

     ```java
     @Data
     @Entity
     @Table(name = "users") // 테이블 이름 명시
     public class UserEntity {
     	@Id
     	@GeneratedValue(strategy = GenerationType.IDENTITY)
     	private Long id;
     	@Column(nullable = false, length = 50, unique = true)
     	private String email;
     	@Column(nullable = false, length = 50)
     	private String name;
     	@Column(nullable = false, unique = true)
     	private String userId;
     	@Column(nullable = false, unique = true)
     	private String encrpytedPwd;
     }
     ```

   - UserRepository Class

     ```java
     public interface UserRepository extends CrudRepository<UserEntity, Long>
     ```

   - UserServiceImpl Class

     ```java
     @Service
     public class UserServiceImpl implements UserService {
       UserRepository userRepository;
       BcryptPasswordEncoder passwordEncoder;
     
       // UserServiceApplication에 인코더 등록해놔야 사용가능(단, 타입이 같으면)
       // @Bean
       // public BcryptPasswordEncoder passwordEncoder() {
       // return new BcryptPasswordEncoder();
     	// }
       
       @Autowired
       public UserServiceImpl(UserRepository userRepository, BcryptPasswordEncoder passwordEncoder) {
         this.userRepository = userRepository;
       }
     
       @Override
       public UserDto createUser(UserDto userDto) {
         userDto.setUserId(UUID.randomUUID().toString());
     
         // ModelMapper 설정 및 변환
         ModelMapper mapper = new ModelMapper();
     
         // 완벽히 매칭하지 않으면 안되는 최고 규칙 적용
        mapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT); 
         UserEntity userEntity = mapper.map(userDto, UserEntity.class);
         userEntity.setEncryptedPwd(passwordEncoder.encode(userDto.getPwd())); // DB에 암호화된 비밀번호 저장됨
     
         userRepository.save(userEntity); 
     
         UserDto returnUserDto = mapper.map(userEntity, UserDto.class);
     
         return returnUserDto;
     	}
     }
     ```

     - UserController Class 변동되는 부분

       ```java
       @Autowired
       public UserController(Environment env, UserService userService) {
       	this.env = env;
       	this.userServce = userService;
       }
       
       // 회원가입
       @PostMapping("/users")
       public ResponseEntity createUser(@RequestBody RequestUser requestUser) {
       	ModelMapper mapper = new ModelMapper();
       	mapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT); 
       	UserDto userDto = mapper.map(userRequest, UserDto.class);
         
         return new ResponseEntity(HttpStatus.CREATED);
       }
       ```

       - ###### <참고사항> 왜 @Autowired로 바로 주입받지 않고 생성자로 사용하는지 

         [생성자 주입과 @Autowired 주입 차이]: https://madplay.github.io/post/why-constructor-injection-is-better-than-field-injection	"생성자 주입을 @Autowired를 사용하는 필드 주입보다 권장하는 하는 이유"

         

         - 빈을 주입하는 순서가 다르기 때문
           1. @Autowired 방식 : 먼저 빈을 생성한 후에 어노테이션이 붙은 필드에 해당하는 빈을 찾아서 주입하는 방법(=필요시 주입)
           2. 생성자 방식 : 생성자로 객체를 생성하는 시점에 필요한 빈을 주입(=바로 주입)
         - **순환 참조 오류**는 (2) 이용시 오류로 뜨지만 (1) 이용시 오류로 검출되지 않음 (모든 로직이 맞다고 생각하고 추가 개발을 진행하고 있는데 순환 참조로 인해 로직이 꼬이는 현상 발생) 따라서, 순환 참조가 있는 객체 설계는 잘못된 설계이기 때문에 **오히려 생성자 주입을 사용하여 순환 참조되는 설계를 사전에 막아야 한다.**

6. ### Users MicroService - Security

   - Spring Security : Authentication(인증) + Authorization(인증 받은 뒤 할 수 있는 권한 설정)

     <img src="https://user-images.githubusercontent.com/64065318/138706104-e03158a7-5011-4ef2-9296-8c3f0f84b5c0.png">

   - WebSecurity Class (Rest API 접근 설정)

     ```java
     @Configuration
     @EnableWebSecurity
     public class WebSecurity extends WebSecurityConfigurerAdapter {
     
     	@Override
     	protected void configure(HttpSecurity http) throws Exception {
         // basic auth를 사용하기 위해 csrf 보호 기능 disable
     		http.csrf.disable();
     		// 해당 url에 대해 모든 권한 허용
     		http.authorizeRequests.antMatchers("/users/**".permitAll());
     		// 해당 설정 없으면 h2-console 접근되지않음
     		http.headers().frameOptions().disable();
     	}
     }
     ```

   - BcryptPasswordEncoder -> 상단 UserServiceImpl Class 참고

     1. 패스워드 같아도 암호화된 결과는 다름
     2. 패스워드를 해싱하기위해 Bcrypt 알고리즘을 사용
     3. 랜덤 Salt를 부여하여 여러번 해시를 적용한 암호화 방식

