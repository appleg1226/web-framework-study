# 2. Resource
일단 Resource가 무엇인가 알아보아야 할 것 같다.
간단하게는 스프링 패키지를 타고 내려가다 보면 resources 라는 폴더가 있다. 
거기에 있는 파일들이 간단하게 Resource들이다.

원래대로라면 자바의 java.net.URL 클래스를 이용해서 Resource를 사용할 수가 있다고 한다. 그러나 이 단순한 클래스로는 다양한 리소스에 대한 동적인 처리가 어렵다. 

특히나 파일들을 가져올 때, 클래스패스를 기준으로 가져올 것인지, 아니면 ServletContext를 위주로 가져올 것인지 이런 다양한 상황에 대해서 대처가 불가능하다.
 
그냥 자바의 기본 라이브러리는 굉장히 단순하고 기능이 적다는 말이다.

그래서 스프링에서는 서버의 자원들을 어떻게 접근하고 관리할 것인가에 대해서 하나의 기능을 만들었다. 스프링 전반적으로 Resource를 관리하는 것을 이 기능을 중심으로 구현을 해놓았다는 것이다.

구성요소는 다음과 같다

1. Resource: 
실제로 다뤄질 데이터들. 리소스의 특징에 따라 UrlResource, ClassPathResource, ServletContextResource 등 다양한 구현체가 있다.

2. ResourceLoader: 
말 그대로 자원을 가져오는 클래스들이다.
ResourceLoader 인터페이스를 구현하면 Resource들을 얻을 자격을 얻는다. 
그 안에 있는 getResource 메서드를 사용하면 하나의 Resource를 얻는다.
모든 ApplicationContext 구현체들은 모두 이 인터페이스를 구현하기 때문에, Resource들을 가져오는 기능을 가지고 있다.

3. ResourcePatternResolver: 
ResourceLoader의 확장판. 
실제로 ResourceLoader를 상속한 형태이다.
이 인터페이스를 구현하면 패턴에 맞는 Resource들을 전부 가져올 수 있다.  
모든 ApplicationContext에 있는 ResourceLoader 구현체들은 사실 이 인터페이스를 구현하고 있다.

4. ResourceLoaderAware
만약 다른 클래스가 context 내에 있는 ResourceLoader를 사용하고 싶다고 해보자.
그러면 ResourceLoaderAware를 구현하면 된다.
그러면 ResourceLoader를 아는(Aware) 상태가 된다.
단 Context에 Bean으로 등록되어야 하기에, Component로 등록되어야 한다.
그러면 Context가 본인을 이 ResourceLoaderAware를 구현한 클래스에 넣어준다.
Context 자체가 애초에 ResourceLoader를 구현한 구현체이기 때문이다.

<br/>

## 이 기능을 통해서 배운 점
Resource 구현을 보면서 놀란 점은 이것이다.

파일을 읽어오는 건 그냥 라이브러리처럼 쓰면 되잖아... 라는 생각이 일반적일 것이다.  
나였다면, Resource를 가져와야하는 방식별로 인터페이스 및 클래스를 만들고(파일, url, s3 등)
기껏해봐야 하나의 인터페이스를 중심으로 구현을 했을 것이다.

그러나 스프링에서는 Resource를 사용하는 클래스를 ResourceLoader로 지정해줘야하고,  
이에 또 확장 기능을 하는 ResourcePatternResolver를 만들고,  
이것을 다른 클래스에서 사용하도록 ResourceLoaderAware라는 클래스를 만들었다.

도대체 이 간단한 기능이 왜 이렇게 많아지는지 의문이 안 드는 건 아니다.  
다만 객체지향 프로그래밍을 극한으로 끌어올린다면 이 정도까지도 할 수가 있구나 라는 생각이 든다.

```
파일을 다른 이름으로 => Resource
파일을 가져오는 애 => ResourceLoader
그것을 전략적으로 더 세분화 시킨 애 => ResourcePatternResolver
파일을 가져오는 애를 데려오는 애 => ResourceLoaderAware
```

이렇게 여러 개로 나눠져 있는 이유는 단순하다.  
책임을 최대한 조각내기 위해서이다.  
스프링은 그 어떤 책임도 한 클래스에서 담당하지 않겠다는 의지를 자주 보여주는 것 같다.

이후에 개발을 하면서 이런 책임을 나누는 방식에 대해서는 가끔 시도해봐야겠다.

참고로 스프링에서는 이 Resource와 관련된 기능들을 굳이 스프링을 쓰지 않는 순수 자바 프로그래밍에서도 라이브러리처럼 사용이 가능하도록 개발을 해놓았다고 한다.   
개발자들의 공익적인 모습과, 잘 만든 기능은 남들과 같이 쓰고 싶어하는 마음이 반영된 기능인 듯 하다.

<br/>

# 3. Validation, Data Binding, and Type Conversion

스프링을 사용하면서 아무렇지 않게 사용했지만,   
사실상 정교한 구현이 이루어진 부분에는 이 부분도 포함된다.

