# Secotion 0. Microservice와 Spring Cloud의 소개

## IT 시스템의 역사

- 60~80년대: Fragile, Cowboys -> 하드웨어 성격에 맞춤
- 90~00년대: Robust, Distributed -> 안정성을 높이고 분산된 시스템 구축
- 10년대 이후: Resilient/Anti-Fragile, Cloud Native -> 클라우드 기반으로 지속적으로 개선됨, DevOps 문화 발생


![image](https://user-images.githubusercontent.com/52458791/136146975-154e9cc2-e71e-474b-85bf-f5e434c2090f.png)
 
## Antilfragile 특징
- Auto scaling -> 데이터 사용량 등 조건에 따라 자동으로 리소스가 스케일링됨
- Microservices -> 하나의 큰 시스템을 모듈별로 세분화함
- Chaos engineering -> 시스템이 예측하지 못한 상황, 불확실성을 견딤
- Continuous deployments -> 파이프라인을 활용한 지속적인 배포

## Cloud Native Architecture 특징

### 확장 가능한 아키텍처
- 시스템의 수평적 확장에 유연
- 확장된 서버로 시스템의 부하 분산, 가용성 보장
- 시스템 또는, 서비스 애플리케이션 단위의 패키지(컨테이너 기반 패키지)
- 모니터링
- 클라우드 업체를 사용해 리소스 비용 감소

### 탄력적 아키텍처
- 서비스 생성-통합-배포, 비즈니스 환경 변화에 대응 시간 단축
- 분할된 서비스 구조
- 무상태 통신 프로토콜 -> 추가적으로 설명!!
- 서비스의 추가와 삭제 자동으로 감지
- 변경된 서비스 요청에 따라 사용자 요청 처리(동적 처리)

### 장애 격리
- 특정 서비스에 오류가 발생해도 다른 서비스에 영향 주지 않음



## Cloud Native Application
- 클라우드 아키텍처의 특징과 antifragile특징을 모두 가짐
- CI/CD – 지속적 통합, 지속적 배포(자동화)

    Ex) 카나리 배포, 블루그린 배포
 
### **블루그린 배포**

