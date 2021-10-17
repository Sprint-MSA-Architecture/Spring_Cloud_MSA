- Netflix Ribbon
  1. Spring Cloud에서 MSA간 통신 - RestTemplate, Feign Client
  2. 리본: 클라이언트측 Load Balancer, 서비스 이름으로 호출 가능
- Netflix Zuul (Client에서 Microservice를 호출하는데 있어서 사용하는 gateway)
  1. Spring Boot 2.4 기준 현재 보류상태 쓰기 위해서는 이전 버전 설정 필요
  2. Ribbon -> Spring Cloud Loadbalancer / Zuul -> Spring Cloud Gateway로 대체 사용
  3. @EnableZuulProxy로 zuul 서비스 등록
  4. zuul-routes 속성으로 microservice 등록하면 통일되어있는 zuul port에서 다른 서비스들 바로 조회 가능

- Spring Cloud Gateway (zuul 다음버전)

  1. 동기방식 -> 비동기 방식 변경
  2. eureka-client 개념이 아닌 Spring-cloud-gateway-routes (id, url, predicates)

  

- Spring Cloud Gateway - Filter

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



- Spring Cloud Gateway - Custome Filter (사전, 사후에 이루어져야할 동작 정의)

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

- Spring Cloud Gateway - Global Filter
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

​	2. 글로벌 필터가 종류되기 전에 커스텀 필터도 작동됨