### ex1. PathVariable에 들어있는 id를 Long으로 변환해주는 변환(Converting) 기능 
```
@GetMapping("something/{id}")
public void getSomething(@PathVariable("id") Long id){
    ...
}
```

### ex2. 필드를 검증(Validation)해주는 어노테이션 기능
```
public class Person {
    @NotNull
    private String name;
    @NotNull
    private Int age;
}
```

## ex3. 필드를 자동으로 Formatting 해주는 기능
```
public class Person {
    ...
    @DateTimeFormat(iso=ISO.DATE)
    private LocalDateTime birth;
}
```

위의 기능들을 우리는 아무런 실제 구현 없이 단순히 선언해서 사용했지만,  
이런 것들이 내부적으로 많은 클래스들의 상호작용을 통해서 구현이 되어 있다. 

## 1. Validation by Using Spring’s Validator Interface  
이전에는 Validation을 수동으로 진행했다.
클래스를 Validation하기 위해서 Validator Interface를 상속한 나만의 Validator를 만든 다음,
그것을 수동으로 사용하곤 했다.

## 2. Resolving Codes to Error Messages  
위의 과정에서 연결되는건데, Validator를 만들 때 error를 어떻게 처리하는지에 대한 내용이다.
DefaultMessageCodesResolver가 관여되는 것 같다.

## 3. Bean Manipulation and the BeanWrapper  
BeanWrapper는 스프링의 Bean이 아니라 예전에 구현된 자바의 스펙인 것 같다.  
여튼 이 클래스를 이용하면 내부의 Property를 조작할 수 있게 된다.  
다만 딱히 메인으로 사용되는 기능은 아닌 듯.  
핵심은 다음에 나오는 PropertyEditor이다.   
이 객체의 주 기능은 string과 Object 사이의 변환을 하는 것이다.  
그리고 스프링은 이 로직을 이용하여 필요한 상황에 클래스를 적절하게 변환한다.  
예를 들면 위의 1번 예제처럼, String을 적절하게 Long으로 변환하도록 해주는 것도 PropertyEditor의 역할이다.

## 4. Spring Type Conversion & 5. Spring Field Formatting  
그러나 PropertyEditor는 Thread Safety하지 않다는 단점이 있다.  
내부적으로 상태를 저장하기 때문이다.  
그래서 스프링 3 이후로 그 대용으로 ConversionService API가 나왔으며,  
그 내부적으로 Converter<S, T>와 Formatter<T>을 사용한다.  
이 ConversionService API는 내부에 Converter들과 Formatter들을 가지고 있다.

Converter는 위의 PropertyEditor가 사용하던 기능을 한다.  
즉 어떤 클래스 S와 T를 변환시켜주는 기능을 한다.  
Formatter도 비슷한 기능을 하지만, String과 Object 사이의 변환만 담당한다.

사용법은 이 링크에서 확인하면 될 것 같다. 백기선님의 강의에서 좀 자세하게 다룬 듯 하다.

https://atoz-develop.tistory.com/entry/Spring-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%B0%94%EC%9D[…]tyEditor-Converter-%EA%B7%B8%EB%A6%AC%EA%B3%A0-Formatter

결론적으로 ConversionService가 이 기능의 가장 위에 있는 기능이라고 할 수 있을 것 같다.  
이 객체에서 Conversion에 대한 대부분의 로직을 가지고 있고, 이것을 사용하면 된다고 볼 수 있겠다.

참고로 스프링부트에서는 이 ConversionService를 상속한 WebConversionService를 가지고 관리를 한다.  
이게 ApplicationContext에 저장되어 있는 듯 하다.

## 6. Configuring a Global Date and Time Format
@DateTimeFormat에 대한 커스터마이징에 대한 내용

## 7. Configuring a Bean Validation Provider
여기에서는 Custom Constraint를 적용하는 것만 확인하면 될 것 같다.  
@Constraint를 사용한 어노테이션을 하나 만든 다음,  
ConstraintValidator를 구현한 Validator 클래스를 하나 만들어주면 된다.

스프링 내부 구현 과정은 다음과 같다.  
LocalValidatorFactoryBean 에서 SpringConstraintValidatorFactory을 구성한다.   
이 Factory는 스프링을 이용하여 ConstraintValidation들을 만드는 역할을 한다.   
아마도 등록된 Custom Validator Bean들을 가져오는 역할을 하는 것 같다.

처음에는 대충 읽으려고 했지만, 막상 읽어보니 좀 다른 느낌이 들기는 한다.  
우리가 편리하게 사용하려고 했던 Validation, Conversion 두 가지에 대한 추상화도 
뒤를 보면 다양한 과정들이 개입하고 있었다.  
DI에 이어서, Resource도 그렇고 이번 3장도 왜 3번째에 존재하는 지 알 것 같다.

사용할만한 프레임워크를 만들려면 참 들어가야할 기능들이 많다는 것을 느낀다.

