---
title: "[spring MVC] Validation"
date: 2021-07-21 13:59:46
tags:
	- spring
categories:
	- spring
	- spring MVC
---

# Validation
* user가 회원가입이나 상품 등록을 할 때, 적절하지 않는 데이터를 입력하는 것을 막아야 한다.
  1. 필수 입력 정보에 Null일때
  2. typeMismatch
  3. 비즈니스 요구사항에 맞지 않을 때: 최소 금액 
* 검증의 종류
  1. BindingResult
  2. validator
  3. @valid

# Map

* item 클래스
```java
@Data
public class Item {
    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;
```
* Controller에서 직접 errors를 만든다.
```java 
 public String addItemV7(Item item, RedirectAttributes redirectAttributes) {
        Map<String, String> errors = new HashMap<>();
        //검증 로직
        log.info("itemName= {}", item.getItemName());

        if(item.getItemName() == null){
            errors.put("itemName", "상품 이름은 필수입니다.");
        }

        //검증 실패시 
        if (!errors.isEmpty()) {
            log.info("errors = {}", errors);
            return "basic/addForm";
        }

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/basic/items/{itemId}";
```
> 검증에 실패하면 errors에 error message를 담아서, 반환한다.
> 그런데 typeMismatch 시, validation.BindException에 따라 위 컨트롤러로 오기도 전에 400 에러 페이지를 응답한다.

# BindingResult
```java
    @PostMapping("/add")
    public String addItemV8(Item item, BindingResult bindingResult,
                            RedirectAttributes redirectAttributes, Model model) {
        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("item", "itemName", "item Name"));
        }
        if (item.getPrice() == null || item.getPrice() < 1000) {
            bindingResult.addError(new FieldError("item", "itemPrice", "가격은 최소 1000원"));
        }
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9999개입니다."));
        }
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                bindingResult.addError(new ObjectError("item", "총액(가격*수량)의 합은 10000원 이상이어야 합니다."));
            }
        }

        if (bindingResult.hasErrors()) {
            log.info("errors= {}", bindingResult);
            return "basic/addForm";
        }
        log.info("itemName= {}", item.getItemName());

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/basic/items/{itemId}";
    }
```
> error는 두 가지인데, fieldError와 ObjectError이다. 

* 오류 메시지를 보여줄 때, 사용자가 입력한 데이터는 사라진다.
* 입력해야 할 코드가 많다.

# @Valid
* class
```java
@Data
public class User {
    private Integer id;

    @NotEmpty
    private String account;

    @NotEmpty
    private String password;

}
```
* 컨트롤러에서 @Valid
```java
@Slf4j
@Controller
public class LoginController {

    @PostMapping("/user/login")
    public ResponseEntity<String> validlogin(@RequestBody @Valid User user, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            List<ObjectError> allErrors = bindingResult.getAllErrors();
            for (ObjectError error : allErrors) {
                System.out.println(error.getDefaultMessage());
            }
        }
        log.info("[User] id= {}, account= {}, pw= {}", user.getId(), user.getAccount(), user.getPassword());
        return new ResponseEntity<>("Ok", HttpStatus.OK);
    }
}
```
> 검증 실패 시, error message 출력한다.

# Validated
* 원하는 속성만 유효성 검사를 하고자 할때
```java
public class ValidationGroups {
    public interface group1 {
    };

    public interface group2 {

    }
}
```
```java
@Data
public class User {
    private Integer id;

    @NotEmpty(groups = {ValidationGroups.group1.class})
    private String account;

    @NotEmpty(groups = {ValidationGroups.group2.class})
    private String password;

}
```

* @Validated(..) 괄호 안에 설정을 추가한다.
```java
    @PostMapping("/user/login2")
    public ResponseEntity<String> validatedLogin(@RequestBody @Validated(ValidationGroups.group1.class)
                                                 User user, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            List<ObjectError> allErrors = bindingResult.getAllErrors();
            for (ObjectError error : allErrors) {
                System.out.println(error.getDefaultMessage());
            }
        }
        log.info("[User] id= {}, account= {}, pw= {}", user.getId(), user.getAccount(), user.getPassword());
        return new ResponseEntity<>("Ok", HttpStatus.OK);
    }
```
> group1에 해당하는 속성만 검증한다. 위 경우, account가 group1에 속하기 때문에, account만을 검증한다. 다른 속성들에 null이 들어가도, error message가 출력되지 않는다.
