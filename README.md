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
