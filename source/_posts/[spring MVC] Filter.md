---
title: "[spring MVC] Filter"
date: 2021-08-07 13:48:02
tags:
---

# 개요
* 로그인 여부에 따른 페이징 처리가 매우 제한적이게 된다. 글쓰기, 댓글달기, 결재, 관리자 페이지 등은 로그인 여부에 의존한다. 
* 많은 로직이 의존하는 부분을 '공통 관심 사항'이라고 한다. 
* '공통 관심 사항'은 스프링 AOP에서 처리할 수도 있지만, 웹 관련 공통 관심 사항은 servlet filter 또는 spring intersepter에서 처리하는 것이 좋다. 왜냐하면, 웹 관련 공통 관심 사항은 Http header 정보 또는 url 정보가 필요하고, HttpServletRequest가 그것들을 처리해주기 때문이다.

# filter and intersepter
```java
    public interface Filter{
        public default void init(FilterConfig filterconfig) throws ServletException{}
        
        public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

        public default void destroy() {}
    }
```
