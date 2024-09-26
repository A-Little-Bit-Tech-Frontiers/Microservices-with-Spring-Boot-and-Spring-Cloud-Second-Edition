
# 스프링 클라우드 게이트웨이를 에지 서버로 사용


## 시스템 환경에 에지 서버 추가

<img width="462" alt="image" src="https://github.com/user-attachments/assets/0fd0fb79-d7d6-4694-9610-ac379ca25f8d">

외부 클라이언트로부터의 모든 요청을 에지 서버에서 받는다. 에지 서버는 URL 경로를 기반으로 들어오는 요청을 라우팅한다. 
예를 들어, `/product-composite` 로 시작하는 요청은 ProductComposite 마이크로서비스로 라우팅하고, `/eureka`로 시작하는 요청은 유레카 검색 서버로 라우팅한다.


<br>

## 라우팅 규칙

1. 조건자(predicate): 수신되는 HTTP 요청 정보를 바탕으로 라우팅
2. 필터(filter): 요청이나 응답을 수정
3. 대상 URI: 요청을 보낼 대상
4. ID: 라우팅 경로 이름

예시)

```yaml

spring:
  cloud:
    gateway:
      routes:
        - id: user-service-post-users
          uri: lb://USER-SERVICE
          predicates: ## 조건
            - Path=/user/users
            - Method=POST
          filters:
            - RewritePath=/user/(?<segment>.*), /${segment}  ## /user/ 뒤의 모든 경로((?<segment>.*))를 캡처한 후, `/`뒤에 재작성

        - id: user-service-get
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user/**
            - Method=GET
          filters:
            - RewritePath=/user/(?<segment>.*), /$\{segment}
            - AuthorizationFilter
```

api-gateway를 통해 userService를 호출하고자 할 때, 위 라우팅 조건을 거치게 된다.  
1. 조건(`predicates`)에 맞는 경우, 지정한 `uri`로 라우팅된다.
2. 라우팅 될 때는 `filters`에 정의된 필터를 거치게 된다. 여기서는 요청 경로를 재작성 해주는 기능도 추가했다.
3. `uri`는 요청을 보낼 대상을 정의한다.
lb는 스프링 클라우드 게이트웨이가 클라이언트 측 로드밸런서를 사용해 유레카에서 대상을 찾도록 지시하는 것이며, USER-SERVICE는 유레카에 등록된 서비스 이름이다.
