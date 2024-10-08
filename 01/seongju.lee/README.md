# 마이크로서비스 디자인 패턴

## 1. 서비스 검색
<img width="366" alt="image" src="https://github.com/user-attachments/assets/97ca5d05-2b1a-42e6-8e51-d50fcd3fe319">

각 마이크로서비스는 동적 IP 주소를 할당받기 때문에, 각 인스턴스를 관리하고 추적하는 프록시가 하나 있어야 한다.  
또한, 각 인스턴스 할당/해지/라우팅 등을 수월하게 관리할 수 있도록 하기 위해 인스턴스 추적용 컴포넌트가 하나 있어야 한다.  

## 2. 에지 서버(== 리버스 프록시)
<img width="387" alt="image" src="https://github.com/user-attachments/assets/19b5fa6a-fda7-4d0d-97db-afb2ad9a26f6">

각 마이크로서비스 인스턴스는 외부에서 접근하지 못하도록 숨기는 것이 바람직하다. 공개된 마이크로서비스는 악의적인 클라이언트의 요청으로부터 시스템을 보호해야 한다.  
때문에, API Gateway 등과 같이 외부 요청을 허용하는 마이크로서비스를 두어 요청을 라우팅하도록 설계하는 방식이다.  

## 3. 리액티브 마이크로서비스

Blocking 방식의 마이크로서비스 통신은 스레드를 점유한다. 동시 요청이 증가하거나 요청 컴포넌트가 증가하면 가용 스레드가 부족해지는 현상이 발생 가능하다.  
때문에 아래와 같이 수행하는 것을 고려해야 함.  
- Async 모델을 사용해서 스레드 대기를 최소화 해야한다.
- Sync 모델이라면, Non-Blocking을 사용하여 응답을 기다리는 과정에서 스레드를 점유를 최소화 해야한다.

## 4. 구성 중앙화

<img width="380" alt="image" src="https://github.com/user-attachments/assets/f6867919-cc83-4bfc-955c-d777b7b46b19">

각 마이크로서비스마다 동일한 구성정보를 각각 가지고 있는 것은 문제가 몇 가지 있다.  
- 실행 중인 마이크로서비스 인스턴스의 구성 정보를 한 번에 볼 수 없다.
- 구성을 업데이트하고, 관련된 마이크로서비스 인스턴스가 올바르게 업데이트 되게 하려면 너무 복잡하다.

마이크로서비스 집합에 대한 구성 정보를 한 곳에 저장하고 환경별 설정(dev, prod 등)을 지원한다.


## 5. 로그 분석 중앙화

<img width="390" alt="image" src="https://github.com/user-attachments/assets/5adb6b82-94dc-47c5-bd3c-248aa45cec2b">

위 다이어그램처럼 인스턴스 로컬에 로그 파일을 기록하는 상황이라면, 전체 시스템 환경에서 발생하는 사건을 개괄하려면 어떻게 해야할까??  
로그를 중앙화하는 새 컴포넌트를 추가하는 해결책이 있다.  
- 로그 수집기가 마이크로서비스에서 로그 이벤트를 표준 로그 형식으로 변환한다. 즉, 구조적이고 검색 가능한 형식으로 중앙 데이터베이스에 저장한다.
- 표준 로그 형식으로 변환하는 이유는 수집한 로그 이벤트를 질의하고 분석하기 편하게 하기 위함이다.


## 6. 분산 추적

외부 요청을 처리하는 동안 각 마이크로서비스간 메시지 흐름을 추적할 수 있어야 한다. 아래와 같이 장애 시나리오가 있을 때 필요할 것이다.  
- 최종 사용자가 특정 장애에 대한 해결을 요청하였을 때, 문제를 일으킨 마이크로서비스를 추적하기 위해서는?
- 특정 엔티티와 관련된 문제를 지원하고자 이와 관련된 로그 메시지를 찾고 싶을 땐? (예. 특정 주문 번호에 대한 문제가 발생했을 때, 주문 처리에 관여한 모든 로그를 추적하고 싶다.)
- 응답 latency를 분석하는 과정에서 어떤 마이크로서비스가 지연을 일으키는지 식별하고 싶다면?

모든 요청 및 메시지에 상관 ID를 넣어 마이크로서비스 사이의 처리과정을 추적할 수 있다. 즉, 중앙화된 로깅서비스에 이 상관 ID를 검색하면 특정 요청에 대한 처리과정을 추적할 수 있다.
이 상관 ID는 요청/응답/메시지가 마이크로서비스 인스턴스로 들어가거나 나갈 때마다 추적 레코드를 생성해야 한다. 

## 7. 서킷 브레이커

특정 마이크로서비스가 응답하지 않으면, 이 마이크로서비스의 클라이언트도 요청에 응답하지 않게 된다. 즉, 장애가 전파되는 것이다.  
특히, Blocking I/O를 사용해 동기식 요청을 하는 경우에 자주 발생한다. 예를 들어, 다수의 요청이 동시에 들어오면 스레드 풀이 빠르게 소진되어 호출이 지연되거나 중단된다.

때문에, 대상 서비스에 문제가 있다는 것을 감지해 새 요청을 보내지 않도록 차단하는 것이 **서킷 브레이커**다.

<img width="600" alt="image" src="https://github.com/user-attachments/assets/dd0a5df6-5380-496e-8d05-3b0aa3e9e066">

- 서비스에 문제가 감지되면 timeout을 무시하고 바로 실패하도록 서킷을 연다.
- 반열림 서킷(장애 복구용 프로브)을 사용한다. 즉, 서비스가 정상 동작하는지 확인하고자 주기적으로 요청을 보낸다.
- 프로브는 서비스의 정상 동작을 감지하면 서킷을 닫는다. 이런 기능은 시스템 환경을 탄력적으로 만들어서 자가 치유 능력을 가능하게 한다.

위 예시에서는 마이크로서비스 E이외의 모든 서킷이 닫혀있고, 모든 트래픽을 허용하고 있는 모습이다. 열려 있는 서킷 브레이커는 **빠른 실패 로직을 사용**한다.


## 8. 모니터링 및 경고 중앙화

응답 시간이나 자원 사용량이 지나치게 높은 경우, 마이크로서비스별 하드웨어 자원 사용량 등을 분석하여 해결할 수 있어야 한다.  

때문에, 각 마이크로서비스 인스턴스의 자원 사용량에 대한 메트릭을 수집하는 모니터링 서비스가 구축되어야 한다.  
- 오토스케일링된 서버를 포함해 시스템 환경에서 사용하는 모든 서버의 메트릭을 수집한다.
- 서버에서 새로 시작된 마이크로서비스 인스턴스를 감지해 메트릭을 수집해야 한다.
- 수집한 메트릭을 조회/분석하기 위한 API와 그래픽 도구를 지원한다.
- 특정 메트릭이 임계값을 초과하면, 트리거되는 경고를 정의할 수 있어야 한다.

