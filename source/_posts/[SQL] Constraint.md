---
layout: posts
title: "[SQL] Constraint"
date: 2021-08-06 13:30:45
tags:
	- sql
categories:
	- sql
---
# 1. Primary key
## 1.1. 정의
x는 primary key에 입력되는 값이다 iff
(1) x는 중복될 수 없다.
(2) x에 null이 입력될 수 없다.

## 1.2. primary key 지정하기
### (1) 하나일 때
### (2) 두 개 이상일때
* prodCode와 prodId가 기본 키라고 가정하자.
```sql
Create table prodTbl(
    prodCode char(3) not null,
    prodId char(4) not null,
    rpodDate datetime not null,
    constraint primary key(prodCode, prodId)
)
```

* 테이블을 create할때 제약조건을 설정하지 못했다면, 
```sql
Alter table usertbl add constraint primary key(prodCode, prodId);
```

## 1.3. 이름 붙이기
primary key의 이름을 붙일 수 있다.
```sql
create table usertbl(
	userId char(8) Not null,
    name VARCHAR(10) not null,
    addr char(9),
    mDate date,
    constraint primary key PK_userTBL_userId (userId)
);
```
___
# 2. Foreign key 제약조건
## 2.1. 정의
* 외래 키를 정의하는 테이블을 '외래 키 테이블'이라고 한다. 외래 키에 의해서 참조되는 테이블을 '기준 테이블'이라고 한다.
* 기준 테이블의 열은 primary key 또는 uqique 조건으로 설정되어 있어야 한다.

## 2.2. 외래 키 설정하기
* foreign key도 테이블 생성 시에 설정할수 있고 생성 이후에 설정할 수 있다.
```sql
create table buytbl(
	num int auto_increment not null primary key,
    userId char(8) not null,
    prodName char(20) not null,
    groupName char(10),
    price int not null,
    amount smallint,
    constraint FK_userTBL_buyTBL foreign key(userId) references usertbl(userId)
);
```
## 2.3. Foreign key Option
* 목적: 기준 테이블의 데이터가 변경되었을 때, 외래 키 테이블도 자동으로 적용되도록 설정해 주기 위한 기능이다.
* 두 가지 상황이 가능하다. 
  1. 기준 테이블의 데이터가 변경되면, 외래 키 테이블의 값도 변경된다.
  2. 기준 테이블의 데이터가 삭제되면, 외래 키 테이블의 값도 삭제(또는 null)된다.

### On Update cascade
* 기준 테이블의 데이터가 변경되면, 외래 키 테이블의 값도 변경해주기 위한 옵션이다.
* 위에서 만든 테이블 buyTBL에 `On Update CASACADE`를 추가해보자.
```sql
alter table buytbl drop foreign key FK_userTBL_buyTBL;
alter table buytbl
	add constraint FK_userTBL_buyTBL
		foreign key(userId) 
        references usertbl(userId)
        On update cascade;
```

### On Delete cascade
* 두 가지 기능으로 세분화 가능하다.
  1. 기준 테이블의 데이터가 삭제되면, 외래 키 테이블의 값도 삭제한다.
  2. 기준 테이블의 데이터가 삭제되면, 외래 키 테이블의 해당 값에 null을 준다. 

* Case1: `On Delete Cascade`
```sql
alter table buytbl drop foreign key FK_userTBL_buyTBL;
alter table buytbl
	add constraint FK_userTBL_buyTBL
		foreign key(userId) 
        references usertbl(userId)
        on update cascade
        on delete cascade;
```
* Case2: `on delete set null`

```sql
alter table buytbl drop foreign key FK_userTBL_buyTBL;
alter table buytbl
	add constraint FK_userTBL_buyTBL
		foreign key(userId) 
        references usertbl(userId)
        on update cascade
        on delete set null;
```
그런데 buyTBL을 생성할 때, userId는 not null로 설정했기 때문에 `on delete set null`이 먹지 않는다. 
___

# 3. Unique
## Definition
variable x가 unique 제약 조건에 설정된다. iff 
(1) x에는 유일한 값value만이 할당될 수 있으며,
(2) x에는 null이 들어갈 수 있다.
## uqique 제약 조건 설정하기
```sql
create table userTBL(
	userId char(15) Not null,
    email char(30) null,
	constraint primary key(userId),
    constraint AK_email unique(email)
);
```
___
# 4. Check 제약조건
## 정의
* check 제약조건은 입력되는 데이터를 점검하는 기능을 한다.
```sql
create table userTBL(
	userId char(15) Not null,
    userName VARCHAR(10),
    birthYear int check (birthYear >= 1900 And birthYear <= 2021),
    mobile1 char(3),
    mobile2 char(8),
	constraint primary key(userId),
    constraint CK_username check (userName is not null)
);

alter table usertbl
	add constraint CK_mobile1
    check (mobile1 in ('010', '02'));
```
___
# 5. Default
## 정의
* 값을 입력하지 않았을 때, 기본 값을 정의해주는 기능이다.
```sql
create table userTBL(
	userId char(15) Not null primary key,
    userName VARCHAR(10),
    birthYear int not null default -1,
	addr char(9) not null default 'seoul',
    email char(30) null,
    mobile1 char(3),
    mobile2 char(8),
    height smallint,
    mDate date,
    constraint AK_email unique(email)
);
alter table userTBL
	alter column height set default 170;
```
* `insert into`로 데이터를 넣어줄 때, `null`이 아니라 `default`를 입력해줘야 한다.