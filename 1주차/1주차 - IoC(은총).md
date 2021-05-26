# 1주차 - IoC

스프링의 가장 핵심은 Inversion of Control(IoC) Container이다.
사실상 제일 중요한 기능이며, 문서의 대부분을 이루는 기술이기도 하다.

IoC를 통해서 우리는 대부분의 코드를 선언적으로 사용할 수 있게 된다.
즉 기존에 Foo aFoo = new Foo(); 를 이용해서 절차적으로 만들던 코드를
단지 Foo() aFoo; 라는 선언만으로 사용할 수 있게 되는 것이다.

사실 이 정도만 따지면 단지 클래스의 선언을 도와주는 프레임워크라고 생각할 수도 있다.

그러나 스프링 IoC의 핵심은 이를 통해서 거의 대부분의 스프링 생태계를 이어준다는 데에 있다.
정말 다양한 모듈들이 다 이 IoC Container를 기반으로 하고 있고 그에 맞춰 구현되어 있기 때문에, 
사용자는 대부분의 기능을 일관된 형식으로 사용할 수 있다. 

그리고 내가 생각하는 장점이 하나 더 있는데, 굉장히 추상화가 되기 때문에, 코드가 깔끔해진다는 장점도 큰 것 같다. 

## 1. Introduction to the Spring IoC Container and Beans
IoC의 중심 패키지는 beans, context 패키지 두 개이다.
beans 패키지에서는 BeanFactory 인터페이스가,
context 패키지에서는 ApplicationContext가 중심을 이룬다.

왜 이렇게 나눠져 있나고 생각 할 수 있는데, 
스프링에서는 사용성을 위해 프레임워크의 설계를 이렇게 두 계층으로 나눈 것이다.
(더 자세한 것은 뒤의 1.16 BeanFactory에 더 자세하게 나온다.)

만약에 스프링부트를 통해서 프로젝트를 실행하지 않고, 직접 스프링을 시작하기를 원한다면
이 두 가지 중 하나를 선택해서 시작해야 한다.

현재 시점에서 보았을 때, 현재 스프링 프로젝트는 거의 SpringBoot를 이용해서 작성한다.
그러나 예전에는 ClassPathXmlApplicationContext 라는 클래스를 사용해서
ApplicationContext context = new ClassPathXmlApplicationContext("SpringBeans.xml");
이런 식으로 불러오곤 했다. 그래서 이 문서에도 XML 기반의 예제도 많이 등장한다.

갑자기 궁금해서 내가 처음 공부했던 스프링 프로젝트를 찾아봤는데,
놀라운 것이 내 프로젝트에 main 메서드가 아예 없었다.

알아보니 Tomcat 어플리케이션을 실행시키면 여기에서 자동으로 web.xml을 읽어와서 
스프링 환경을 구성하는 것이었다.
지금까지 내가 구동시킨게 내 프로젝트가 아니라 톰캣이었다니..
이에 대한 간단한 순서 하나를 덧붙인다.

```
1. 웹 애플리케이션이 실행되면 Tomcat(WAS)에 의해 web.xml이 로딩된다. (load-on-startup 으로 톰캣시작시 servlet 생성가능하도록 설정 가능)
2. web.xml에 등록되어 있는 ContextLoaderListener(Java Class)가 생성된다. ContextLoaderListener 클래스는 ServletContextListener 인터페이스를 구현하고 있으며, ApplicationContext를 생성하는 역할을 수행한다.
3. 생성된 ContextLoaderListener는 applicationContext.xml을 로딩한다.
4. applicationContext.xml에 등록되어 있는 설정에 따라 Spring Container가 구동된다. 이때 개발자가 작성한 비즈니스 로직에 대한 부분과 DAO, VO 객체들이 생성된다.
출처: https://12bme.tistory.com/555
```


## 2. Container Overview
ApplicationContext에서는 IoC 컨테이너의 Initiating, Configuring, Assembling the beans
이렇게 세 가지 일을 주로 수행한다.
그러기 위한 설정은 xml, annotation, java config 을 통하여 읽는다.

여기에는 xml 설정이 주로 나와있지만, 지금은 사용하지 않으니 적당히 넘기고,
나중에 annotation, java 설정들을 더 열심히 보았다.

위에서도 말했지만,
스프링을 시작하려면 ApplicationContext를 직접 만드는 과정이 필요하다.
읽어올 설정 방식에 따라 실제 구현된 하위 클래스를 선언하여 어플리케이션을 시작하면 된다. 

이제 그렇게 되면 ApplicationContext에서 복잡한 과정을 거쳐서 Spring 어플리케이션을 실행하게 된다.  

