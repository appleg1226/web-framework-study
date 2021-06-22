## **1. module 분석**

스프링 프레임워크에서 web과 관련된 모듈은 총 네 가지가 있다.
websocket 을 제외한 세 가지를 알아보면 다음과 같다.  
  
**1) spring-web**  
spring-web 모듈은 web 서버를 구축하는 데 있어서 **공통적인 역할**을 하는 클래스들이 모여있는 곳이다.  
그러다보니 각종 인터페이스들이 많이 존재한다.  
예를 들면 **request, response**에 대한 인터페이스 같은 것들이다.  
아니면** Exception, Filter, Cors **같은 것들과 관련된 기능들이 들어있다.  
여기에 있는 클래스들은 나머지 webmvc, webflux, websocket 에서 공통적으로 사용될 것이다.  
  
**2) spring-webmvc**  
spring-webmvc 모듈에는 'servlet'이라는 하나의 패키지만 존재한다.   
servlet 패키지에는 **Tomcat**에서 사용하는 **DispatcherServlet** 이 존재한다.  
즉 완전히 T**omcat**에 종속된 모듈임을 알 수 있다.  
**Tomcat**에서 받아온 요청을 실제로 수행하는 핵심 역할을 하는 부분으로 구성되어 있다.  
  
**3) spring-webflux**  
spring-webflux 모듈은 **Reactive Streams** 을 지원하는 웹 프레임워크다.  
non-blocking 프레임워크를 만들 수 있으며 자세한 개념 설명은 여기선 생략한다.  
여기에도 위의 **DispatcherServlet**과 같은 기능을 하는 **DispatcherHandler** 라는 클래스가 존재한다.

---

## **2. DispatcherServlet 초기화 단계**

  
이전 Spring web 문서 관련 포스트에서도 정리했지만, **DispatcherServlet**은 Tomcat에서 가져다 쓰는 Spring의 클래스이다.   
Tomcat에서 이 **DispatcherServlet**을 찾아서 등록한 다음,   
요청이 들어왔을 때 이 **DispatcherServlet**에게 그 요청의 처리를 맡긴다.  
  
DispatcherServlet은 상속관계를 확인해보면  
그 위에 Spring의 **FrameworkServlet**과 **HttpServletBean**을 상속하고 있지만,  
결과적으로는 Serlvet 표준 스펙인 **HttpServlet**을 상속하고 있다.
  
그렇기 때문에 Tomcat이 이 dispatcherServlet을 자연스럽게 사용할 수 있는 것이다.  
Tomcat과 Spring이 서로 같이 아는 **'공통의 인터페이스'**를 구현했기 때문에 이게 가능하다.  
  
애초에 접근의 관계가 Tomcat -> Spring 이렇게 된다는 것을 염두해야 한다.  
즉, Tomcat은 Spring의 코드를 알지만, Spring은 Tomcat의 코드를 모른다.

  
그렇기 때문에 Spring은 Tomcat이 알 수 있게 설정을 **'선언'** 할 수 있는 방법 밖에 없다.

---

## **3. 선언 할 클래스 두가지**

Tomcat은 Spring에게 두 가지 **선언**을 요구한다.  

**첫 번째**는 본인들의 구동 순서에 맞게 ApplicationContext를 초기화 할 수 있도록 해주는 **Listener Class**,  
두 번째로는 사용할 구체적인 **Serlvet** 구현체를 요구한다.  
  
그게 web.xml 설정에 나와있는  
**ContextLoaderListener**와 **DispatcherServlet**이다.

```
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```

  
Tomcat 은 이 설정을 읽고 본인들의 어플리케이션에 등록할 것이다.  
  
ContextLoaderListener 는 내부적으로 **initWebApplicationContext** 이라는 메서드를 갖고 있기 때문에 

ApplicationContext를 초기화 시킬 수 있다.  
  
**DispatcherServlet**은 Tomcat이 추후에 생성자를 통하여 직접 만들어서 본인의 Servlet으로 관리하게 될 것이다.  

---

## **4\. DispatcherServlet 의 초기화**

