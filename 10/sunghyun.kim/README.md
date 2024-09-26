# 스프링 클라우드 게이트웨이를 에지 서버로 사용

<img width="740" alt="스크린샷 2024-09-26 오후 9 20 32" src="https://github.com/user-attachments/assets/33dca836-7abe-4f91-9afb-4ae9b94e7929">


외부 클라이언트는 모든 요청을 에지 서버로 보낸다. 에지 서버는 URL 경로를 기반으로 들어오는 요청을 라우팅한다.

## 스프링 클라우드 게이트웨이 설정

### 복합 상태 점검 추가

외부에서의 상태 점검 요청은 에지 서버를 거쳐야 한다. <br>
모든 마이크로서비스 상태를 점검하는 복합 상태 점검도 에지 서버로 옮긴다.

<img width="686" alt="스크린샷 2024-09-26 오후 9 23 28" src="https://github.com/user-attachments/assets/392ce5b7-0a3b-4d9f-80ef-42ed3cdd4191">


## 스프링 클라우드 게이트웨이 구성

스프링 클라우드 게이트웨이를 구성할때 가장 중요한 점은 라우팅 규칙 설정이며, 몇 가지 다른 구성도 설정해야 한다.

1. 스프링 클라우드 게이트웨이는 넷플릭스 유레카를 사용해 트래픽을 보낼 마이크로서비스를 찾으므로 유레카 클라이언트를 구성해야 한다.
2. 개발 환경을 위한 스프링 부트 액추에이터를 구성한다.
3. 스프링 클라우드 게이트웨이의 내부 처리 로그를 보고자 로그 레벨을 구성한다.

```
logging:
  level:
    root: INFO
    org-springframework.cloud.gateway.route.RouteDefinitionRouteLocator: INFO 
    org.springframework.cloud.gateway: TRACE
```

### 라우팅 규칙

라우팅 규칙은 자바 DSL을 사용한 프로그래밍 방식이나, 구성 파일을 사용해 설정할 수 있다. <br>
자바 DSL을 사용해 프로그래밍 방식으로 설정하는 것은 규칙이 데이터베이스 등의 외부 저장소에 저장돼 있거나, RESTful API나 게이트웨이로 전송된 메시지를 통해 런타임에 제공되는 경우 유용한다. <br>
보통은 구성 파일에서 라우팅 경로를 설정하는 것이 더 편리하다.

1. predicate: 수신되는 HTTP 요청 정보를 바탕으로 경로를 선택
2. filter: 요청이나 응답을 수정
3. destination URI: 요청을 보낼 대상
4. ID: 라우팅 경로 이름

<br>

URL 경로가 /product-composite/로 시작하는 요청을 product-composite 서비스로 라우팅해야 한다면 라우팅 규칙을 다음과 같이 정의한다.

```
spring.cloud.gateway.routes:
- id: product-composite
  uri: lb://product-composite
  predicates:
  - Path=/product-composite/**
```

스웨거 UI로 요청을 라우팅할 수 있도록 product-composite 서비스에 대한 라우팅 경로를 추가한다.

```
- id: product-composite-swagger-ui
  uri: lb://product-composite
  predicates:
  - Path=/openapi/**
```