## 3. Bean Overview
이 장에는 Bean의 정의가 나와있는데 생각보다 복잡하고, 관리하는게 많다.
이렇게 많은 필드를 정의해줘야 DI를 완벽하게 사용할 수 있나보다.
그래서 그 필드를 정리해보면

```
- Class: 패키지명까지 포함된 클래스 이름
- Name: Bean의 이름인데, 정의할 때 따로 정의하지 않으면 스프링이 알아서 생성해준다.
- Scope: Singleton인지, 호출될 때마다 생성되는 Prototype인지 등의 정보를 담는다.
- Constructor Arguments: 해당 빈의 내부에 생성자 인자가 있다면 해당 클래스들의 정보를 저장한다.
- Properties: 해당 빈의 내부에 Properties가 있다면 해당 클래스들의 정보를 저장한다.
- Autowiring mode: @Autowired와는 상관이 없고 웬만하면 건드릴 필요가 없는 옵션이다. Bean의 내부에 주입이 필요한 의존성이 있을 경우에 그것에 대한 설정을 지정해줄 수 있다. 원래대로라면 @Autowired 같은 것으로 표시를 해줘야 하는데, 이 모드의 기본값을 건드릴 경우에는 @Autowired 없이도 주입을 해줄 수가 있다. 그러나 추천하지 않는 방법이다. 
- Lazy Initialization mode: 스프링의 빈은 대부분 시작에 초기화되지만, 그것을 사용될 때로 늦출 수가 있다.
- Initialization method: 빈이 생성될 때 빈 내의 특정 메서드를 실행시킬 수 있는 옵션
- Destruction method: 빈이 제거될 때 빈 내의 특정 메서드를 실행시킬 수 있는 옵션
```


빈은 보통 세 가지 방법으로 초기화가 된다.
1. Constructor
2. Static Factory Method
3. Instance Factory Method

그러나 웬만하면 Constructor를 통해서 초기화를 하게 될 것이다.


## 4. Dependencies
- 생성자 주입 vs setter 주입  
Spring 팀의 추천은 생성자 주입을 하는 것이다.
이는 객체를 불변으로 만들 수도 있고, 의존성의 null도 방지한다. 
또한 항상 초기화된 객체를 반환한다.
하지만 Setter를 사용하는 예는 JMX MBean을 이용해서 추가적으로 의존성을 주입하는 경우가 있겠다.
이외에는 그냥 생성자를 사용하면 될 것 같다.

컨테이너가 Bean을 초기화 하는 과정은 다음과 같다.
```
1. ApplicationContext가 만들어지고, XML, Java Config, Annotation 설정들을 읽어와서 각 Bean의 Definition을 채운다.
2. 각 Bean들에 대하여 properties, constructor arguments 등과 관련된 정보가 정리되어 채워진다.
3. 정의된 내용들을 바탕으로 실제 인스턴스들이 주입된다. 
```

이외에도 Bean의 사용에 대한 추가적인 정보
- 콜렉션을 빈으로 만들 수도 있다.
- @DependsOn 을 사용하면 특정 빈이 먼저 초기화하도록 할 수 있다.
- @Lazy 를 사용하면 Bean이 사용될 때 초기화 시킬 수 있다. 

## 5. Bean Scopes
Bean을 스프링에서 사용할 때 매번 만들 것인지, 아니면 만들어진 것을 계속 반환할지 선택할 수 있다.
```
- Singleton: 한 번만 초기화 시킨다.
- Prototype: 빈 생성이 요청될 때마다 새로운 인스턴스를 받는다.
- Request, Session, Application, WebSocket: Spring Web과 관련된 Scope로서 특정 상황마다 빈을 새로 생성한다. 
```

만약에 Singleton Bean 내부에 Request Bean이 주입된다면 어떤 일이 발생할까?
이렇게 되면 사실상 Request Bean은 한 번 초기화되고 새로 생기지 않게 되는 현상이 발생한다.
그럴 때는 \<aop:scoped-proxy/>을 선언해주면 내부 Request Bean의 프록시가 Singleton에 주입되고,
프록시 내부적으로 Request Scope를 유지시켜 준다.

이외에도 스프링에서는 나만의 커스텀 Scope를 만들 수도 있다.

## 6. Customizing the Nature of a Bean
이 장에서는 Bean을 좀 더 심화적으로 다루는 방법들을 다루고 있다.

먼저 Bean의 생명주기에 특정 메서드를 실행시킬 수 있다.
@PostConstruct와 @PreDestroy 를 메서드에 달아주면 사용할 수 있다.
이건 가끔 사용되는 것 같다.

또한 스프링을 직접 사용하는 유저의 입장이 아닌, 
Spring bean을 직접 접근해야 하는 스프링 내부 확장에 적합한 것들도 설명이 되어있다.
*ApplicationContextAware* 라는 인터페이스를 구현하면 스프링 빈이 이를 통해서 ApplicationContext를 조작할 수 있다고 하는데, 사실 좀 복잡하고 사용할 일이 없을 것 같긴 하다.