DispatcherServlet의 코드를 보면 initStrategy을 호출하여 초기화 하는 것을 볼 수가 있다.   
그래서 이 initStrategy()를 도대체 어디서 호출하는지 계속해서 타고 올라가 보았다.

> initStrategy()  
> \-> DispatcherServlet의 onRefresh()에서 initStrategy()를 호출  
> \-> FrameworkServlet의 initWebApplicationContext()에서 onRefresh()을 호출  
> \-> FrameworkServlet의 initServletBean()에서 initWebApplicationContext()을 호출  
> \-> HttpServletBean의 init()에서 initServletBean()을 호출  
> \-> tomcat의 Apache.catalina.core.StandardWrapper의 initServlet()에서 init()을 호출

이런 식으로 체인이 구성되어 있다.  
(Debugger를 사용하면 한 방에 확인할 수 있다)  
  
initStrategies의 각 init 과정은 이제 내가 스프링에 작성한 코드들을 Scan하여 등록하는 일이 될 것이다.  
예를 들어, 내가 Controller에 작성한 메서드들은 initHandlerMappings 를 사용하여 찾아낼 것이다.  
  

---

## **5\. 나의 요청은 어떤 과정을 거치는가(Tomcat 부터 Servlet까지)**

  
(Tomcat에 대해서는 아직 공부하지 못했기 때문에 Tomcat은 Spring과 맞닿는 끝부분만 확인을 해볼 예정이다.)  
  
요청 과정이 어떻게 진행되는 지를 확인하기 위해서 IntelliJ의 Debugger를 사용했다.  
그림처럼 요청을 처리하는 가장 핵심부분이 되는 DispatcherService의 doService() 메서드에 Break Point을 찍었다.  
그러면 딱 그 부분에서 요청이 캡쳐될 것이다.
이제 어떤 과정을 거치는지 바로 확인을 할 수가 있다.
  
호출 스택을 확인하면 **DispatcherServlet**을 **FrameworkServlet**에서 호출하고,  
쭉쭉 타고 내려가다보면 **org.apache.catalina.core** 에서 실행을 하는 것을 확인할 수 있다.  
이 **Catalina**가 Tomcat의 패키지 이름이기 때문에 Tomcat과의 접점을 찾는 데에 성공한 것이다.  
  
그 다음으로는 내부적으로 어디까지 들어가나 Debugger의 **Step-Into** 를 열심히 눌러 보았다.   

> doService()  
> \-> **doService**: doDispatch() 호출하여 실질적인 요청 처리  
> \-> **doDispatch**: handlerAdapter를 가지고 handle() 호출하여 요청 처리  
> \-> **RequestMappingHandlerAdapter**: handleInternal() 호출  
> \-> **RequestMappingHandlerAdapter**: invokeHandlerMethod() 호출  
> \-> **RequestMappingHandlerAdapter**: invokeAndHandle() 호출  
> \-> **ServletInvocableHandlerMethod**: invokeForRequest()를 이용하여 controller 메서드의 return값 가져옴  
> \-> **ServletInvocableHandlerMethod**: handleReturnValue()을 이용하여 return 과정 처리 요청  
> \-> **HandlerMethodReturnValueHandlerComposite**: handleReturnValue() 호출하여 처리 요청  
> \-> ...  
> \-> Flush를 이용하여 tomcat에서 response 처리

  
생각보다 굉장히 복잡한 과정을 가지는데, Call Stack이 꽤나 길다.   
딱 **HttpEntity**를 만드는 과정까지만 따라갔다.  
이후에는 Tomcat에서 세부적으로 Response를 처리하는 과정을 거친다.  
  

---

## **6. 결론**

정리하자면 주로 살펴본 부분은 두 가지이며, 이 두 가지가 web 프레임워크의 핵심이다.  
  
**1) Initialization**  
**2) request 처리**  
  
**Initialization**의 분석은 그 핵심인 **ContextLoaderListener**와 **DispatcherServlet**의 **initStrategy()** 을 중심으로 진행했으며  
  
**request 처리**는 debugger를 이용하여 DispatcherServlet의 **doService**를 중심으로 전후 과정을 파악하는 데에 주력했다.
