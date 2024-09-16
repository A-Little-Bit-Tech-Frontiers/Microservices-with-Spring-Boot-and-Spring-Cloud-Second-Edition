# 2장 스프링 부트 소개

### Spring Webflux

Project Reactor 를 기본 구현으로 사용하여 리액티브 프로그래밍을 지원하며, Non-Blocking I/O 클라이언트 개발을 지원한다.

**특징**

- 스프링 MVC 와 유사하면서도 리액티브 서비스 지원
- Functional-oriented 모델 기반 라우터 및 핸들링 방식
- 기본적으로 HttpClient 를 WebClient 로, 웹 서버를 Netty 로 사용

### Spring Data JPA

**Repository**

- 리액티브를 사용하는 프로젝트에서 리액티브 데이터베이스 드라이버를 사용하려면, Spring Data 하위 프로젝트에서 사용할 수 있다.
    - RDB - **R2DBC**, Hibernate Reactive
    - Others - Cassandra, MongoDB, Redis 는 자체적으로 Reactive 를 지원

### Spring Cloud Stream

- 스프링 클라우드 에서 pub/sub 패턴을 지원하는 메시지 브로커를 통합하기 위한 프로젝트다.
- 기본적으로 Apache Kafka, RabbitMQ 를 지원한다.
