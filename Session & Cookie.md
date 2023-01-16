# Session & Cookie



## 목차

- [HTTP의 특징과 쿠키와 세션을 사용하는 이유](#HTTP의-특징과-쿠키와-세션을-사용하는-이유)
- [쿠키 (Cookie)](#쿠키-(Cookie))
- [쿠키의 동작과정](#쿠키의-동작과정)
- [세션 (Session)](#세션-(Session))
- [세션의 동작 방식](#세션의-동작-방식)
- [HttpSession을 사용한 로그인 및 로그아웃 처리](#HttpSession을-사용한-로그인-및-로그아웃-처리)
- [세션 만료시간 설정](#세션-만료시간-설정)
- [인터셉터](#인터셉터)
- [스프링 인터셉터 구현](#스프링-인터셉터-구현)



## HTTP의 특징과 쿠키와 세션을 사용하는 이유

- ### HTTP 프로토콜의 특성이자 약점인 connectionless, stateless 를 보완하기 위해 사용

  - #### Connectionless

    - ##### 클라이언트가 요청을 한 후 응답을 받으면 그 연결을 끊어버리는 특징

    - ##### HTTP는 먼저 클라이언트가 Request를 서버에 보내면, 서버는 클라이언트에게 요청에 맞는 Response를 보내고 접속을 끊는 특성

  - #### Stateless

    - ##### 통신이 끊나면 상태를 유지하지 않는 특징

    - ##### 연결을 끊는 순간 클라이언트와 서버의 통신이 끝나며 상태 정보는 유지하지 않는 특성

    - ##### 예를들면 로그인 페이지에서 로그인을 성공했음에도, 페이지를 이동할때 마다 계속 로그인을 해야함

- ### 쿠키와 세션을 사용했을 경우, 한 번 로그인을 하면 그 사용자에 대한 인증을 유지



## 쿠키 (Cookie)

- ### 클라이언트 로컬에 저장되는 키와 값이 들어있는 작은 데이터 파일

- ### 사용자 인증이 유효한 시간을 명시할 수 있으며, 유효 시간이 정해지면 브라우저가 종료되어도 인증이 유지

- ### 클라이언트의 상태 정보를 로컬에 저장했다가 참조

- ### 클라이언트에 300개까지 쿠키저장 가능, 하나의 도메인당 20개의 값만 가질 수 있음

- ### Response Header에 Set-Cookie 속성을 사용하면 클라이언트에 쿠키를 만들 수 있음

- ### 쿠키는 사용자가 따로 요청하지 않아도 브라우저가 Request시에 Request Header를 넣어서 자동으로 서버에 전송

- ### 구성요소

  - #### 이름 : 각각의 쿠키를 구별하는데 사용되는 이름

  - #### 값 : 쿠키의 이름과 관련된 값

  - #### 유효시간 : 쿠키의 유지시간

  - #### 도메인 : 쿠키를 전송할 도메인

  - #### 경로 : 쿠키를 전송할 요청 경로




## 쿠키의 동작과정

- ### 클라이언트가 페이지를 요청

- ### 서버에서 쿠키를 생성

- ### HTTP 헤더에 쿠키를 포함시켜 전송

- ### 브라우저가 종료되어도 쿠키 만료 기간이 있다면 클라이언트에서 보관하고 있음

- ### 같은 요청을 할 경우 HTTP 헤더에 쿠키를 함께 보냄

- ### 서버에서 쿠키를 읽어 이전 상태 정보를 변경 할 필요가 있을때 쿠키를 업데이트하여 변경된 쿠키를 HTTP 헤더에 포함시켜 응답



## 세션 (Session)

- ### 세션은 쿠키를 기반하고 있지만, 사용자 정보 파일을 브라우저에 저장하는 쿠키와 달리 세션은 서버 측에서 관리

- ### 서버에서는 클라이언트를 구분하기 위해 세션 ID를 부여하며 웹 브라우저가 서버에 접속해서 브라우저를 종료할 때까지 인증상태를 유지

  - #### 접속 시간에 제한을 두어 일정 시간 응답이 없다면 정보가 유지되지 않게 설정이 가능

- ### 사용자에 대한 정보를 서버에 두기 때문에 쿠키보다 보안에 좋지만, 사용자가 많아질수록 서버 메모리를 많이 차지

  - #### 동접자 수가 많은 웹 사이트인 경우 서버에 과부하를 주게 되므로 성능 저하의 요인

- ### 클라이언트가 Request를 보내면, 해당 서버의 엔진이 클라이언트에게 유일한 ID를 부여(세션 ID)



## 세션의 동작 방식

- ### 클라이언트가 서버에 접속 시 세션 ID를 발급 받음

- ### 클라이언트는 세션 ID에 대해 쿠키를 사용해서 저장하고 가지고 있음

- ### 클라이언트는 서버에 요청할 때, 이 쿠키의 세션 ID를 같이 서버에 전달해서 요청

- ### 서버는 세션 ID를 전달 받아서 별다른 작업없이 세션 ID로 세션에 있는 클라이언트 정보를 가져와서 사용

- ### 클라이언트 정보를 가지고 서버 요청을 처리하여 클라이언트에게 응답



## HttpSession을 사용한 로그인 및 로그아웃 처리

- ### 서블릿에서 HttpSevlet 클래스를 통해 제공

- ### HttpSession을 사용하면 세션 생성, 조회, 삭제를 편하게 사용 가능

- ### 추적 불가능한 키를 가진 쿠키 생성

  - #### 쿠키명은 JSESSIONID이며 타입은 HttpOnly이기에 클라이언트에서 조작할 수 없음

- ### SessionController

<img width="1294" alt="스크린샷 2023-01-15 오후 5 32 19" src="https://user-images.githubusercontent.com/51348720/212531189-39c2ea59-e07c-4757-84a1-0acb9c2487b9.png">

```java
@Api("세션 로그인 및 로그아웃")
@RestController
@RequestMapping("${property.api.end-point}")
@Slf4j
public class SessionController {

    // 로그인, 세션로그인
    @PostMapping("/auth/sessionLogin")
    public String login(@RequestBody UserDto user, HttpServletRequest request){
        // 유저 인증 서비스에서 유저 조회시 유저가 없을땐 N / 있으면 세션값 생성 및 email 정보 세션안으로 set
        if (isEmpty(user)) {
            return "N";
        } else {
          	// 세션 생성
            HttpSession session = request.getSession(true);
            log.info("세션 생성 전 : {}", session.getAttribute(SessionConst.SESSION_USER_EMAIL));
            session.setAttribute(SessionConst.SESSION_USER_EMAIL, user.getEmail());
            log.info("JSESSIONID : {}", session.getId());
            log.info("세션 생성 후 : {}", session.getAttribute(SessionConst.SESSION_USER_EMAIL));
            log.info("세션 생성 날짜 : {}", new Date(session.getCreationTime()));
            log.info("세션 유효 시간 : {}분", session.getMaxInactiveInterval() / 60);
            log.info("최근 세션 접근 시간 : {}", new Date(session.getLastAccessedTime()));
            return "Y";
        }
    }
  
  // 로그아웃, 세션 로그아웃
    @GetMapping("/auth/sessionLogout")
    public String logout(HttpServletRequest request) {
        HttpSession session = request.getSession();

        log.info("JSESSIONID : {}", session.getId());
        log.info("세션 삭제 전 : {}", session.getAttribute(SessionConst.SESSION_USER_EMAIL));

        if(session == null) {
            return "N";
        } else {
            // 세션 삭제
            session.removeAttribute(SessionConst.SESSION_USER_EMAIL);
            // 사용자가 요청을 또 보내면 모든 정보가 그대로 남아있기 때문에 재생성
            // request.getSession(true);
            log.info("세션 삭제 후 : {}", session.getAttribute(SessionConst.SESSION_USER_EMAIL));
            log.info("세션 삭제 날짜 : {}", new Date());
            return "Y";
        }
    }
}
```

- ### request.getSession() 

  -  #### getSession 메서드는 세션을 생성 혹은 조회하는 메서드

  - #### true일 경우 

    - ##### 세션이 있으면 기존 세션을 반환

    - ##### 없으면 새로운 세션을 생성

  - #### false일 경우

    - ##### 세션이 있으면 기존 세션을 반환

    - ##### 없으면 Null 반환

  - #### default값은 true

- ### setAttribute(세션 정보)

  - #### 파라미터로 준 사용자의 정보를 세션에 담아줌

- ### removeAttribute(세션 정보)

  - #### 파라미터로 준 세션의 정보를 세션에서 제거

- ### getId() 

  - #### 세션 아이디(JSESSIONID)의 값

- ### getCreationTime()

  - #### 세션 생성일시

- ### getMaxInactiveInterval()

  - #### 세션의 유효시간

- ### getMaxInactiveInterval()

  - #### 세션과 연결된 사용자가 최근에 서버에 접속한 시간

  - #### 클라이언트에서 서버로 sessionId를 요청한 경우 갱신

  

## 세션 만료시간 설정

- ### HTTP는 Connectionless 이기에 서버측에선 클라이언트가 웹 브라우저를 종료했는지를 알 수 없다. 그렇기에 세션을 언제 삭제해야할지 판단하기 어려움

- ### 보안성 문제점

  - #### JESESSIONID를 탈취당한 경우 시간이 흘러도 해당 쿠키로 악용될 수 있음

  - #### 세션은 기본적으로 메모리에 생성되는데, 메모리의 크기가 무한하지 않기에 사용하지 않는 세션이 관리되지 않으면 성능저하가 발생

- ### 스프링부트에서는 application.yml 또는 application.properties에서 글로벌 설정 가능

  - #### 타임아웃을 설정하면 WAS 내부에서 해당 세션을 삭제

- ### application.yml

```yaml
server:
  port: 8080
  servlet:
    session:
      timeout: 1800 # 세션 만료 1800초(30분)
```



## 인터셉터

- ### 클라이언트의 요청과 관련되어 전역적으로 처리해야하는 작업을 처리

  - #### 로그인을 하지 않은 사용자는 접근할 수 있는 페이지가 제한적이며 로그인이 필요한 페이지 접근이 허용되서는 안됨

  - #### 로그인이 필요한 모든 컨트롤러 로직에 로그인 여부를 확인하는 코드를 작성하는 것은 비효율적임

  - #### 여러 로직에서 공통으로 관심이 있는 부분을 공통 관심사(cross-cutting concerns)라 함

  - #### AOP로 처리할수 있으나, 웹과 관련된 공통 관심사를 처리할 때는 서블릿 필터 또는 스프링 인터셉터에서 처리하는게 좋다.

- ### HttpServletRequest나 HttpServletResponse 등과 같은 객체 자체를 조작할 수는 없으나, 내부적으로 갖는 값은 조작할 수 있음

- ### 인터셉터의 흐름

  - #### HTTP 요청

  - #### WAS

  - #### 필터

  - #### 서블릿

  - #### 스프링 인터셉터

  - #### Controller

  - #### 즉 디스페처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출됨

- ### 페이지마다 컨트롤러의 api를 호출하기전에 가로채서 세션 인증을 하기 위해 인터셉터 사용

- ### 인터셉터는 세분화 되어있음

  - #### preHandler(컨트롤러 호출 전)

    - ##### 컨트롤러 호출 전에 호출되며 반환 타입은 Boolean

    - ##### 반환 값이 false면 그 뒤는 진행하지 않음

  - #### postHandle(컨트롤러 호출 후)

    - ##### 핸들러 어댑터 호출 후 호출

    - ##### 컨트롤러에서 예외가 발생하면 postHandle은 호출되지 않음 

  - #### afterCompletion(요청 완료 이후)

    - ##### view가 렌더링 된 후에 호출

    - ##### 항상 호출되기에 이전에 발생한 예외가 있을 경우 파라미터로 받아서 어떤 예외가 발생했는지 확인 가능

  - #### 예외가 발생하면 PostHandle()같은 경우 호출되지 않기 때문에 예외처리가 필요하다면 afterCompletion()을 사용해야 함



## 스프링 인터셉터 구현

- ### SessionLogInterceptor

```java
@Slf4j
public class SessionLogInterceptor implements HandlerInterceptor {

    public static final String LOG_ID = "logId";

    // 컨트롤러 호출 전에 호출
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse
            response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        //인터셉터는 싱글톤패턴이기 때문에 setAttribute를 활용하여 밑에 있는 afterCompletion에게 UUID값을 전달한다
        request.setAttribute(LOG_ID, uuid);

        if (handler instanceof HandlerMethod) {
            //호출할 컨트롤러 메서드의 모든 정보가 포함되어 있다.
            HandlerMethod hm = (HandlerMethod) handler;
            log.info("Handler Method : {}" , hm);
        }

        log.info("REQUEST : [{}] / [{}] / [{}]", uuid, requestURI, handler);
        return true; //false면 다음 호출이 진행되지 않는다

    }

    // 컨트롤러 호출 후에 호출
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse
            response, Object handler, ModelAndView modelAndView) throws Exception {

        log.info("postHandle : [{}]", modelAndView);

    }

    // 뷰 랜더링 후에 호출
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse
            response, Object handler, Exception ex) throws Exception {

        String requestURI = request.getRequestURI();
        String logId = (String)request.getAttribute(LOG_ID);
        log.info("RESPONSE : [{}] / [{}]", logId, requestURI);
        if (ex != null) {
            log.error("AfterCompletion : {}", ex);
        }
    }
}
```

- ### request.setAttribute(LOG_ID, uuid)

  - #### 서블릿 필터와는 다르게 스프링 인터셉터는 호출 시점이 분리되어 있기에 각각의 메서드가 호출되는 시점에 변수들의 값 유지가 되지 않음

  - #### preHandle에서 지정한 값을 postHandle이나 afterCompletion에서 사용하려면 어딘가에 담아둬야 하는데, 이 인터셉터는 싱글톤패턴으로 사용되기에 멤버변수를 사용하면 안됨

  - #### setAttribute를 활용하여 밑에 있는 afterCompletion에게 UUID값을 전달

- ### HandlerMethod hm = (HandlerMethod) handler

  - #### Object handler 매개변수에는 핸들러 정보고 HandlerMethod가 넘어옴

- ### SessionInterceptor

  - #### 실제 로직을 구현

- ### SessionLogInterceptor이 먼저 실행되게 webConfig에서 지정을하였기 때문에 Handler Method의 로그가 나옴([WebConfig 설정](#WebConfig))

<img width="1294" alt="스크린샷 2023-01-15 오후 5 33 01" src="https://user-images.githubusercontent.com/51348720/212531236-6f26d10e-1aa0-4221-9e03-5773cf03e540.png">

- ### 세션 인증 로직

```java
@Slf4j
public class SessionInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        log.info("인증 체크 인터셉터 실행 : {}", requestURI);
        HttpSession session = request.getSession(false);


        // 포스트맨 테스트 예외처리
        if("Y".equals(request.getHeader("testYn"))) {
            return true;

        } else {
            //인증이 불가능한 사용자의 접근 검증
            if (isEmpty(session)  || isEmpty(session.getAttribute(SessionConst.SESSION_USER_EMAIL))) {
                log.info("미인증 사용자 요청");
					
              	// 화면단에서 응답코드를 403으로 줘서 세션 만료처리
                response.setStatus(403);

                //컨트롤러 호출을 막기 위해 false 반환
                return false;

            } else {

                log.info("JSESSIONID  : {}", session.getId());
                log.info("세션 정보 : {}", session.getAttribute(SessionConst.SESSION_USER_EMAIL));
                log.info("세션 생성 날짜 : {}", new Date(session.getCreationTime()));
                log.info("세션 유효 시간 : {}분", session.getMaxInactiveInterval() / 60);
                log.info("최근 세션 접근 시간 : {}", new Date(session.getLastAccessedTime()));

                //인증이 가능한 사용자는 다음 컨트롤러 호출
                return true;

            }
        }
    }
}

```

- ### 세션이 만료되었을때

<img width="1293" alt="스크린샷 2023-01-15 오후 5 36 18" src="https://user-images.githubusercontent.com/51348720/212531497-68debbcd-0a92-448e-8a98-7b88e27a815c.png">

<img width="714" alt="스크린샷 2023-01-15 오후 5 35 50" src="https://user-images.githubusercontent.com/51348720/212531418-f7280fd7-a3b1-4d31-87e9-290ca84681e0.png">



- ### WebConfig

  - #### 생성한 스프링 인터셉터를 설정에 등록

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        //로그를 출력하는 인터셉터
        registry.addInterceptor(new SessionLogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns(
                        "/api/userAthn/login",
                        "/api/auth/sessionLogin",
                        "/api/auth/sessionLogout",
                        "/api/userAthn/signUp",
                        "/api/userAthn/emailCheck",
                        "/api/userAthn/pwChg",
                        "/error"
                );

        //로그인 체크 인터셉터
        registry.addInterceptor(new SessionInterceptor())
                //로그 인터셉터 뒤에 작동할 수 있도록 설정
                .order(2)
                //인터셉터를 적용할 URL 패턴 설정
                .addPathPatterns("/**")
                //인터셉터 적용을 제외 할 URL 패턴 설정
                .excludePathPatterns(
                        "/api/userAthn/login",
                        "/api/auth/sessionLogin",
                        "/api/auth/sessionLogout",
                        "/api/userAthn/signUp",
                        "/api/userAthn/emailCheck",
                        "/api/userAthn/pwChg",
                        "/error"
                );
    }

}
```

- ### addInterceptor 

  - #### 인터셉터 등록

- ### order

  - #### 인터셉터의 호출 순서를 지정

- ### addPathPatterns()

  - #### 인터셉터를 적용할 URL 패턴을 지정

- ### excludePathPatterns()

  - #### 인터셉터에서 제외할 패턴을 지정