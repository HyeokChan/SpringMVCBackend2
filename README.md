# Spring MVC 2
***
### 타임리프
#### 특징
1. 서버 사이드 HTML 렌더링
    - 백엔드 서버에서 HTML을 동적으로 렌더링 하는 용도로 사용된다.
2. 네츄럴 템플릿
    - 순수 HTML을 유지하는 특징이 있어, 오직 서버를 통해 렌더링 되어야 하는 JSP와 달리 웹 브라우저에서 직접 파일을 열어도 미리 지정한 내용을 확인할 수 있다.
    - 또한, 서버를 통해 뷰 템플릿을 거치면 서버로 부터 전달된 동적인 결과를 확인할 수 있다.
3. 스프링 통합지원
    - 스프링과 자연스럽게 통합되고, 스프링의 다양한 기능을 활용할 수 있다.

#### 자주 사용하는 표현식
1. 변수 표현식 : ${...}
2. 기본 객체 : 
    - http 요청 파라미터 접근 : ${param.username}
    - http 세션 접근 : ${session.lgnId}
    - 스프링빈 접근 : ${@hellBean.hello('Spring')}
3. URL 링크 : @{...}
4. 쿼리 파라미터 : @{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}
5. 속성 추가 : 
    - th:attrappend : 속성 값의 뒤에 값을 추가
    - th:attrprepend : 속성 값의 앞에 값을 추가
    - th:classappend : class 속성에 값을 추가
6. 반복 : th:each="info : ${list}"
7. 조건 : th:if="${bool}"
8. 템플릿 : 
    - th:replace : 조각으로 대체
    - th:insert : 조각 추가

#### 스프링 통합
1. form 처리
    - th:object="${...}" : 객체 지정
    - th:field="*{...}" : 객체의 property 사용, id, name 처리
2. 체크박스 처리
    - 체크박스에 th:field 처리를 하여 체크여부에 따라 true/false 값으로 전송 가능

***

### 메시지, 국제화
#### 메시지
1. messages.properties와 같은 관리용 파일을 통해 다양한 메시지를 한 곳에서 관리
2. 관리 설정
    - item : 상품
    - item.itemName : 상품명
3. 사용
    - th:text="#{item.itemName}"

#### 국제화
1. 메시지 관리 파일을 나라별로 관리하여 서비스를 국제화
2. messages_en.properties, messages.properties 파일로 영어, 한국어 관리
3. 접근 나라 분류 
    - Accept-Language 헤더 값을 통해 Locale 정보 접근
    - Locale에 따라 messages.properties 자동 선택

***

### 검증 - Validation
#### VO에 검증 어노테이션 추가
1. @NotBlank : 빈 값 + 공백만 있는 경우 허용 안함
2. @NotNull : null을 허용 안함
3. @Range(min = 1000, max = 100000) : 범위 안의 값만 허용
4. @Max(9999) : 최대 9999까지 허용
5. @Min(1) : 최소 1까지 허용
#### BeanValidator 한계
1. 등록화면, 수정화면과 같이 다른 기능 처리에 같은 VO를 사용하면 검증 로직이 어긋날 수 있다.
2. 해결 방법
    - VO Validation 어노테이션에 groups를 지정해서 group 별로 validation을 동작한다.
    - 등록처리, 수정처리에 다른 VO를 사용한다.

***

### 로그인처리 - 쿠키, 세션
#### 쿠키
1. 영속쿠키 : 만료날짜를 입력하면 해당 날짜까지 유지
2. 세션쿠키 : 만료날짜를 생략하면 브라우저 종료시까지 유지
3. 쿠키 생성
    ~~~
    Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
    response.addCookie(idCookie);
    return "redirect:/";
    ~~~
4. 쿠키 사용
    ~~~
    @GetMapping("/")
    public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, 
                            Model model){
        ...
    }
    ~~~
5. 로그아웃 처리 : 쿠키의 종료날짜를 0으로 지정
    ~~~
    Cookie cookie = new Cookie(cookieName, null);
    cookie.setMaxAge(0);
    response.addCookie(cookie);
    ~~~
6. 보안문제
    - 쿠키값은 임의로 변경 가능
    - 쿠키에 보관된 정보 노출 가능
    - 악의적인 요청을 계속 시도할 수 있음
7. 보안문제 대안
    - 쿠키에는 사용자 별로 예측 불가능한 임의의 토큰 값을 노출하고 서버에서 토큰과 사용자Id를 매핑해서 관리
    - 토큰의 만료시간을 짧게 유지

#### 세션
1. 쿠키의 보안문제를 해결하기 위해 중요한 정보는 서버에 저장하고, 클라이언트와 서버는 임의의 식별자 값으로 연결
2. 서버에서 임의의 식별자 값으로 세션id 생성
3. 생성된 세션id와 보관될 값을 서버의 세션 저장소에 보관
4. 서블릿 http 세션
    - HttpSession session = request.getSession(); : 세션이 있으면 세션 반환, 없으면 세션 생성
    - session.setAttribute("loginInfo", loginMember); : 세션정보 저장
    - 세션삭제 : 
        ~~~
        HttpSession session = request.getSession(false); 
        if (session != null) {
            session.invalidate();
        }
        ~~~
5. 세션 어노테이션
    - @SessionAttribute : 세션을 편리하게 사용할 수 있도록 스프링이 지원하는 어노테이션
    - @SessionAttribute(name = "loginMember", required = false) Member loginMember : 파라미터 값으로 바로 사용 가능
    - (기존 세션이 없을 때, 신규 세션을 생성하지는 않음)