![image](https://user-images.githubusercontent.com/52458791/136147066-322517fb-e483-4e55-8953-16c4db4fe637.png)

여기서 말하는 블루란 이전 버전을 뜻하고 그린은 새로운 버전을 뜻한다. 일단 이전 버전과 새로운 버전을 모두 띄우고 새로운 버전이 준비되었다고 판단할때 기존 버전의 연결을 서비스로부터 제거한다. 이 방식의 장점은 서비스 중단점이 없고, 버전 혼재에 따른 문제가 없다는 것이다. 다만 이전 버전과 새 버전을 동시에 실행하기 때문에 리소스가 두배로 필요하다는 단점이 있다. 이 방식은 서비스 매시(Service Mesh) Knative 같은 확장 서비스를 쓰지 않으면 블루 그린 배포는 수동으로 진행된다.
 
### **카나리 배포**

![image](https://user-images.githubusercontent.com/52458791/136147104-9b22cc22-d257-4086-9265-65358a0d203a.png)

먼저 카나리라는 이름의 뜻부터 알아보자. 카나리는 원래 광부들이 탄광에 들어가기 전에 갱도로 먼저 날려보내던 새다. 만약 새가 탄광 안으로 들어가서 나오지 못하면 유독 가스가 있다고 판단하여 내부로 들어가지 않았다고 한다. 카나리 릴리즈는 새로운 버전을 하나씩 올려보면서 해당 버전이 잘 동작한다 싶으면 기존 버전을 내린다. 그런 다음 또 다시 새로운 버전을 올리는 식으로 배포를 진행한다. 위험을 빠르게 감지할 수 있는 장점이 있다. 또한 A/B테스트가 가능하며, 성능 모니터링에 유용하다. 트래픽을 분산시킬 때는 라우팅을 랜덤하게 할 수 잇고, 사용자로 분류할 수도 있다.

**※참고 - A/B테스트**

**A/B 테스트**란 웹 사이트 방문자를 임의로 두 집단으로 나누고, 한 집단에게는 기존 사이트를 보여주고 다른 집단에게는 새로운 사이트를 보여준 다음, 두 집단 중 어떤 집단이 더 높은 성과를 보이는지 측정하여, 새 사이트가 기존 사이트에 비해 좋은지를 정량적으로 평가하는 방식을 말한다.

여기에서 성과란 새 사이트가 목표로 했던 바에 따라 다른데, 보통은 회원 가입율, 재방문율, 구매전환율 등의 지표를 본다.

과학 혹은 의학에서 무작위비교연구(RCT; Randomized-controlled trial)라 불리는 방법을 인터넷 마케팅에 적용한 것이라고 생각하면 된다.

주로 웹사이트의 마케팅과 관련하여 많이 쓰이지만 디자인, 인터페이스, 상품 배치 등을 개선하기 위해서 사용하기도 하며, 웹사이트가 아닌 모바일 앱, 게임 등의 분야에서도 활용된다.


- DevOps -> Development + Operations + QA

![image](https://user-images.githubusercontent.com/52458791/136147144-1f688c15-7fcd-43fd-8454-55ce170e8811.png)
 

- Container 가상화
 
![image](https://user-images.githubusercontent.com/52458791/136147191-c01fdd29-76b6-4bd8-97a7-fa9812e0242c.png)

## **12Factors** – 클라우드 서비스 구성시 고려해야 할 점들
### 1. 코드베이스 – Base Code
- 코드와 app 사이에는 1:1 관계가 성립해야함, 1개의 코드베이스에서 여러개의 앱(서비스)을 배포해선 안된다.
- SVN, GIT과 같은 VCM을 이용하여 코드 관리
- 마이크로서비스마다 코드베이스를 가짐, 다른 마이크로서비스와 공유되지 않음
- 공유가 필요한 코드는 라이브러리화해서 공유

### 2. 종속성(의존성) – Dependency Isolation
- app의 의존관계는 명시적으로 선언되어야 함 (maven인 경우 pom.xml, gradle인 경우 build.gradle, node인 경우 npm)
- 모든 라이브러리는 central - maven repository, nexus 와 같은 라이브러리 저장소에서 내려받을수 있어야함

### 3. 설정 - Configurations
- 설정정보는 app 코드에 저장하지 않고, 환경설정파일로 분리해서 사용해야 함 (표프 템플릿경우 : globals.properties)
- 로컬,개발,운영환경별 설정파일로 분리하여 사용하는것이 좋음
- MSA에서는 설정정보를 외부중앙서버에서 별도로 관리

### 4. 백엔드 서비스 – Linkable Backing Services
- app에서 네트워크로 접근하여 이용하는 모든서비스를 연결된 리소스로 취급
- rest api, db, cache, smtp, mq등등
- 예로 mysql을 사용하다가 app의 코드변경없이 설정변경만으로 oracle을 사용할 수 있어야 함 (jpa 의 dialect같은것)

### 5. 빌드, 릴리즈, 실행 – Stages of Creation
- 코드베이스는 3단계를 거쳐 배포로 변환된다.
- 빌드: 라이브러리와 소스를 내려받아 컴파일하여 하나의 패키지를 만듬 (예: jar, war 파일 생성)
- 릴리스: 빌드된 패키지에 환경설정 정보를 조합 (예: dockerfile을 이용하여 이미지 생성, 버전별로 기록되어 롤백가능)
- 실행: 릴리스에서 만들어진 결과물을 실행환경에서 애플리케이션을 실행 (예 : dockerimages를 내려받아서 컨테이너로 실행)

### 6. 프로세스 (stateless 프로세스) – Stateless Processes
- 애플리케이션을 하나 혹은 여러개의 무상태(stateless) 프로세스로 실행 (수평확장, 무상태)
- 무상태 장점: 수평확장하기에 용이하기 때문에 장애 대응성이 좋아짐 
- 상태유지가 필요한 데이터는 DB나 인메모리캐쉬같은 백엔드서비스를 이용하여 처리함

### 7. 포트 바인딩 – Port Binding
- 실행환경에서 app을 실행할때 특정 WAS에 의존적이지 않고 (예: php 경우 apache에 의존적, jsp 경우 tomcat등 was에 의존적)
- 내부에 tomcat과 같은 서블릿 컨테이너를 내장시켜 단독실행가능한 app으로 만들어야함. (예: spring boot)

### 8. 동시성 - Concurrency
- 수직확장이 아니라 수평확장을 통한 동시성을 말함(프로세스 복제, 동일서버증설)
- webapp 계층을 무상태계층으로 개발하면 상태를 가지지 않으므로 수평확장/축소를 자유롭게 할수 있음. (예: jwt 인증)

### 9. 폐기 가능 - Disposability
- 중지될때 진행중이던 프로세스는 모두 완료되어야 함 (graceful shutdown 지원)
- 가능한 빠른 시간내에 시작/종료되어야 함 (마이크로서비스를 작은 규모로 유지)
- 도커나 쿠버를 사용해 빠르게 서비스 중지 가능

### 10. 개발/프로덕션 환경 일치 – Development & Production Parity
- 배포환경의 차이는 크게 3가지
- 시간의 차이 (배포 간의 간격: 몇 주 vs 몇 시간)
- 담당자의 차이 (코드 작성자와 코드 배포자: 다른사람 vs 같은사람)
- 툴의 차이 (개발 환경과 production 환경: 불일치 vs 최대한 유사함)
=> 개발환경과 배포환경을 최대한 동일하게 유지하는것을 중요하게 생각함

### 11. 로그 - Logs
- 원문에서는 app은 로그파일을 작성하거나 관리하려해선 안되고, 로그를 버퍼링없이 표준출력(stdout)에 출력하라고 되어 있음.
- log4j같은 라이브러리를 이용해서 파일로 쓰는 것은 I/O 병목현상을 일으킬수 있어서 위반, 단 console에 쓰는것은 OK.
- 로그중앙화: Fluentd, Logstash등의 로그수집기를 이용하여 로그중앙집중화

### 12. Admin 프로세스 – Admin Processes For Eventual Processes
- 일회성 admin 프로세스는 실제 서비스하는 운영환경에서 실행돼야 함. (flyway 와 같은 마이그툴 사용)
- 여기서 말하는 일회성 admin 프로세스는 데이터 마이그레이션 script 또는 코드, shell script등을 말함
- 일회성 처리를 위한 기능들도 코드베이스와 함께 관리하고 배포되어야 함
-이런 점 때문에 12FactorApp 에서는 REPL Shell이 제공되는 언어를 선호 (일회성 스크립트 작성에 유리)
- Java 9버전부터 REPL Shell을 제공
- 관리도구를 사용해 리소스를 모니터링 할 수 있어야함

### Pivotal +3
- API first – 모든 서비스는 API로 제공되어야함
- Telemetry – 모든 지표를 수치화하고 시각화하여 관리 가능해야함
- Authentication and authorization – 인증 및 권한 관리를 사용해 데이터를 전달해야함

**※참고**

HTTP 특징(무상태성 프로토콜)

https://victorydntmd.tistory.com/286

## Monolithic vs MSA
 
![image](https://user-images.githubusercontent.com/52458791/136163767-1bd61116-0864-442b-bcd4-2202d99717a1.png)

### Monolith Architecture

모든 업무 로직이 하나의 애플리케이션 형태로 패키지 되어 서비스
애플리케이션에서 사용하는 데이터가 한곳에 모여 참조되어 서비스되는 형태
### Microservice
Small autonomous services that work together

중앙화를 최소화 하고 각 서비스마다 다른 언어와 다른 데이터 스토리지를 가져도 된다.
각 서비스에 맞는 언어와 환경을 선택한다.

![image](https://user-images.githubusercontent.com/52458791/136163960-73c6a233-95ef-450b-80cc-c594a429b4ce.png)

![image](https://user-images.githubusercontent.com/52458791/136163987-9311525d-cb03-4ab0-8653-5af66304958c.png)

### Microservice Architecture란?
베조스의 메일
1) 모든 팀은 서비스 인터페이스로 데이터와 기능을 공개하세요.
2) 팀들은 이 인터페이스를 통해 서로 통신 하세요.
3) 직접 링킹, 다른팀 저장소에 직접 억세스, 공유메모리, 백도어 등, 다른 어떤 통신방법도 허용되지 않습니다. 네트워크를 통한 서비스 인터페이스 호출만 허용합니다.
4) 어떤 기술을 사용하는가는 중요하지 않습니다. HTTP, Corba, Pubsub, 커스텀 프로토콜 다 괜찮습니다.
5) 모든 서비스 인터페이스는 예외없이 기초부터 모두 외부에서 사용 가능하도록 설계되어야 합니다. 즉, 팀들은 인터페이스를 외부 개발자가 이용가능하도록 계획하고 설계해야 한다는 것입니다. 예외는 없습니다.
6) 이를 지키지 않는 사람은 해고 될것입니다.
7) 고맙습니다. 좋은 하루 되세요!

