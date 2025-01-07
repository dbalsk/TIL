# **HTTP**
>HTTP 트랜잭션, HTTP 요청 메소드, HTTP 응답 상태코드를 통해 HTTP에 대해 이해해보자.
  
## HTTP 트랜잭션
하나의 HTTP 요청과 응답 묶음
    
**✔ 요청-응답 과정**   
1. 클라이언트가 HTTP 요청
2. 서버에서 HTTP 요청 처리 ex) 데이터베이스 처리 
3. 서버가 HTTP 응답 

**✔ HTTP 트랜잭션 살펴보기** 
> 개발자 도구를 통해 HTTP 트랜잭션의 상세 내용을 볼 수 있는 탭을 살펴보자.

- **Headers**: **헤더** 정보 (요청/응답헤더 나눠서 볼 수 있음.)
- **Payload**: 데이터 전송에 포함되는 내용의 데이터 (요청의 바디 데이터 없으면 볼 수 없음.) 
- **Preview**: 응답의 **바디**에 포함된 데이터 **보기 좋은 형태**로 미리보기
- **Response**:응답의 **바디**에 포함된 데이터 **있는 그대로** 보기
- Initiator: HTTP 트랜잭션이 시작되는 주체와 종류
- Timing: HTTP 트랜잭션 단계별 정보
- Cookies: 요청과 응답에 포함된 쿠키

## HTTP (hyperText Transfer Protocol)
웹페이지들 사이에서 링크를 타고다니며 정보를 전송하는 통신규약(Protocol)

**✔ HTTP 특징**
> 현재 HTTP/1.1 버전이 주로 사용되고 있다. 각 버전을 살펴보며 HTTP의 특징을 이해해보자. 
- HTTP/1.1에 기반한 HTTP 특징
    1. **클라이언트의 요청**으로 HTTP 트랜잭션 시작
    2. **무상태성**: 이전 HTTP 트랜잭션과 다음 HTTP 트랜잭션 사이에 연관관계가 없음. 즉, 서버는 클라이언트를 **구분할 수 없음**.(같은 클라이언트라도)
    3. **비연결성**: 데이터를 주고받기 위해 연결을 하고 HTTP 트랜잭션이 **종료되면 연결을 끊어버림**. 
        - 장점: 자원 효율적 사용
        - 단점: 매번 HTTP 트랜잭션마다 연결 맺고 끊는 과정 추가 -> **오버헤드** 발생
    4. **사람이 볼 수 있는** 형태로 헤더 주고받음. (바디는 압축하여 주고받음)
    > +오버헤드 Overhead   
    > 어떤 작업을 위해 필연적으로 수반되는 부수적인 작업 (HTTP/1.1에서는 커넥션에 대한 지속적인 연결을 통해 오버헤드를 줄임)
- HTTP/2 특징
    1. **다중화**: 요청한 순서에 상관없이 응답이 오는 대로 처리 가능 (HTTP/1.1에서는 요청 순서에 맞게 응답이 와야하는 제약이 있음) 
    2. **헤더 압축(HPACK)**: 헤더를 압축하여 성능 향상 (HTTP/1.1에서는 헤더의 사이즈가 크고 파싱에도 비효율적) 
    3. **서버 푸쉬**: 서버에서 클라이언트에게 예상되는 요청을 미리 보냄. 
- HTTP/3 특징
    1. **QUIC**(UDP를 사용하는 프로토콜) 사용: 기존의 TCP 헤더에 비해 가벼운 **UDP 헤더**(출발지/목적지 포트, 데이터 길이, 체크섬정도만 포함)를 사용하고 **필요한 기능은 추가 구현**하여 효율적/빠른 HTTP 통신  
    >+TCP vs UDP   
    >**TCP**는 연결을 지향하고 손실을 네트워크 레이어에서 감지하는 프로토콜 -> 안정성 높음 / 많은 시간 소모 (3웨이 핸드쉐이크 있어야만 통신 가능)   
    >**UDP**는 연결을 맺지 않고 손실된 데이터가 있어도 재전송하지 않는 프로토콜 -> 가볍고 빠름 / 필요한 기능은 애플리케이션 레이어에서 구현 가능.


## HTTP 요청 메소드
HTTP 요청 메소드에 따라 다른 동작을 한다. 즉, 행위에 맞게 적적할 메소드 선택이 필요하다.

**✔ 주요 HTTP 요청 메소드**   
- **GET**: 특정 자원에 대한 **조회** 요청 
- HEAD: 특정 자원에 대한 조회 요청 (바디를 제외한 **헤더만** 응답으로)
- **POST**: 새로운 자원 **생성** 요청 (요청 바디에 있는 내용을 바탕으로 생성)
- **PUT**: 기존 자원을 요청 바디에 있는 내용으로 **변경**
- PATCH: 기존 자원을 변경 (자원 전체가 아닌 **일부만** 변경)
- **DELETE**: 특정 자원 **제거**
- OPTIONS: 해당 경로에서 **어떤 HTTP 요청 메소드** 사용 가능한지 확인

**✔ HTTP 메소드의 주요 특징**   
- **안전한** 메소드: 대상이 되는 **자원의 상태를 변경하지 않는** 메소드.    
-> **GET** / HEAD / OPTIONS 
    > +GET 메소드   
GET 메소드는 특정 URL에 접속하기만 해도 요청이 날아갈 수 있기에 자원의 상태를 변경하는 API에 사용하는 것은 위험한 API 설계. 즉 **GET은 반드시 안전한 메소드**여야함.

- **멱등성** 있는 메소드: **한 번 호출**과 **여러 번 호출**한 것이 **같은 자원의 상태**를 가지는 메소드   
-> GET / PUT / DELETE

즉, 해당 기능의 **안전성/멱등성**을 보고 **적절한 HTTP 요청 메소드**를 사용해야함. 

**✔ CRUD에 적절한 HTTP 메소드** 
- Create(생성하기) -> **POST** (멱등성이 없으므로 POST가 적절)
- Read(읽기) -> **GET** (안전해야하므로 GET이 적절)
- Update(수정하기) -> **PUT** (보통 멱등성이 있고 수정한다는 의미에서도 PUT이 적절)
- Delete(삭제하기) -> **DELETE** (보통 멱등성이 있고 삭제한다는 의미에서도 DELETE가 적절)

## HTTP 응답 상태코드
클라이언트가 보낸 HTTP 요청이 서버에서 **어떻게 처리**되었는지에 대한 정보를 제공한다. 100번 대에서 500번 대까지의 값을 사용한다.

**✔ 상태코드의 대역별 의미**
- 1xx: 정보성 상태코드 (요청을 받았으나 계속 진행)   
- 2xx: **성공**을 의미하는 상태코드 (요청을 성공적으로 수신/이해/수락)
    - 200: 요청 성공
- 3xx: **리다이렉트**를 의미하는 상태코드 (요청을 서버에서 처리하지 않고 다른 곳으로 유도. 2번의 HTTP 트랜잭션 발생.)
    - 301: 리다이렉트하여 다른 경로로 페이지 이동 
- 4xx: **클라이언트 오류**에 해당하는 상태코드 (요청에 잘못된 구문 있거나 수행할 수 없음.)
    - 404: 요청 실패 ex) 잘못된 경로로 접속
    - 400: 파라메터 누락
- 5xx: **서버 오류**에 해당하는 상태코드 (서버에서 요청을 처리하는 과정에 문제 발생.) 
    - 500: 서버 측 에러 

</br>

## 참고자료
[이것이 백엔드 개발이다](https://product.kyobobook.co.kr/detail/S000211834105)   