## 7. Bean Definition Inheritance
Bean은 부모를 Abstract 클래스로 두고 템플릿처럼 사용할 수도 있다.
이 때는 Spring의 관련 문법을 따르는데 XML에만 정의되어 있고 Annotation에는 정의되어 있지 않다.
이 방법은 아마도 사용할 일은 없을 것 같다.
이미 Bean의 상속은 다른 방법으로 구현이 가능하다.

## 8. Container Extension Points
Spring Bean을 활용하는 심화 방법에 대한 내용이다.
스프링에서는 Bean을 생성하는 과정에 PostProcessor를 껴넣을 수가 있다.
특정 빈이 생성될 때마다, 특정 처리를 할 수도 있고, Metadata를 변경시킬 수도 있다.
이 기능은 스프링을 이용한 커스텀 기능을 만들 때 좋을 것 같다.

## 9. Annotation-based Container Configuration
이 장에서는 드디어 어노테이션 기반의 스프링 설정들을 다룬다.
기본 사용법 보다는 특별히 몰랐던 점들을 위주로 정리해보려고 한다.

1) @Autowired는 AutowiredAnnotationBeanPostProcessor에서 다뤄진다.
2) @Autowired는 컬렉션에 사용되면 해당 타입의 모든 빈을 주입해준다.
3) @Autowired(required = false) 또는
setter에 지정된 @Autowired 파라미터에 Optional<T>이나 @Nullable 옵션을 주면
해당 빈이 주입되지 않아도 예외를 발생시키지 않는다. 
4) BeanFactory, ApplicationContext, Environment, ResourceLoader, ApplicationEventPublisher, and MessageSource 같은 것들을 @Autowired로 바로 가져올 수 있다.
5) @Primary, @Qualifier 를 이용해서 같은 타입인 빈이 많을 경우에 주입될 빈을 더 미세 조정할 수 있다.
6) @Qualifier 어노테이션을 이용하여 새로운 어노테이션을 만들어 기능을 대신할 수 있다.
7) @Autowired는 내부 Generic도 구분을 해서 가져올 수 있다.
Info<Person>은 @Autowired Info<Person> 에 주입되고, @Autowired Info<Animal> 에는 주입되지 않는다.
8) @Value는 properties에 있는 값들을 주입해주는데, Spring boot에서는 *PropertySourcesPlaceholderConfigurer*가 이 일을 담당한다.


## 10. Classpath Scanning and Managed Components
이 장에서는 가장 핵심이라고 할 수 있는 
Component Scan, Component 등을 다룬다.
클래스를 빈에 등록하기 위해서는 어떤 빈을 가져올 것인지 찾는 과정이 필요한데,
그 과정에 대한 자료가 모여있는 장이다.

우선 어떤 것이 Bean으로 등록되어야 하는지를 확인하려면 그것을 마킹을 해줘야 한다.
그 표시가 바로 @Component이다. 
이 표시가 되어있는 클래스들이 빈으로 등록이 될 것이다.
@Service, @Controller 같은 어노테이션들은 다 이 @Component를 확장시킨 것이다.
문서 내 용어로는 Compose Annotation 이라고 말한다.

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {
    // ...
} 
```
이렇게 어노테이션의 선언 위에 붙여주면 @Service가 @Component와 같은 취급을 받는다.
마치 상속과 비슷한 느낌이다.

Component Scan을 지정해주면 해당 내부 패키지 지정에 따라서 부트스트랩 시 Component들을 스캔한다.
그러나 해당 내용은 너무 복잡하여 문서에는 나오지 않았다.
관련해서 분석된 자료를 아래 링크에서 참고할 수 있다.
https://thswave.github.io/spring/2015/02/02/spring-mvc-annotaion.html

다음은 추가적인 정보
- Component-Scan에는 필터를 지정할 수도 있다.
- 각 Component에도 이름 지정이 가능하다.
- Component에 @Scope 지정도 가능하다.
- Component에 @Qualifier 지정도 가능하다.


## 11. Using JSR 330 Standard Annotations
스프링은 JSR 330 명세도 제공한다.
스프링 이전에 이러한 명세를 사용했던 사람들에게 편리하도록 제공해 주는 기능들이기 때문에,
일반적인 사용자에겐 넘겨도 되는 장이다. 


## 12. Java-based Container Configuration
이 장에서는 이전과는 다르게 XML이 아닌 @Configuration과 @Bean을 사용한 설정 방법을 다룬다.

- @Configuration과 @Component는 둘 다 내부에 @Bean을 선언이 가능하기는 하다. 
그러나 @Configuration 내부의 @Bean들 끼리는 서로 간의 의존성을 주입할 수 있다.
반면, @Component에서는 그게 안 된다. 그래서 버그 방지를 위해서 웬만하면 @Bean은 @Configuration 내부에 선언하는 것이 좋다.
<br/>
- Java-based Configuration 방식에서는 *AnnotationConfigApplicationContext* 이 메인으로 사용된다.(ApplicationContext의 하위 구현체)
<br/>
- 웹을 사용할 때는 *AnnotationConfigWebApplicationContext* 을 같이 사용한다.
<br/>
- Bean끼리 의존성이 있을 때 Configuration에서는 내부적으로 Singleton을 여러개 생성해준다.
  
```
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());  => clientDao() 가 일단 싱글톤으로 생성
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());  => clientDao()가 새로운 싱글톤으로 생성 
        return clientService;
    }

    @Bean  
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

