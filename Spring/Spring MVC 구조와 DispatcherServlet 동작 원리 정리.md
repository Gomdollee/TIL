> 스프링 공부하다가 Spring MVC 구조와 DispatcherServlet 에 대한 지식이 부족한거같아 따로 정로 하기로 함 

# Spring MVC 구조와 DispatcherServlet 동작 원리 정리
## 먼저 웹 요청은 어떻게 들어오는가? 

브라우저에서 어떤 특정 주소를 요청하면 예를 들어 
``` http
GET /users/1
```
이 요청은 서버로 들어온다.

그런데 Java 진영에서는 이 요청을 누가 처음받냐면 바로 **Servlet 기반 웹 컨테이너가** 가 받는다. 

대표적으로 
- Tomcat 
- jetty 
- Undertow 

이런 것들이 **Servlet Container / Web Contatiner** 이다. 

여기서 대략적인 흐름은 
``` http
브라우저 -> 톰캣(서블릿 컨테이너) -> 서블릿 -> 스프링 MVC -> 컨트롤러 -> 서비스 -> 레포지토리 -> DB 
```

여기서 중요한 건 
- 톰캣은 요청을 받아서 적절한 Servlet에게 전달 
- Spring MVC는 그 Servlet 위에서 동작하는 프레임워크 
- DispatcherServlet은 Spring MVC의 핵심 Servlet 
  
이다. 

## Servlet이란 무엇인가 
### Servlet의 본질 
Servlet은 한마디로 말하면 
> HTTP 요청을 받아서 처리하고, HTTP 응답을 만드는 Java 객체 규약 

이다. 

자바 웹 초기에는 개발자가 직접 Servlet을 만들어서 요청을 처리했다. 

예를들면 
``` java
@WebServlet("/hello")
public class  HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response throws IOException) {

        response.setContentType("text/plain;charset=UTF-8");
        response.getWriter().write("hello");
    }
}
```

이 코드에서 봐야할 점은 
- `HttpServletRequest` : 요청 정보 
- `HttpServletResponse` : 응답 생성 
- `doGet()` : GET 요청 처리
- `doPost()` : POST 요청 처리

이다. 

즉, Servlet은 개발자가 직접 이렇게 작성하던 웹 진입점이었다. 

## 그럼 Servlet이 왜 중요한가? 
결국 Spring MVC를 쓰더라도 그 바닥에는 Servlet이 깔려있기 때문이다. 

예를 들어 우리가 흔히 쓰는 
- `@Controller`
- `@RestController`
- `GetMapping`
- `PostMapping`

이런 것들은 다 편한 추상화일 뿐이고, 실제 HTTP 요청/응답은 결국 **Servlet API** 를 통해 다뤄진다 

쉽게 갱각해 Spring MVC는 Servlet을 없앤 게 아니라 Servlet을 더 잘 쓰게 감싼 것이다. 

## Servlet Contatiner는 무엇인가 
Servlet 혼자서는 동작할 수 없다 
누군가는 Servlet을 생성하고, 초기화하고, 요청마다 호출해줘여 한다 

그 역항을 하는 것이 **Servlet Contatiner**이다. 

대표적으로 Tomcat이 이 역할을 한다. 

대략적으로 하는 일을 정리하자면 
- 서버 소켓 열기 
- HTTP 요청 받기 
- 요청 파싱
- 어떤 Servlet이 처리할지 찾기 
- Servlet 객체 생성/관리 
- service() 호출 
- 응답 반환 
- 생명주기 관리 

즉, 개발자는 비즈니스 로직만 작성하고 실제 네트워크 처리와 객체 생명주기는 컨테이너가 맡아주는 구조다. 

## Servlet 생명주기 
Servlet은 요청마다 새로 만드는 것이 아니라 보통 한 번 생성해두고 재사용 한다. 

기본 생명주기를 설명하면 

### init()
서블릿이 처음 생성될 때 한 번 호출 된다. 

### service()
요청이 들어올 때마다 호출 된다. 

HTTP 전용 서블릿인 HttpServlet에서는 내부적으로 메서드를 나눈다. 

- GET → doGet()
- POST → doPost()
- PUT → doPut()
- DELETE → doDelete()

### destroy()
서버 종료나 서블릿 제거 시 호출 된다. 
즉 
``` http
초기화 1번 -> 요청마다 service/doGet/doPost 반복 → 종료 시 destroy
```

## Servlet 방식의 한계 

Servlet만으로 웹 개발을 하면 너무 불편하다. 

