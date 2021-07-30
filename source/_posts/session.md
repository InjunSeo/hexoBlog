---
title: "[spring MVC] session"
date: 2021-07-27 12:55:53
tags:
---

# 1. Cookie에 담기
## 서버에서 쿠키를 생성, 조회하기
Cookie라는 클래스로 클라이언트에 응답할 쿠키정보 생성한다.

### 서버에서 쿠키 생성하기
```java
@PostMapping("login")
    public String loginV1(@ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        log.info("id= {}, pw= {}", form.getLoginId(), form.getPassword());
        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }
        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
        if (loginMember == null) {
            bindingResult.reject("loginFail", "Id or password is not correct");
            return "login/loginForm";
        }
        // session cookie
        Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getLoginId()));
        response.addCookie(idCookie);
        log.info("idCookie= {}", idCookie);
        return "home";
    }
```
* `new Cookie("memberId", String.valueOf(loginMember.getLoginId()))` : Cookie에 key/value 인수 담고 생성한다.
* `response.addCookie(idCookie)` : 방금 만든 Cookie인 idCookie를 response(HttpServletResponse로 만든 객체)에 담아준다.

### 서버에서 쿠키 조회하기
```java
GetMapping("/")
public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {
    if (memberId == null) {
        return "home";
    }

    Member loginMember = memberRepository.findById(memberId);
    if (loginMember == null) {
        return "home";
    }

    model.addAttribute("member", loginMember);
    return "loginHome";
}
```
* `homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model)` : 전송된 쿠키 정보 중에서 key가 "memberId"인 쿠키값을 찾고, `memberId`에 할당한다. `required=false`는 쿠키정보가 없는 비회원도 접근 가능하다. 
___
## 서버에서 쿠키 없애기
* 쿠키의 종료 날짜를 0으로 줘서 만료시킬 수 있다.
```java
    @PostMapping("/logout")
    public String logoutV1(HttpServletResponse response) {
        expiredCookie(response, "memberId");
        return "redirect:/";
    } 

    private void expiredCookie(HttpServletResponse response, String cookieName) {
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0);
        response.addCookie(cookie);
    }
```
___
# 2. Session으로 로그인

* 세션 관리 기능의 역할
  1. session 생성: 
  2. session 조회: 
  3. session 만료: 

## 2.1. Session Manager 직접 만들기
* (1) SessionManger 클래스 만들고 component로 등록한다.
```java
@Component
public class SessionManager {
    public static final String SESSION_COOKIE_NAME = "mySessionId";
    private Map<String, Object> sessionStore = new HashMap<>();

    public void createSession(Object value, HttpServletResponse response) {
        //create Session
        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);
        // create Cookie and addCookie
        Cookie cookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(cookie);
    }

    public Object getSession(HttpServletRequest request) {
        Cookie cookie = findCookie(request, SESSION_COOKIE_NAME);
        if(cookie == null){
            return null;
        }
        return sessionStore.get(cookie.getValue());
    }

    public void expire(HttpServletRequest request) {
        Cookie cookie = findCookie(request, SESSION_COOKIE_NAME);
        if(cookie != null){
            sessionStore.remove(cookie.getValue());
        }
    }

    private Cookie findCookie(HttpServletRequest request, String cookieName) {
        if(request.getCookies() == null){
            return null;
        }
        return Arrays.stream(request.getCookies())
                .filter(c -> c.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }
}
```

* (2) login 시 session 생성
```java
   @PostMapping("login")
    public String loginV2(@ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        
        // session Manager로 session 생성, member 정보 보관\
        sessionManager.createSession(loginMember, response);
        return "home";
    }
```
* (3) session 조회
```java
@GetMapping("/")
public String homeLoginV2(HttpServletRequest request, Model model) {
    Member member = (Member) sessionManager.getSession(request);

    if (member == null) {
        return "home";
    }

    model.addAttribute("member", member);
    return "loginHome";
}
```
* (4) session 만료
```java
 @PostMapping("/logout")
    public String logoutV2(HttpServletResponse response, HttpServletRequest request) {
        sessionManager.expire(request);
        return "redirect:/";
    }
```

___

## 2.2. Servlet: HttpSession을 이용한 로그인 처리
* SessionManager의 역할을 하는 객체를 서블릿에서는 HttpServlet 클래스를 통해 제공한다.

* (1) session 조회용 상수
```java
public interface SessionConst {
		String LOGIN_MEMBER = "loginMember";
}
```
* (2) session 생성, 조회, 만료
```java
   @PostMapping("login")
    public String loginV3(@ModelAttribute LoginForm form, BindingResult bindingResult,
                          HttpServletRequest request, HttpServletResponse response) {
        
        }
        // session 생성, member 정보 보관
        HttpSession session = request.getSession();
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);
        return "redirect:/";
    }

    @PostMapping("/logout")
    public String logoutV3(HttpServletRequest request) {
        HttpSession session = request.getSession(create: false);
        if (session != null) {
            session.invalidate();
        }
        return "redirect:/";
    }
```
* `request.getSession()`: session을 생성 또는 조회한다. `getSession(create: true)`가 default다.
  1. true: session이 있으면 기존 session을 반환한다. session이 없으면, 새로운 session을 생성해서 반환한다.
  2. false: session이 있으면 반환하고, 없으면 새로운 session을 생성하지 않고, null을 반환한다.
* `session.invalidate()`: session 제거

___

## 2.3 @SessionAttribute
* @SessionAttribute로 session 조회
 ```java
     @GetMapping("/")
    public String homeLoginV3Spring(@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false)Member loginMember,
                                    HttpServletRequest request, Model model) {

        if (loginMember == null) {
            return "home";
        }
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
 ```
 * `@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false)`: client에게 전달받은 session 중에서 key가 일치하는 것을 찾는다. `required=false`이므로, 일치하는 것이 없으면 null을 반환한다.