MSA의 가장 본질적인 부분이 나왔습니다.
서로에 관여함 없이 독립된 배포를 합니다.

내부적으로 사용한 똑같은 플랫폼을 2006년 공개하였고 이것이 아마존 웹 서비스(AWS)입니다.

### Microservice의 특징
1) Challenges -> 개발의 패러다임을 바꾸어야함
2) Small Well Chosen Deployable Units -> 독립적으로 배포가능해야함
3) Bounded Context -> 각 서비스의 경계가 잘 구분되어야함
4) RESTful -> 서로 Rest API로 통신해야함
5) Configuration Management -> 환경 설정 정보는 외부의 시스템을 통해 관리해야함(하드 코딩해야 하는 정보 관리)
6) Cloud Enabled -> Cloud Native 기술을 이용해야함
7) Dynamic Scale Up And Scale Down -> 동적으로 스케일을 구성해야함
8) CI/CD -> 자동화 배포가 필요함
9) Visibility -> 이러한 서비스들을 시각화해야함

### 고려해야될점
Q1) Multiple Rates of Change

도입했을 때 어느 정도의 공수를 받아들일 수 있을 것인가

Q2) Independent Life Cycles

각 서비스들이 독립적으로 개발되고 운영되는지

Q3) Independent Scalability

서비스들의 확장성이 잘 고려되었는가

