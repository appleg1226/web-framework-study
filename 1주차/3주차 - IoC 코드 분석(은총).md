스프링 Core 문서를 읽으니 스프링이 어떤 기능을 제공하는지는 확실히 알게 되었다.  
그렇지만 이번 스터디의 목표는 IoC 컨테이너의 구현을 확실히 확인하자는 것이었기 때문에,

다음 목표는 코드로 잡고 스프링 github 내용을 다운 받아 IntelliJ를 이용해서 분석하게 되었다.  
  
문서의 흐름과 비슷하게 다음의 순서로 코드를 분석해보았다.

**1) Spring BeanFactory 가 구현된 spring-beans 모듈 정리**  
**2) Spring ApplicationContext 가 구현된 spring-context 모듈 정리**  
**3) 그 중에서 핵심이 되는 클래스들의 내부 구현 확인**

---

## **spring-beans 모듈 구성**

#### **beans 패키지**: Java Beans를 다루기 위한 클래스들이 놓여있다.

```
- factory(*): Spring IoC Container의 핵심이 들어있는 부분. 따로 빼서 정리한다. 
- propertyeditors: spring core 3장에 해당하는 클래스들이 들어있다. 
  다양한 클래스들을 변환하기 위한 editor들이 구현되어 있다.
- support: 이 모듈 내부적으로 사용할 utility 클래스들을 모아놓은 곳이다.
```

#### **factory(\*)**: 위의 factory 패키지 내부 구조를 확대

```
- annotation: autowired 같은 어노테이션들과 beanpostprocessor 가 들어있다.
- config: spi interface와 configuration과 관련된 편의 클래스들이 모여있다.
- groovy: groovy support
- parsing: bean의 parsing과 관련된 부분
- serviceloader: java의 serviceloader를 제공하기 위한 패키지
- support(핵심): factory 패키지를 보조하는 클래스들과 beanfactory의 구현체가 존재한다.
- wiring: bean과 metadata의 wiring을 관리
- xml: xml과 관련된 것들
```

  
**핵심 기능이 구현된 클래스는 다음과 같다.**  
1) AbstractBeanFactory(약 2,100줄)  
2) AbstractAutowireCapableBeanFactory(그 아래, 이것도 약 2,000줄이다.)  
3) DefaultListableBeanFactory(가장 아래, 약 2,100줄)


여기에 Bean의 관리와 관련된 대부분의 로직이 구현되어 있다.  
여기까지가 ApplicaitonContext의 상위 모듈인 BeanFactory와 관련된 구조이다.

---

## **spring-context 모듈 구성**

```
- cache: Spring Cache 와 관련된 기능이 구현되어 있다.
- context(*): Spring ApplicationContext 와 관련된 핵심 기능이 담겨 있다.
- ejb: Java EE의 EJB와 관련된 부분
- format: spring core 문서의 formatter가 구현되어 있는 패키지
- instrument.classloading: Load time weaver와 관련된 기능이 모여있다. 
- jmx: spring의  jmx 기능 구현과 관련된 패키지
- jndi: jndi를 쉽게 다루도록 도와주는 패키지
- remoting: 스프링의 remoting 과 관련된 패키지
- scheduling: Spring Scheduling 기능이 구현되어 있다.
- scripting: Spring의 Scripting을 도와주는 패키지
- stereotype: Spring IoC의 핵심 어노테이션이 정의(Service, Controlller 등)
- ui: web의 view에서 사용할 theme 같은 것을 관리하는 패키지
- validation: Spring Core 문서의 Validation 과 관련된 기능들이 구현되어 있다.
```

#### **context**(\*): 위의 factory 패키지 내부 구조를 확대. 패키지 상위에는 applicationContext와 관련된 인터페이스들이 정의되어 있다.

```
- annotation(핵심1): Bean과 관련된 어노테이션들(@ComponentScan, @Bean, @Configuration 등)이 정의되어 있다. 
  또한 해당 어노테이션과 관련된 Processor, context 구현체들도 존재한다.
- config: 설정과 관련된 보조 패키지
- event: EventListener와 관련된 기능들이 구현되어 있다.
- expression: Expression Parsing과 관련된 보조 패키지
- i18n: Locale 과 관련된 패키지
- index: component index와 관련된 패키지
- support(핵심2): AppilcationContext의 실제 구현체들이 존재한다.
- weaving: Context에 대한 Load time weaving 과 관련된 부분이 구현되어 있다.
```

#### **ApplicationContext 상위 클래스에 대한 분석**

ApplicationContext -> ListableBeanFactory -> BeanFactory 로 상속을 받고 있다.  
여전히 ApplicationContext는 빈을 읽는 정도와 관련된 메서드만 갖고 있다.


#### **ApplicationContext 주요 구현 클래스에 대한 분석**

**1) AbstractApplicationContext**  
상위 클래스들의 다양한 메서드들을 구현해 놓았다. 약 1,500줄의 코드를 가진다.  
  
**2) GenericApplicationContext**  
위의 추상클래스를 상속한 클래스.  
내부적으로 DefaultListableBeanFactory를 사용한다.(composition에 가깝다.)  
참고로 이는 BeanFactory의 가장 하위 클래스이며, Bean 관리에 대한 가장 구체화된 클래스이다.  
  
