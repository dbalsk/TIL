# **DTO-엔티티**
>[상품관리애플리케이션](https://github.com/dbalsk/TIL/blob/main/SpringBoot/%EC%83%81%ED%92%88%EA%B4%80%EB%A6%AC%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98.md) 실습을 통해 DTO, 도메인객체(엔티티/값객체)에 대해 학습해보자.
  
## DTO (Data Transfer Object)
**데이터를 전송하는 객체**로 계층 간 데이터 교환에 사용된다. 비즈니스 로직을 담은 도메인 객체와는 구분되며 외부로 노출되는 데이터 구조와 내부 데이터 구조를 **분리**하기 위해 사용된다. 

>+DTO 사용이유  
DTO가 없다면 도메인 객체가 클라이언트에게 노출됨. 위와 같은 경우, 도메인 객체의 변경이 클라이언트에게 바로 영향을 주지만 DTO가 있다면 변환 로직을 통해 클라이언트에게 영향을 미치지 않을 수 있음. 또한 목적별로 여러개의 DTO를 서로 다른 API에 대해 서로 다른 DTO를 제공할 수도 있음.   
         
## 도메인 객체
애플리케이션에서 사용되는 **데이터**와 **핵심 지식(비즈니스 로직)** 을 담은 객체이다. 엔티티(Entity)와 값 객체(Value Object)로 구분된다.

**✔ 엔티티(Entity)**  
도메인 객체이면서 id를 가지는 존재이다. 

**✔ 값 객체(Value Object)**  
도메인 객체이면서 id를 가지지 않는 존재이다. 

## DTO-엔티티 변환
- [DTO-엔티티 변환 위치(컨트롤러 vs 서비스)](https://github.com/HomoEfficio/dev-tips/blob/master/DTO-DomainObject-Converter.md)  
-> **서비스 계층**에서 처리하는 것이 타당. 

- DTO-엔티티 변환 방법  
    1. 생성자
    2. 정적 팩토리 메서드
    3. **ModelMapper 매핑 라이브러리** 

</br>

## 참고자료  
[이것이백엔드개발이다](https://product.kyobobook.co.kr/detail/S000211834105)
