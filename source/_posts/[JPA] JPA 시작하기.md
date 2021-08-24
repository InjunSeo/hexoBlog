---
title: "[JPA] Start JPA" 
date: 2021-08-24 15:29:24
tags:
	- JPA
categories:
	- JPA
	
---

# JPA 설정
```java
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```
# JPA 동작 확인
## 객체와 table 생성, 매핑하기

* 테이블 만들고 객체 만들기
```java
@Entity
public class Member {

    @Id
    private Long id;
    private String name;
    //Getter, Setter
}
```
## 간단 기능: 회원 저장, 찾기, 수정
```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        try {
            /** save member */
            Member member = new Member();
            member.setId(1L);
            member.setName("memberA");

            em.persist(member);
            
            /** find member */
            Member findMember = em.find(Member.class, 1L);
            System.out.println("findMember = " + findMember.getId() + ", " + findMember.getName());
            
            /** edit member */
            findMember.setName("seomoon");
            
            tx.commit;
        } catch (Exception e) {
            tx.rollback();
        }finally {
            em.close();
        }
        emf.close();
    }
}
```
* 앞서 만들어놓은 "hello"라는 이름의 설정 정보를 읽어와서 만든다.
* `EntityManagerFactory`는 하나만 생성해서 애플리케이션 전체에서 공유한다.
* `EntityManager`는 쓰레드 간 공유하지 않는다. 
* JPA의 모든 데이터 변경은 트랜잭션 안에서 실행한다.

## 