Q4) Failure Isolation

각 오류들이 잘 격리되고 독립적인가

Q5) Simplify Interactions with External Dependencies

외부 종속성과의 상호 작용을 단순화해 종속성을 줄이고 내부 응집도는 높여야한다

Q6) Polyglot Technology

각 서비스들이 여러가지 언어, DB 등을 사용할 수 있는가

### Microservice Team Structure 아마존 Devops팀의 규칙

- Two Pizza team -> 점심으로 피자 두판을 나눠먹을 수 있는 팀(4~5명)
- Teams communicating through API contracts
- Develop, test and deploy each service independently
- Consumer Driven Contract

## SOA vs MSA
### 서비스의 공유 지향점

SOA – 재사용을 통한 비용 절감

MSA – 서비스 간의 결합도를 낮추어 변화에 능동적으로 대응

### 기술 방식
SOA – 공통의 서비스를 ESB에 모아 사업 측면에서 공통 서비스 형식으로 서비스 제공
MSA – 각 독립된 서비스가 노출된 REST API를 사용

|아키텍처|특징|결합도|확장성|성공사례|기술적난이도|비용|
|---|---|---|---|---|---|---|
|Monolithic|하나의 프로젝트|매우 높음|매우 낮음|매우 많음|쉬움|낮음|
|SOA|여러 서비스, 하나의 버스|낮음|높음|적음|어려움|중간|
|Microservice|독립된 여러 서비스|매우 낮음|매우 높음|많음|어려움|매우높음|


## RESTful API란

### 성숙도 모델
 
![image](https://user-images.githubusercontent.com/52458791/136164056-c620eef1-cb6e-4bae-9696-67ab1f6353f2.png)

- LEVEL 0:
Expose soap web services in rest style

- LEVEL 1:
Expose resources with proper uri

- LEVEL 2:
Level1 + HTTP Methods

- LEVEL 3:
Level2 + HATEOAS
DATA + NEXT POSSIBLE ACTIONS

### 설계 고려사항

- Consumer first

- Make best use of HTTP

- Request methods -> GET/POST/PUT/DELETE Level2까지는 지켜야 한다

- Response Status -> 200//201/400/401/404 상황에 맞는 응답 코드 반환

- No secure info in URI

- Use plurals -> 단수 보다 복수 형태의 값을 사용해야함

- User nouns for resources -> 동사보다 명사형태로 표시하는게 좋다, 
사용자에게 직관성 제공

- For exceptions -> 일관적인 end point 사용하는 것이 좋다. 
Define a consistent approach


![image](https://user-images.githubusercontent.com/52458791/136164119-a84b5b9e-de85-4c5b-a6a2-e20fab2fe7b1.png)



 

### Microservices Architecture Components
 
![image](https://user-images.githubusercontent.com/52458791/136164179-ac4152a5-26d3-495a-8e41-2b6bca8f07b8.png)

### Service Mesh
 
![image](https://user-images.githubusercontent.com/52458791/136164236-4c8b3d1c-07ed-43ec-8f50-02322d283358.png)

MSA 인프라 -> 미들웨어
- 프로식 역할, 인증, 권한 부여, 암호화, 서비스 검색, 요청 라우팅, 로드 밸런싱
- 자가 치유 복구 서비스
서비스간의 통신과 관련된 기능을 자동화

## CNCF(Cloud Native Computing Foundation)
### Cloud Native Interactive Landscape
 
![image](https://user-images.githubusercontent.com/52458791/136164278-fcfd0e3a-c43b-44ad-a0fb-0eb016e2e949.png)


### 핵심 제품들

![image](https://user-images.githubusercontent.com/52458791/136164531-f6d8ad3f-136e-43d7-9a13-75509251d0ea.png)
 
## Spring Cloud

### 환경 설정
Centralized configuration management - Spring Cloud Config Server

Location transparency – Naming Server(Eureka)

Load Distribution(Load Balancing) – Ribbon(Client Side), Sping Cloud Gateway

Easier Rest Clients - FeignClient

Visibility and monitoring’ – Zipkin Distributed Tracing, Netflix API gateway

Fault Tolerance – Hystrix



## 참고

### soa vs msa vs monolithic

https://wonit.tistory.com/487

### 12 factors

https://medium.com/dtevangelist/12-factors-%EB%9E%80-b39c7ef1ed30
https://hhjeong.tistory.com/183

### CNCF와 Cloud Native

https://dalsacoo-log.tistory.com/entry/CNCF-Cloud-Native-Landscape

### 성숙도 모델

https://velog.io/@younge/REST-API-%EC%84%B1%EC%88%99%EB%8F%84-%EB%AA%A8%EB%8D%B8-Maturity-Model-eqqyjqff