6. 세션id url노출 옵션
    - 웹브라우저가 쿠키를 지원하지 않을 때, 쿠키 대신에 url을 통해서 세션을 유지한다.
    - 이 방법을 사용하려면 url에 세션id값을 포함해서 전달해야 한다.
    - 쿠키를 통해서만 세션을 유지하는 방법 (url X)
        ~~~
        application.properties > 
        server.servlet.session.tracking-modes=cookie    
        ~~~
7. 타임아웃 설정 : 마지막 요청 시간으로 부터 1800초 유지
    ~~~
    application.properties > 
    server.servlet.session.timeout=1800
    ~~~

### 로그인처리 - 필터, 인터셉터
#### 서블릿 필터
1. 필터 흐름 : http요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
2. 필터 인터페이스 Filter 메서드 
    - init() : 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출
    - doFilter() : 고객의 요청이 올 때마다 해당 메서드가 호출됨. 필터 로직 구현 부분
    - destroy() : 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출됨
#### 스프링 인터셉터
1. 스프링 인터셉터 흐름 : http요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
2. 스프링 인터셉터 인터페이스 HandlerInterceptor 메서드
    - preHandle() : 컨트롤러 호출 전에 호출됨
    - postHandle() : 컨트롤러 호출 후에 호출됨
    - afterCompletion() : 뷰가 렌더링 된 이후에 호출됨

### 예외처리, 오류페이지
#### 예외 동작
1. 예외 동작 방식 : 컨트롤러(예외발생) -> 인터셉터 -> 서블릿 -> 필터 -> WAS
2. 예외 발생 테스트 : 
    - throw new xxxException(오류메시지) : 예외 발생 시킴
    - response.sendError(HTTP상태코드, 오류메시지) : 서블릿 컨테이너에게 오류가 발생했음을 알림
#### 오류화면 제공
1. 서블릿 오류페이지 등록
    ~~~
    @Component
      public class WebServerCustomizer implements
      WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
          @Override
          public void customize(ConfigurableWebServerFactory factory) {
              ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-
      page/404");
              ErrorPage errorPage500 = new
      ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
              ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-
      page/500");
              factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
          }
    }
    ~~~
    - 발생한 오류에 따라 에러 컨트롤러 호출
2. 에러 컨트롤러 동작
    ~~~
    @Slf4j
      @Controller
      public class ErrorPageController {
          @RequestMapping("/error-page/404")
          public String errorPage404(HttpServletRequest request, HttpServletResponse
      response) {
              log.info("errorPage 404");
              return "error-page/404";
          }
          @RequestMapping("/error-page/500")
          public String errorPage500(HttpServletRequest request, HttpServletResponse
      response) {
              log.info("errorPage 500");
              return "error-page/500";
          }
    }
    ~~~
    - 새로 작성한 에러 페이지 호출
3. 에러페이지 처리 순서 : WAS에서 오류 감지 -> 에러페이지 확인 -> 필터 -> 서블릿 -> 인터셉터 -> 컨틀롤러(에러컨트롤러) -> View
#### 서블릿 예외처리 - 필터
1. 이미 확인한 인증 및 권한을 WAS에서 오류를 감지하고 필터, 인터셉터를 거쳐갈 때, 다시 체크하는 것은 비효율적임
2. 필터, 인터셉터를 거쳐갈 때, 클라이언트로 부터 발생한 정상 요청인지, 오류페이지를 출력하기 위함인지 구별 필요
3. DispatcherType의 추가정보를 통해 식별 가능
4. DispatcherType
    - REQUEST : 클라이언트 요청
    - ERROR : 오류 요청
    - FORWADR : 서블릿에서 다른 서블릿이나 JSP 호출
    - INCLUDE : 서블릿에서 다른 서블릿이나 JSP의 결과를 포함
    - ASYNC : 서블릿 비동기 호출
5. 필터 설정
    ~~~
    FilterRegistrationBean<Filter> filterRegistrationBean = new
      FilterRegistrationBean<>();
              filterRegistrationBean.setFilter(new LogFilter());
              filterRegistrationBean.setOrder(1);
              filterRegistrationBean.addUrlPatterns("/*");
              filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST,
      DispatcherType.ERROR);
              return filterRegistrationBean;
    }
    ~~~
    - setDispatcherTypes 메서드를 통해 필터를 적용할 dispatcherType 설정, 기본값은 REQUSET만
#### 서블릿 예외처리 - 인터셉터
1. 인터셉터의 경우 서블릿이 제공하는 기능이 아니라 스프링이 제공하는 기능이다. 따라서 DispatcherType과 무관하게 항상 호출됨
2. 인터셉터 설정파일에서 오류페이지를 호출하는 경로를 excludePathPatterns를 사용하여 제외
    ~~~
    @Override
      public void addInterceptors(InterceptorRegistry registry) {
          registry.addInterceptor(new LogInterceptor())
                  .order(1)
                   .addPathPatterns("/**")
                .excludePathPatterns(
    "/css/**", "/*.ico"
    , "/error", "/error-page/**" //오류 페이지 경로
    );
        //@Bean
        public FilterRegistrationBean logFilter() {
            FilterRegistrationBean<Filter> filterRegistrationBean = new
    FilterRegistrationBean<>();
    }
    ~~~
#### 스프링부트 오류페이지
1. 스프링부트는 webServerCustomizer 생성, ErrorPage 설정, ErrorPageController 로직처리를 모두 기본으로 제공함
2. 기본적으로 오류가 발생했을 때, /error를 요청
3. 구체적인 에러페이지 파일명이 있으면 해당 View를 렌더링
##### 뷰 선택 우선순위
1. 뷰 템플릿
    - resources/templates/error/500.html
    - resources/templates/error/5xx.html
2. 정적리소스
    - resources/static/error/400.html
    - resources/static/error/404.html
    - resources/static/error/4xx.html
3. 적용대상이 없을 때
    - resources/templates/error.html

***