---
title:  "[spring MVC] Exception"
date: 2021-08-14 13:29:43
tags:
---

# HandlerExceptionResolver
```java
    public interface HandlerExceptionResolver { 
        ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex);
    }
```
# spring 제공 ExceptionResolver
## ResponseStatusExceptionResolver
## DefaultHandlerExceptionResolver
## ExceptionHandlerExceptionResolver