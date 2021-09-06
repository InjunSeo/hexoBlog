---
title: [JPA] Value Type
date: 2021-09-04 14:42:39
tags:
---

# 1. 데이터 타입의 분류 
## JPA의 데이터 타입 분류
* 엔티티 타입
  * @Entity로 정의되는 객체
  * 데이터가 수정되어도 식별자로 계속 추적된다. 
  * ex., Member Entity의 name, memberId 값을 변경하면, 식별자로 인식 가능하다.
* 값 타입
  * java에서 int, String처럼 순전히 값으로 사용되는 기본 타입primitive type 또는 객체를 말한다.
  * 식별자가 따로 없기 때문에, 데이터가 수정되면 추적되지 않는다.
  * e.g., int a의 값을 변경하면, 완전히 다른 값으로 대체된다.

* 값 타입의 분류
  1. 기본값 타입: 
    * primitive type: int, double
    * Integer, Long
    * String
  2. embedded type, 복합 값 타입
  4. collection value type
 
# 2. 기본값 타입
# 3. embedded type


