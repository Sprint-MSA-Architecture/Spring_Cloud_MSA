#### 각각의 microservice가 가져가는 갱신 정보 가져가는 방법

1. 서버 재기동

2.  Actuator refresh

   - Spring Boot Actuator -> Application 상태 모니터링, Metric 수집을 위한 Http End point 제공
   - 문제점: Appilcation server가 많을때 각각 refresh 진행해야하기 때문에 번거로움

3. Spring cloud bus 사용

   - 분산 시스템의 노드(Microservice)를 경량 메세지 브로커(RabbitMQ)와 연결

   - 상태 및 구성에 대한 변경 사항을 연결된 노드에게 전달(Broadcas(t)

   - (AS-IS) P2P 방식 -> (T0-BE) Middleware 통해 안정적인 전달, 송신/수신측 상대방의 환경 상관없이 자신의 일만 할 수 있도록 변경


![이미지1](../../../../../dev/Spring_Cloud_MSA/week#8-섹션 8. Spring Cloud Bus/images/이미지1.jpg)