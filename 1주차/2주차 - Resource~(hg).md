# AOP


## aop란 
- OOP 와 유사하지만, 아래가 aspect(관점)이 핵심이다.

```aidl
AOP the unit of modularity is the aspect
``` 
- aspect의 모듈화 
  - 핵심/ 부가 기능을 기준으로 관점으로 나누어서 보고, 그 관점을 기준으로 각각 모듈화하겠다.
  
- 흩어진 관심사 (Crosscutting Concerns)
  
## AOP의 주요 개념 


- Aspect : 흩어진 관심사를 모듈화 한 것. 주로 부가기능을 모듈화함. 주로 @Aspect 선언을 통해 구
- Target : Aspect의 타겟. (Spring AOP는 프록시 기반이기 때문에, 프록시가 대상임 )
- JointPoint : Advice가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능
- Advice : joint point 지점의 aspect 에 의해 취해질 행동 (@Around 등등으로 정의)
- PointCut : JointPoint의 상세한 스펙


- Aspect 종류들
@Before: 어드바이스 타겟 메소드가 호출되기 전에 어드바이스 기능을 수행
@After: 타겟 메소드의 결과에 관계없이(즉 성공, 예외 관계없이) 타겟 메소드가 완료 되면 어드바이스 기능을 수행
@AfterReturning: 타겟 메소드가 성공적으로 결과값을 반환 후에 어드바이스 기능을 수행
@AfterThrowing: 타겟 메소드가 수행 중 예외를 던지게 되면 어드바이스 기능을 수행
@Around: 어드바이스가 타겟 메소드를 감싸서 타겟 메소드 호출전과 후에 어드바이스 기능을 수행


## . Spring AOP Capabilities and Goals
- 자바 기반 구현 (추가 컴파일링 필요 없음) 
- Spring AOP는  method기반 AOP만 지원함
- 완전한 AOP를 의도하지 않고, Spring Container 에 의해 관리되는 Bean들만 관리
- IoC Container의 보조 역할
- 더 강력한걸 원하면 (모든 객체 적용 등 ) Aspect J 사용해라

 
## 5.3. AOP Proxies

- Jdk dynamic vs CGLIB

https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html

- 어찌됐든 Proxy 기반 사용한다.

