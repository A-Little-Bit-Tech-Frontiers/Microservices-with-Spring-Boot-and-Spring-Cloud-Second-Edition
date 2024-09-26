# 스프링 클라우드 소개

| 디자인 패턴 | 컴포넌트 |
| --- | --- |
| 서비스 검색 | Netflix Eureka |
| 에지 서버 | Spring Cloud Gateway |
| 구성 중앙화 | Spring Cloud Config |
| 서킷 브레이커 | Resilience4j |
| 분산 추적 | Spring Cloud Sleuth 및 Zipkin |


## 넷플릭스 유레카를 검색 서비스로 사용

유레카에 마이크로서비스를 등록하고, 클라이언트가 HTTP 요청을 유레카에 등록된 인스턴스로 전송한다.

<br>

## Spring Cloud Gateway를 에지 서버로 사용

<img width="532" alt="image" src="https://github.com/user-attachments/assets/dedeb906-faa9-4e65-89ff-134c2c3f821e">

스프링 클라우드 게이트웨이는 스프링 5와 프로젝트 리액터, 스프링 부트 2 기반의 논블로킹 API를 사용한다. 즉, 스프링 클라우드 게이트웨이는 Zuul v1에 비해 더 많은 양의 동시 요청을 처리할 수 있다. 

스프링 클라우드 게이트웨이는 아래와 같은 작업을 수행할 수 있다.

- 마이크로서비스를 설정
- 스프링 시큐리티 OAuth2를 함께 사용해 OAuth2.0과 OIDC를 기반으로 에지 서버에 대한 접근을 보호할 수 있다.
- 호출자의 신원 정보(예: 발신자의 사용자 이름, 이메일 주소)를 마이크로서비스로 전달

<br>

## 구성 중앙화를 위해 Spring Cloud Config 사용

Spring Cloud Config는 아래와 같은 다양한 저장소에 구성 파일을 저장할 수 있다.

- 깃 저장소
- 로컬 파일 시스템
- 하시코프 볼트
- JDBC 데이터베이스

Spring Cloud Config는 계층 구조로 구성을 관리할 수 있다. 예를 들어, 공통 구성 정보는 공통 파일에, 특정 마이크로서비스 구성 정보는 별도의 구성 파일에 배치할 수 있다.

Spring Cloud Config는 구성 변경을 감지하며, Spring Cloud Bus를 사용해 영향받는 마이크로서비스로 알림을 보내는 기능도 지원한다. Spring Cloud Bus는 Spring Cloud Stream을 바탕으로 카프카 혹은 RabbitMQ를 사용해 알림을 전송한다

<img width="562" alt="image" src="https://github.com/user-attachments/assets/4c666cf1-a4e2-4c8a-8168-72e4af4d3b07">

협력 관계

1. Config 서버에 구성 요청
2. 구성 파일 저장소로부터 구성 가져옴
3. 깃 저장소에 깃 커밋이 푸시되면 컨피그 서버에 알림을 보내도록 구성 가능(선택)
4. Config 서버는 클라우드 버스를 통해 변경 이벤트를 게시 함. 변경에 영향을 받는 마이크로서비스는 이에 반응함

<br>

## 복원력 향상을 위해 Resilience4j 사용

**Resilience4j**

- 서킷 브레이커: 원격 서비스가 응답하지 않을 경우에 발생하는 연쇄 장애를 방지하고자 사용
- rate limiter: 지정한 시간 동안의 서비스 요청 수를 제한하고자 사용
- bulkhead: 서비스에 대한 요청 수를 제한하고자 사용
- 재시도: 때때로 발생하는 임의의 오류를 처리하고자 사용
- time limiter: 오랫동안 느리거나 응답이 없는 서비스의 응답을 기다리지 않고자 사용

<img width="413" alt="image" src="https://github.com/user-attachments/assets/5ca8839a-fc72-422a-8ccd-0238bd509099">

<br>

## Spring Cloud Sleuth와 Zipkin을 사용한 분산 추적

스프링 클라우드에서 제공하는 Sleuth는 상관 ID(corelation ID)를 사용해 하나의 처리 흐름에 포함된 요청과 메시지/이벤트에 표시를 남긴다. Sleuth는 추적 데이터를 저장 및 시각화하고자 분산 추적 시스템인 zipkin으로 보낸다.

Sleuth와 Zipkin이 분산 추적 정보를 처리하고자 사용하는 인프라는 구글 대퍼(Google Dapper)를 기반으로 한다. 
대퍼에서는 전체 워크플로의 추적 정보를 **추적 트리(trace tree)**라고 부르며, 기본 작업 단위와 같은 트리의 하위 집합을 **스팬(span)**이라고 부른다. 
스팬은 추적 트리를 형성하는 **하위 스팬(sub-span)**으로 구성된다. 
상관 ID는 TraceId라고도 부르며, 소속된 추적 트리의 TraceId와 고유한 SpanId를 사용해 스팬을 식별한다.

<img width="616" alt="image" src="https://github.com/user-attachments/assets/39b4b023-947c-4206-ba2a-7374382ae7ee">

 마이크로서비스에 Zipkin 서버에 대한 런타임 의존성을 추가하고 싶지 않다면 메시지큐를 통해 비동기 방식으로 추적 정보를 전송하는 것이 좋다.