**3) AnnotationConfigApplicationContext**  
위의 GenericApplicationContext를 상속한 클래스다.  
Annotation과 관련된 설정을 자동으로 읽어오는 데에 특화된 Context 구현체이다.   
이외에도 GenericXmlApplicationContext, 또는 웹 모듈의 WebApplicationContext 가 있다.

다른 상위 클래스들과의 특징은 생성자를 실행하는 단계에서 컨텍스트를 초기화 시킨다는 것이다.  
즉 클래스를 생성함과 동시에 Context의 생성을 완료할 수 있다.


---

## **AnnotationConfigApplicationContext에서 Context를 생성하는 과정**

요새는 스프링을 사용할 때 대부분 Annotation을 사용하여 셋팅하기 때문에 이 클래스를 집중적으로 분석하기로 했다.

IDE를 이용해 코드를 따라가면서 분석해보았다. 

#### **1) 생성자 실행**

해당 Context를 생성할 때 두 가지 방법으로 Context 를 초기화 시킬 수 있다.

```
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    this();
    register(componentClasses);
    refresh();
}

public AnnotationConfigApplicationContext(String... basePackages) {
    this();
    scan(basePackages);
    refresh();
}

```

굉장히 직관적인데, 클래스들을 넣어주거나, basePackage를 직접 넣어주면  
register나 scan을 진행하고, refresh를 하는 방식으로 진행될 것이다.  
  
일반적으로는 scan 을 사용하는 상황이 많기 때문에 그 쪽으로 따라가도록 하겠다.

#### **2) 스캔 메서드 실행**

AnnotationConfigApplicationContext에는 내부적으로 ClassPathBeanDefinitionScanner  
가 scanner로 선언되어 있다.

```
@Override
public void scan(String... basePackages) {
    ...
    this.scanner.scan(basePackages);
    ...
}

```

  
scan 메서드에서는 이 ClassPathBeanDefinitionScanner를 이용하여 스캔을 시작한다.

#### **3) 등록할 Bean들을 실질적으로 찾기**

Scanner 클래스에서는 먼저 basePackage들을 바탕으로 Bean으로 등록될 후보들을 찾는 작업을 한다.

이 구체적인 작업은 **ClassPathScanningCandidateComponentProvider** 에서 진행된다.  
  
이 provider에는 내부적으로 component가 아닌 것을 거르고 Bean의 대상자들을 가져온다.  
파일을 읽는 과정에서 Spring core의 Resource 클래스들이 사용된다.

#### **4) 등록할 Bean들을 검증하고 저장하기**

그리고 **ClassPathBeanDefinitionScanner** 에서는  
각종 검증 작업을 거친 뒤에 **Bean**들을 **registry**에 저장하게 된다. 

#### **5) ApplicationContext 생성을 마무리하기**

다음 작업은 refresh() 를 호출하는 일이다. 이는 상위 클래스인 **AbstractApplicationContext**에 구현되어 있다.

사실 이 과정이 제일 복잡한 듯 하다. 메서드 내부적으로 하는 것들이 굉장히 많다.  
이 과정에서 bean post processor 같은 것들이 동작하는 것을 확인했는데, 각종 후처리들을 담당하는 것 같다.  
그리고 최종적으로 context 초기화가 마무리 된다.

---

## **코딩 Convention에 대한 약간의 분석**

#### **\- 오버로딩중인 메서드들을 do~ 를 붙여서 최종 구현을 하는 패턴**

예를 들면 getBean이라는 오버로딩 된 다수의 메서드들이 doGetBean이라는 구체화된 private 메서드들을 호출하는 식으로 구현되었다.


####   
**\- interface, abstract, class**

인터페이스에는 **명세**만,

Abstract 클래스에는 **핵심 코드들을 정의**,

최종 클래스에는 **각 상황에 맞춘 구체화된 로직**을 갖는다. 

이전에는 Abstract class도 interface와 크게 다르지 않다고 생각했는데 실제 구현을 보니 그러지 않았다.

#### **\- 주석**

길어지는 소스코드에는 주석을 붙이는 편이다. 실제로 읽을 때 도움이 많이 된다. 아무리 객체지향적으로 잘 구현되어 있어도 어느 한 부분은 절차지향적으로 짜일 수 밖에 없다.

#### **\- 코드의 분리 정도**

리팩터링 책의 철학과는 반대되는 것 같긴 한데,

가끔 핵심 메서드는 나누지 않고 길게 만드는 것이 가독성에 도움이 되는 것 같기도 하다.  
내부적으로 오히려 private 메서드로 많이 잘라놓으면 왔다갔다 정신이 없을 수도 있을 것 같다.  
스프링은 대부분의 핵심 메서드는 무지하게 길다. 다만 주석이 깔끔해서 괜찮은 듯 하다.

(물론 복잡한 로직은 무조건 private으로 빼긴 한다) 

#### **\- 인터페이스의 구현**

인터페이스를 여러개 구현한 클래스의 경우, 오버라이딩할 인터페이스들을 구역을 나눠 주석으로 구분하기도 한다.