예를 들어 매번 해야 하는 것 
- URL 매핑 직접 관리 
- 파라미터 꺼내기 
- 문자열 변환 
- 예외 처리 
- 뷰 이동 처리 
- 중복 로직 처리 
- JSON 응답 수동 작성 
- 컨트롤러 구조화 어려움 

예를 들어 요청 파라미터 하나만 받아도 
``` java
String name = request.getParameter("name");
int age = Integer.parseInt(request.getParameter("age"));
```
이런 작업을 계속 해야한다. 

그래서 나온 게 MVC 패턴이고, 그 MVC를 Servlet 기반에서 잘 구현해주는 프레임워크가 **Spring MVC** 이다.

## Spirng MVC란 무엇인가 
spring MVC는 한마디로 
> Servlet 기반으로 MVC 패턴을 구현한 웹 프레임워크다. 

특징을 살펴보면 
- Front Controller 패턴 사용 
- 요청 매핑 자동화 
- 파라미터 바인딩 
- 타입 변환
- 검증
- 예외 처리
- 뷰 리졸빙 
- JSON/XML 응답 처리 

이 모든 걸 해결해 준다. 

그리고 이 Spring MVC의 중심에 있는 것이 바로 
> DispatchetServlet

이다. 

## DispatcherServlet이란 무엇인가

DispatcherServlet은 스프링 MVC의 핵심 서블릿이다. 

즉 

> 모든 웹 요청을 가장 먼저 받아서, 
> 적절한 컨트롤러로 보내고,
> 결과를 받아서 뷰 또는 응답으로 만들어준느 중앙 조정자

이다. 

쉽게 설명하면 
- Servlet 세계와 Spring 세계를 연결하는 다리 
- Spring MVC이 프론트 컨트롤러 
- 요청 처리 총관리자 

이다. 

## Front Controller 패턴이란 

DispatcherServlet을 이해하려면 먼저 Front Controller 패턴을 알아야 한다. 

예전 방식은 URL 마다 각각 Servlet이 따로 있었을 수 있다. 
``` java
/login → LoginServlet
/join → JoinServlet
/order → OrderServlet
```

이러한 방식은 공통 처리가 어렵다. 

예를 들면 
- 인증 체크 
- 로깅 
- 인코딩 설정 
- 예외 처리 
- 공통 응답 형식 

이런 걸 각 Servle마다 넣어야 한다. 

그래서 나온 방식이 
> 모든 요청을 하나의 진입점이 먼저 받고, 거기서 분기하자 

이다. 

그 하나의 진입점이 바로 Front Controller이고, Spring MVC에서는 그 구현체가 DispatcherServlet 이다. 

## DispatcherServlet의 역할

- 요청 받기
- 어떤 컨트롤러가 처리할지 찾기
- 적절한 메서드 호출
- 파라미터 바인딩
- 컨트롤러 결과 받기
- View 이름 해석 또는 JSON 변환
- 응답 반환
- 예외 발생 시 예외 처리기 호출

## 전체적인 흐름으로 보기 

``` java
클라이언트 요청
   ↓
Servlet Container(Tomcat)
   ↓
DispatcherServlet
   ↓
HandlerMapping
   ↓
HandlerAdapter
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
DB
   ↑
Controller 결과 반환
   ↓
ViewResolver or HttpMessageConverter
   ↓
HTTP Response
```

## Handler, HandlerMapping, HandlerAdapter란

### Handler
Handler는 요청을 처리할 대상이다. 

### HandlerMapping
DispatcherServlet이 요청을 받으면 어떤 핸들러가 처리해야할지 찾아야 한다.
그 역할이 HandlerMapping이다. 

예를 들면 
- /hello → hello() 메서드
- /users/1 → getUser(Long id) 메서드

이렇게 URL과 핸들러를 연결해준다. 

대표적으로 
- RequestMappingHandlerMapping

이 사용돠고 @RequestMapping, @GetMapping 등을 읽어서 매핑 정보를 관리한다. 

### HandlerAdapter 
핸들러를 찾았다고 바로 호출할 수는 없다. 

왜냐하면 핸들러 종류가 다양할 수 있기 때문이다. 

- 옛날 Controller 인터페이스 구현체
- HttpRequestHandler
- @RequestMapping 메서드 기반 핸들러

호출 방식이 다 다르다. 

그래서 스프링은 **어댑터 패턴**을 둔다. 

즉, “이 핸들러를 호출할 줄 아는 실행기”가 HandlerAdapter이다. 

