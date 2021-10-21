- ### Netflix Ribbon

  1. Spring Cloud에서 MSA간 통신 - RestTemplate, Feign Client
  2. 리본: 클라이언트측 Load Balancer, 서비스 이름으로 호출 가능

- ### Netflix Zuul (Client에서 Microservice를 호출하는데 있어서 사용하는 gateway)

  1. Spring Boot 2.4 기준 현재 보류상태 쓰기 위해서는 이전 버전 설정 필요
  2. Ribbon -> Spring Cloud Loadbalancer / Zuul -> Spring Cloud Gateway로 대체 사용
  3. @EnableZuulProxy로 zuul 서비스 등록
  4. zuul-routes 속성으로 microservice 등록하면 통일되어있는 zuul port에서 다른 서비스들 바로 조회 가능

- #### Spring Cloud Gateway (zuul 다음버전)

  1. 동기방식 -> 비동기 방식 변경
  2. eureka-client 개념이 아닌 Spring-cloud-gateway-routes (id, url, predicates)

  ------

  

- #### Spring Cloud Gateway - Filter

  1. FilterConfig.java(자바코드에 적용)

     ```java
     @Configuration
     public class FilterConfig {
     	@Bean
     	public RouteLocator gatewayRoutes(RouteLocatorBuilder builder) 		{ 
     			1. route 정보(해당 path에 대한 요청이 들어오면 uri로 이동, 이때 필터를 적용하겠다 addRequestHeader / addResponseHeader)
     		}
     }
     ```

     

2. application.yml 파일에 설정

```yml
cloud:
	gateway:
		routes:
			- id:
				filters:
					- AddRequestHeader
					- AddResponseHeader
```

------



- #### Spring Cloud Gateway - Custome Filter (사전, 사후에 이루어져야할 동작 정의)

1. SserverHttpRequest, ServerHttpResponse 사용 -> application.yml 파일에 등록 or FilterConfig에 정의

```java
@Component
@Slf4j
public class CustomFilter extends AbstractGatewayFilterFactory<CustomFilter, Config> {
	public CustomFilter {
		// Custom Pre Filter
		return (exchange, chain) -> {
			SeverHttpRequest, ServerHttpResponse
			
			// Custom Post Filter
			return chain.filter(exchange).then(Mono.fromRunnable() -> {
				
			})
		};
	}
	public static class Config {
		// Put the configuration properties
	}
}
```

------



- #### Spring Cloud Gateway - Global Filter

1. 어떤 라우트가 실행된다해도 공통적으로 실행되는 필터

```yml
cloud:
	gateway:
		default-filters:
			- name: GlobalFilter # 글로벌 필터 설정한 파일 이름
				args:
					baseMessage: SpringCloud Gateway Global Filter
					preLogger: true
					postLogger: true
```

 	2. 글로벌 필터가 종류되기 전에 커스텀 필터도 작동됨

- #### Spring Cloud Gateway - Custom Filter(Logging Filter)

  1. 처리프로세스: Gateway Client -> Gateway Handler -> Global Filter -> Custome Filter -> Logging Filter -> Proxied Service

     ```java
     GatewayFilter filter = new OrderedGatewayFilter((exchange, chain) -> {
       // log.info 정의
     }, Ordered.HIGHEST_PRECEDENCE); // 우선 순위를 높게 잡으면 위에 처리프로세스와 다르게 우선적으로 수행된다
     ```

     추가적인 filter 원할 시에

     ```yml
     filters:
     	- name: CustomFilter
     	- name: LoggingFilter # 기존 커스텀 필터이름만 쓰는 것과 다르게 name: 이름 추가 필요
     		args:
     			baseMessage:
     			preLogger:
     			postLogger:
     ```

     ------

     

- #### Spring Cloud Gateway - Eureka 연동

  1. Eureka Server의 역할: Service Discovery, Service Registration

     ![스크린샷 2021-10-19 오후 12.01.29](/Users/tudin/Library/Application Support/typora-user-images/스크린샷 2021-10-19 오후 12.01.29.png)

  2. 유레카 서버에 있는 위치를 이용할 경우 다른 부분

```yml
routes:
 - id: first-service
 	 uri: lb://MY-FIRST-SERVICE # 포트번호 명시 없이 로드발랜서 통해서 가는 경로로 바꿔줘야함
 	 predicates:
 	 	- Path=/first-service/**
```

------



- ### Spring Cloud Gateway - Load Balancer

  1. Client Service 기동 방법 3가지: VM option, mvn 명령어, java 명령어

  2. 해당 서비스 2개 이상 작동 중일때 (포트번호 다름) 이 중 어떤 서비스가 작동되고 있는지 확인하는 방법

     1. port 번호 0 으로 바꾸고 랜덤포트 사용

     2. instance id 값 부여

        ```yml
        eureka:
        	instance:
        		instance-id: ${spring.application.name}:{spring.application.instance_id:${random.value}}
        ```

        

  3. ServiceController

     ```java
     Environment env;
     
     @Autowired
     public FirstServiceController(Environment env) {
     	this.env = env;
     }
     
     // 서버 포트 가져오는 2가지 방법
     log.info("Server port={}", request.getServerPort());
     log.info("Server port=%s", env.getProperty("local.server.port"));
     
     ```

     