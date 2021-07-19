---
title: "[spring MVC] @RequestMapping" 
tags:
---
# 기본 요청
* @RequestMapping
```java
@Slf4j
@RestController
public class MappingController {

    @RequestMapping({"/hello-basic", "/hello"})
    public String helloBasic() {
        log.info("hello-Basic");
        return "ok";
    }
```
 * "/hello-basic" 또는 "/hello" url 호출 시, 해당 메서드가 실행되도록 매핑해준다.
 * Http 메서드를 모두 허용해준다. GET, POST, PUT 등

 # HTTP 메서드 매핑
 * 특정 HTTP 메서드 요청만 허용해준다.
```java
    @GetMapping("/mapping-get")
    public String mappingGet() {
    log.info("mappingGet");
        return "ok";
    }
```
* @PostMapping, @PutMapping 등

# 경로 변수 사용
* @PathVariable
```java
  @GetMapping("/mapping/{userId}")
  public String mappingPath(@PathVariable("userId") String data) {
  log.info("mappingPath userId={}", data);
      return "ok";
  }
```
* 특정 parameter 조건 매핑

  * params="mode=debug",

  * pramas = "!mode"

```java
  @GetMapping(value = "/mapping-param", params = "mode=debug")
  public String mappingParam() {
  log.info("mappingParam");
      return "ok";
  }
  /* params에 "mode=debug" 포함되어야 요청을 받아들인다.
```
* 그 밖의 조건들
  1. 특정 헤더 조건 매핑: header=".."
  2. 미디어타입 조건 매핑: consumes="..", produces=".."

# Reference
* 김영한(2021-03-21), Spring MVC 1편, Ch6 S13 ~ S14

 