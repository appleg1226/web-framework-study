# Spring을 중심으로 한 웹 프레임워크 스터디

## 1. 스터디 내용
- 스프링 프레임워크 공식 문서 공부를 통하여, 웹 프레임워크의 구조 익히기(세부적인 편의 기능보다는 핵심적인 기능 위주로)
- 스프링 프레임워크의 코드를 분석하여 구현 패턴(디자인 패턴), 코딩 컨벤션, 고급 문법, 객체 지향 등을 배우기
- 스프링 이외의 다른 웹 프레임워크들(자바 이외에도)을 참고하며 프레임워크의 다양한 구현 패턴에 대해서 익숙해지기
- 저수준 http 라이브러리/프레임워크에 대한 이해(Netty, Servlet 등)
- 비동기 기술에 대한 공부(Java 비동기, Reactor, Netty의 EventLoop) 
- 위의 학습 내용들을 이용하여 핵심 기능을 위주로 가벼운 웹 프레임워크 개발해보기(구현 세부 스펙은 추후 결정)
<br/>

## 2. 방식
1. 스터디 내용에 대하여 각자 블로그 또는 Github에 스터디 내용 정리
2. 추가적으로, 각자 분량을 정하여 간단하게 내용 공유
<br/>

## 3. 예상 일정
### 1. Spring Core 문서 읽고 정리 (2~3주)
- https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans

### 2. Spring Core 부분 코드 분석(2~3주)
코드를 읽으면서 클래스 디자인, 코딩 스타일, 사용된 문법 정리하기
- org.springframework.beans
- org.springframework.context

### 3. Servlet 등 http 서버 라이브러리/프레임워크 공부(2~3주)
  
### 4. Spring Web MVC 문서 읽고 정리(2~3주)
- https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc

### 5. Spring Web MVC 코드 분석(2~3주)
- org.springframework.web

### 6. Spring Boot 문서 읽고 정리(2~3주)
- https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#legal

### 7. 웹 프레임워크 개발 준비(2~3주)
- 다양한 웹 프레임워크 구현 방식 스터디(Java의 Armeria, Kotlin의 Ktor, Node의 Nest.js, Golang의 gin 등)
- 개발할 프레임워크 아키텍처 설계
- 개발 방식 및 개발 방향 정하기 

### 8. 개발 시작(기간 예상 안 됨)
- 웹 프레임워크 개발

### * 이외에 추가적으로 공부하거나 적용할 만한 내용
  - Spring Data Project(JPA, Mongo 등)
  - Spring WebFlux
  - Reactor 
  - Netty