- 각 @Bean은 생명주기 메서드도 호출할 수 있고, @Scope의 지정, Aliasing, Description 등을 제공한다.
 

## 13. Environment Abstraction
스프링에서 굉장히 간편한 것 중에 하나는 설정에 대한 관리이다.
크게 Profile와 Property 두 가지로 나뉜다.

Profile은 내가 원하는 환경 이름을 지정하여 그에 해당하는 DI를 수행할 수 있도록 하는 것이다.
내가 development라는 환경 이름을 사용하고 싶다면
@Profile("deployment")와 @Profile("development") 중에 뒤의 Bean만 초기화가 된다.
이처럼 동적으로 Bean의 초기화가 가능해진다.

스프링 부트에서는 application.properties에 Profile을 담거나, -Dspring.proflies.active 를 이용해서 활성화 시켜주면 된다.
@Profile("default")로 기본 Profile도 지정하도록 하는 기능도 있다.

Property는 @PropertySource 어노테이션을 이용하여 불러올 수가 있다.
```@PropertySource("classpath:/com/myco/app.properties")```
이런 식으로 경로 내의 파일을 불러와서 환경에 등록이 가능하다.


## 14. Registering a LoadTimeWeaver
LoadTimeWeaver는 클래스를 동적으로 조작하는 라이브러리이다.
Spring JPA에 사용되는 기술이라고 하며, 
뒷부분의 AoP 부분에 추가 설명이 나와있으나 해당 문서에도 구체적인 내용은 들어있지 않다.


## 15. Additional Capabilities of the ApplicationContext
핵심적인 기능 이외에도 ApplicationContext가 제공하는 추가 기능을 담고 있다. 일반적인 경우에는 잘 사용되지 않는다.
- MessageSource: internationalization를 지원하는 부분. 
- Custom Event: 스프링 내부적으로 이벤트를 사용할 수 있도록 해준다.
- ContextLoader와 ContextLoaderListener를 이용해서 웹 어플리케이션을 쉽게 시작할 수 있도록 해준다.
자세한 내용은 Spring Web을 공부할 때 알 수 있을 것 같다.


## 16. The BeanFactory
이 장에서는 BeanFactory vs ApplicationContext를 다룬다.
도대체 왜 이 둘은 나뉘어 있는걸까?
나는 BeanFactory가 ApplicationContext와 비슷한 기능을 갖는다면 왜 굳이 만들었을까라는 의문이 들었다.

각 인터페이스의 각 기본 구현체는 다음과 같다. 
BeanFactory의 경우 DefaultListableBeanFactory을 통해 구현이 되어있고,
ApplicationContext는 *AnnotationConfigApplicationContext* 등 다양한 하위 클래스들을 통해 구현하게 된다.

물론 BeanFactory를 사용하여 프로그래밍이 가능은 하지만, 일반적인 스프링을 사용하는 경우에는 사용할 이유가 없다.
다만 스프링의 Bean 기능만을 사용하고자 하거나, 스프링을 중심으로 무엇인가를 개발할 때 사용을 할 수는 있겠다.

그냥 처음부터 ApplicationContext만 구현해 놓으면 얼마나 좋을까라는 생각을 해보긴 했다.
그러나 다양한 계층으로 구현하고 분리되어 있는 데는 이유가 있다.

계층이 나뉘어 있기 때문에 이후에 확장이 용이하다.
ApplicationContext가 필요없을 경우에는 BeanFactory만 확장해서 개발하면 된다.
불필요한 레이어를 사용하기를 원하지 않는다면 필요한 레이어만 사용하면 된다.
이처럼 상속을 사용한다면 이후 구현에 있어서 확장성과 자유도가 굉장히 올라간다. 

이러한 클래스 구조에 대해서는 좀 더 익숙해지고 배울 필요가 있을 것 같다.
