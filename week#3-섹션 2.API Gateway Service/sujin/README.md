- Netflix Ribbon
  1. Spring Cloud에서 MSA간 통신 - RestTemplate, Feign Client
  2. 리본: 클라이언트측 Load Balancer, 서비스 이름으로 호출 가능

- Netflix Zuul (Client에서 Microservice를 호출하는데 있어서 사용하는 gateway)
  1. Spring Boot 2.4 기준 현재 보류상태 쓰기 위해서는 이전 버전 설정 필요
  2. Ribbon -> Spring Cloud Loadbalancer / Zuul -> Spring Cloud Gateway로 대체 사용
  3. @EnableZuulProxy로 zuul 서비스 등록
  4. zuul-routes 속성으로 microservice 등록하면 통일되어있는 zuul port에서 다른 서비스들 바로 조회 가능