---
title: "[spring MVC] @Controller"
date: 2021-07-15 19:29:15
tags:
---

# @Controller
* HTTP 요청, 응답함에 있어 @controller가 지원하는 annotation들을 살펴보자.
* 주요 파라미터 지원 목록
  1. Http header
  2. Http body
  3. Http API
* @Controller에서 사용 가능한 파라미터들: [docs.spring](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)
* 응답 값: [docs](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annreturn-types)

___

## HTTP 요청값 처리
* Http 요청 데이터를 조회하는 방법: Http 요청 메시리를 통해 클라이언트가 서버로 데이터를 전달하는 방법
  1. GET: query parameter
  2. POST: HTML form
  3. HTTP Message body에 데이터 담아서 요청: 주로 JSON

### @ReuqestParam
* 메서드 옆에 `@Requestparam` 추가해서 사용

```java
@Slf4j
@Controller
public class RequestParamController {
    
    @ResponseBody
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String name,
            @RequestParam("age") int age) {
        log.info("userName= {}, age= {}", name, age);
        return "Ok";
    }
```
변수를 HTTP parameter 이름과 같게 설정하면, `@RequestParam(name="x")` 생략 가능하다.
```java
    @ResponseBody
    @RequestMapping("/request-param-v3")
    public String requestParamV3(
            @RequestParam String username,
            @RequestParam int age) {
        log.info("username= {}, age= {}", username, age);
        return "ok";
    }
```
 `@RequestParam`도 생략 가능하다.
```java
    @ResponseBody
    @RequestMapping("/request-param-v4")
    public String requestParamV4(String username, int age) {
        log.info("username= {}, age= {}", username, age);
        return "ok";
    }
```
parameter 필수 여부 
```java
    @ResponseBody
    @RequestMapping("/request-param-required")
    public String requestParamRequired(
            @RequestParam(required = true) String username,
            @RequestParam(required = false) int age) {
//        int a = null;
//        Integer b = null;
        log.info("username= {}, age= {}", username, age);
        return "ok";
    }
```
기본값 적용
```java
    @ResponseBody
    @RequestMapping("/request-param-default")
    public String requestParamDefault(
            @RequestParam(required = true, defaultValue = "guest") String username,
            @RequestParam(required = false, defaultValue = "-1") int age) {
        log.info("username= {}, age= {}", username, age);
        return "ok";
    }
```
파라미터를 Map으로 조회 가능하다.
```java
 
    @ResponseBody
    @RequestMapping("/request-param-map")
    public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
        log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
        return "ok";
    }
```
### @ModelAttrubute
* 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어주기 위해 `@modelAttribute` 사용한다.
```java
    @ResponseBody
    @RequestMapping("/model-attribute-v1")
/*    public String modelAttributeV1(@RequestParam String username, @RequestParam int age) {
        HelloData helloData = new HelloData();
        helloData.setUsername(username);
        helloData.setAge(age);
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        log.info("helloDate={}", helloData);
        return "ok";*/
    public String modelAttributeV1(@ModelAttribute HelloData helloData) {
        log.info("username= {}, age= {}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }

    @ResponseBody
    @RequestMapping("/model-attribute-v2")
    public String modelAttributeV2(HelloData helloData) {
        log.info("username= {}, age= {}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }
}
```

### @RequestBody 
* HTTP message body에 데이터를 직접 담아서 요청할때 사용한다.데이터를 담을 때, 크게 두 가지 방식이 있다.
  1. 단순 메시지
  2. JSON

### @RequestBody:  (Case1) 단순 메시지