# 5. Aspect Oriented Programming with Spring
문서에도 나와있듯이 AOP가 OOP와 두 글자나 비슷해서 대체제가 아니냐라는 말이 많다.  
물론 그 말은 AOP의 코드를 몇 줄 보면 바로 들어가긴 할 거다.

AOP라는 것은 어떤 패러다임이다.  
많은 글들에 공통의 관심사라는 단어를 중심으로 한 설명들이 많이 있는데   
맞지만 뭔가 딱 이해가 되는 설명은 아니다.

내가 원하는 메서드들을 실행할 때 전이나 후에, 또는 동시에 어떤 공통된 행동을 넣어주겠다는 것이다.  
가장 쉬운 예시는 stopwatch를 이용하여 메소드의 시작 이전과 반환 이후에 시간을 체크해 줄 수 있다.  
AOP의 문법을 간단한 하나의 예시로 보면 다음과 같을 것이다.

```
@Aspect -> (1) 
@Component  -> (2)
public class AopAspect{
    @Around("com.my.somethingsomething()")  -> (3) 
    public void checkTime(ProceedingJoinPoint pjp) {    -> (4)
        // do something
        Object result = pjp.proceed();   -> (5)
        // do something
    }
}  
```  

(1) 이걸 달아줘야 AOP를 사용 가능하다. 애초에 이름부터 A(Aspect )OP 이다.  
(2) Bean으로 등록해줘야 한다.  
(3) 괄호 안에는 어떤 메서드/이름을 대상으로 할지에 대한 필터가 들어간다.   
    어노테이션 옆에는 함수 전/후/둘다 등의 삽입 위치가 들어간다.  
(4) pjp라는 것은 AOP를 적용할 대상 메서드를 실행시킨다.  
(5) 위에도 나왔듯이 메서드가 실행된다. 이 전후로 필요한 일을 할 수도 있고, 심지어 리턴값도 받을 수 있다.

## 1. AOP Concepts
이 장에서는 몇몇 용어들을 정의한다.
나만의 방식으로 풀어보았다.
- Aspect: 공통의 목적을 가진 메서드들로 구성된 클래스. 내부에 AOP 메서드들이 구성된다.
- PointCut: 패턴 매칭등을 통해서 대상들을 찾아내고, 어떤 방식(전후)을 사용할지를 결정하는 방법
- Join Point: 대상 메서드가 실행되는 위치이자 점. 위의 예시의 pjp가 이에 해당
- Advice: 대상 메서드에 적용시킬 동작들. 로깅이나 트랜잭션 같은 공통의 행위들

## 2. Spring AOP Capabilities and Goals
스프링 AOP는 AspectJ 같은 전문 AOP 프레임워크와는 목표가 다르다고 한다.  
스프링의 방식은 method execution 이라는 방식만 지원을 하고 있으며,  
일반적인 사용에만 초점을 맞추고 있기 때문에, 추가 기능을 지원하고 있지 않다.

그런 기능을 사용하려면 AspectJ 같은 것들을 사용하라고 말한다.  
스프링에서는 IoC Container를 보조하는 역할로 많이 사용하고 있으며,  
비즈니스 로직을 더욱 편하게 사용할 수 있는 방법만 제공하고 있다. 

## 3. AOP Proxies
AOP의 핵심은 프록시 패턴을 사용하는 것이다.  
대상 메서드를 Proxy 클래스로 감싸고, 그것을 가지고 스프링 AOP가 실행한다.  
다만 이런 프록시를 생성하는 방식은 스프링 내부적으로도 두 가지가 있다.  
JDK Dynamic Proxy를 사용하는 방법과 CGLIB을 사용하는 방법으로 나뉜다.

## 4. @AspectJ Support
4장은 AOP에 대한 설명서이다. 다양한 문법에 대해서 정리하고 있기 때문에 여기에선 정리하지 않는다.  
Retry에 대한 AOP 예제가 있는데 이건 좀 나중에도 참고할 만한 것 같다.

## 6. Chossing which AOP Declaration Style to Use
스프링과 AspectJ를 비교하고 어떤 것을 선택해야 하는지를 제시해준다.  
스프링에서는 Compiler/Weaver를 사용하지 않기 때문에 이런 기능을 원한다면 AspectJ를 사용해야 한다.

## 8. Proxying Mechanisms
JDK의 Proxy 방법과 CGLIB은 그 방식이 다르며, 대상도 다르다.  
JDK는 대상 클래스가 특정 interface를 구현해야 proxy가 가능하지만,  
CGLIB은 단순한 클래스에도 proxy를 사용할 수 있다는 특징이 있어,  
Spring에서는 상황마다 다르게 사용하는 것 같다.  
참고로 성능은 CGLIB이 더 좋다고 한다.

### 참고 링크)
1) https://do-study.tistory.com/83
2) https://velog.io/@ha0kim/2020-12-28-AOP-%ED%8A%B9%EC%A7%95-%EB%B0%8F-%EB%8F%99%EC%9E%91%EC%9B%90%EB%A6%AC
3) https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html








