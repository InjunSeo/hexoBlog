---
title:  "[spring MVC] Exception"
date: 2021-08-14 13:29:43
tags:
---
# Introduction
* 에러가 났을 때, Web application에서 그 에러를 표시해 주 수 있는 방법이 많지 않다. e.g., error page
## Exception 처리의 두 방법
* ControllerAdvice: Global 예외처리
* ExceptionHandler: 특정 controller의 예외처리
___

# ControllerAdvice
## web application의 error 표시

```java
    public class User {
        private String name;
        private int age;
}
```
controller를 작성한다.
```java
@RestController
@RequestMapping("/api/user")
public class ApiController {
    @GetMapping("")
    public User get(@RequestParam(required = false) String name,
                    @RequestParam(required = false) Integer age) {
        User user = new User();
        user.setName(name);
        user.setAge(age);

        int a = age + 10;

        return user;
    }
```

age에 null을 주고 보내면, 우리는 웹 화면에서 에러 페이지를 보게 된다. 다음과 같은 정도의 에러 메시지를 표시한다.
 "status": 500,
    "error": "Internal Server Error"
하지만, 터미널(intelliJ)에서는 더 자세한 정보를 볼 수 있다.
`NullPointerException: null`

## ControllerAdvice
```java
@RestControllerAdvice
public class GlobalControllerAdvice {
    //내가 잡고자하는 method
    @ExceptionHandler(value = Exception.class)
    public ResponseEntity exception(Exception e) {
        System.out.println("GlobalControllerAdvice.exception");
        System.out.println("error Message: " + e.getLocalizedMessage());

        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("");
    }
}
```
* `System.out.println(e.getClass().getName());`: 어디서 잘못된 것인가? 다음이 출력된다. `org.springframework.web.bind.MethodArgumentNotValidException`

* 이제, `MethodArgumentNotValidException`만 따로 잡을 수 있는 기능을 만들고 싶다.
```java
@RestControllerAdvice
public class GlobalControllerAdvice {
    
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public ResponseEntity MethodArgumentNotValidException(MethodArgumentNotValidException e) {

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(e.getMessage());
    }

}

```
* ControllerAdvice의 옵션 기능을 통해, 특정 패기지 하위의 클래스들에 대한 예외처리를 전담하게 만들 수 있다.
* 그렇다면, 특정 controller만을 위한 예외 처리는 어떻게 해야 하는가?

## Exceptionhandler
* ControllerAdvice 안에 만든 `@ExceptionHandler`를 따로 떼어서, ApiController에 붙여넣으면 된다.

```java
@RestController
@RequestMapping("/api/user")
public class ApiController {
    
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public ResponseEntity MethodArgumentNotValidException(MethodArgumentNotValidException e) {

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(e.getMessage());
    }
}

```
### 우선순위
* 동일한 exception 처리 메서드 @ExceptionHandler가 ControllerAdvice와 특정 controller에 모두 있다면 우선순위가 어떻게 되는가? 
* 특정 controller의 @ExceptionHandler가 exception을 잡게 된다. 

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