#### 서블릿
```java
@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest req, HttpServletResponse res) throws IOException {
        ServletInputStream inputStream = req.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        log.info("messageBody= {}", messageBody);
        res.getWriter().write("Ok");
    }
```
#### spring MVC 제공 parameter: :`InputStream` (reader), `OuterStream` (writer) 
```java

    @PostMapping("/request-body-string-v2")
    public void requestBodyStringV2(InputStream inputStream, Writer writer) throws IOException {
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        log.info("messageBody= {}", messageBody);
        writer.write("Ok");
    }
```
#### HttpEntity: HTTP header, body 정보를 편라하게 조회
* 응답에서도 HttpEntity 사용 가능
```java    

    @PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {
        String messageBody = httpEntity.getBody();
        log.info("messageBody= {}", messageBody);
        return new HttpEntity<>("Ok");
    }

```
#### @Requestbody: 메시지 바디 정보를 직접 조회
```java
    @ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody){
        log.info("messageBody={}", messageBody);
        return "Ok";
    }
}
```
### @RequestBody:  (Case2) JSON
#### Servlet 사용
```java
@Slf4j
@Controller
public class RequestBodyJsonController {
    private ObjectMapper objectMapper = new ObjectMapper();

    @PostMapping("/request-body-json-v1")
    public void requestBodyJsonV1(HttpServletRequest req, HttpServletResponse res) throws IOException {
        ServletInputStream inputStream = req.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody= {}", messageBody);
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        res.getWriter().write("ok");
    }
```
#### @RequestBody
```java
    @ResponseBody
    @PostMapping("/request-body-json-v2")
    public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {

        log.info("messageBody= {}", messageBody);
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "Ok";
    }
```
#### @RequestBody 에 직접 만든 객체를 지정할 수 있다
```java
    @ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData helloData) {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "Ok";
    }

    @ResponseBody
    @PostMapping("/request-body-json-v5")
    public HelloData requestBodyJsonV5(@RequestBody HelloData data) {
        log.info("username= {}, age= {}", data.getUsername(), data.getAge());
        return data;
    }
```
#### HttpEntity를 사용해서 객체로 데이터 저장 가능하다.
```java    
    @ResponseBody
    @PostMapping("/request-body-json-v4")
    public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) {
        HelloData data = httpEntity.getBody();
        log.info("username= {}, age= {}", data.getUsername(), data.getAge());
        return "Ok";
    }

   

}
```

* HttpEntity , @RequestBody 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
___

## HTTP 응답 처리
* 우리가 이전에 본 spring MVC 구조에서, Controller는 ModelAndView에 경로를 담아서 반환했다. 하지만, annotation 기반의 @controller에서는 다른 것들 --String, void 등도 가능하다. 이것이 어떻게 가능한지에 대해서는 다음 장에서 살펴보겠다.
* 여기서 우리는 스프링에서 응답 데이터를 만들고 반환하는 기능들에 대해서 알아보고자 한다.

```java
@Controller
public class SpringMemberListControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members")
    public ModelAndView process(){
        List<Member> members = memberRepository.findAll();
        ModelAndView modelAndView = new ModelAndView("members");
        modelAndView.addObject("members", members);
        return modelAndView;
    }
}
```
* 스프링 서버에서 응답 데이터를 만드는 방법
  1. static resource: 해당 파일(.html)을 변경 없이 그대로 서비스하는 것
  2. view template: 뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다
  3. HTTP message

### view로 전달
```java
@Controller
public class ResponseViewController {

    @RequestMapping("/response-view-v1")
    public ModelAndView responseViewV1() {
        ModelAndView modelAndView = new ModelAndView("response/hello.html")
                .addObject("data", "hello, view-V1");
        return modelAndView;
    }
}
```

```java

    @RequestMapping("/response-view-v2")
    public String responseViewV2(Model model) {
        model.addAttribute("data", "hello, view-v2");
        return "response/hello.html";
    }


```
* response/hello 로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다
### Http API
* view template을 거치지 않고, 직접 Http 메시지 바디에 JSON 같은 형식으로 데이터를 담아서 보내는 방법

```java
@Controller
//@RestController
    public class ResponseBodyController {
    @GetMapping("/response-body-string-v1")
    public void responseBodyV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");   
    }

    /**
    * HttpEntity, ResponseEntity(Http Status 추가)
    * @return
    */
    @GetMapping("/response-body-string-v2")
    public ResponseEntity<String> responseBodyV2() {
        return new ResponseEntity<>("ok", HttpStatus.OK);
    }
```

```java
    @ResponseBody
    @GetMapping("/response-body-string-v3")
    public String responseBodyV3() {
        return "ok";
    }
```
* @ResponseBody 를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할 수 있다

```java
    @GetMapping("/response-body-json-v1")
    public ResponseEntity<HelloData> responseBodyJsonV1() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);
        return new ResponseEntity<>(helloData, HttpStatus.OK);
    }

    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    @GetMapping("/response-body-json-v2")
        public HelloData responseBodyJsonV2() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);
        return helloData;
    }
}
```

# Reference
* 김영한(2021-03-21), Spring MVC 1편, Ch6 S13 ~ S14