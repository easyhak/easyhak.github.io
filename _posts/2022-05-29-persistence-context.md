---
title: "JPA 영속성 컨텍스트(Persistence Context)"
categories:
  - JPA
tags:
  - [study, java]
toc: true
toc_sticky: true
sidebar: 
  nav: "docs"
---

# 영속성 컨텍스트란(Persistence Context)?
영속성 컨텍스트란 **엔티티를 영구 저장** 하는 환경 이라는 뜻이다.
애플리케이션이 데이터베이스에서 꺼내온 객체를 보관하는 역할을 한다. 영속성 컨텍스트는 엔티티 매니저( Entity Manager )를 통해 엔티티를 조회하거나 저장할때 엔티티를 보관하고 관리한다.
![persistence-context](/assets/images/persistence-context-working.png)

영속성 컨텍스트를 통해 벌어지는 일들을 정리해 보았다.

## 쿼리 실행 시점
```java
Member member = new Meber();
member.setName("hi");
em.persist(member); 
logger.info("EntityManager 호출함");
// member 영속화
tx.commit(); 
logger.info("EntityTransaction.commit 호출함");
```
console 출력값
```sql
INFO ~~~~~ - EntityManager 호출함
DEBUG org.hibernate.SQL - insert into member(id, name) values(?, ?)
INFO ~~~~~ - EntityTransaction.commit 호출함
```
나도 그랬고 많은 사람들이 JPA를 처음 접할 때 EntitiyManager가    
호출 될 때 쿼리문이 실행될 것이라고 생각한다. 하지만 그렇지 않다.   
   
특별히 flush나 JPQL등을 이용한 호출을 하지 않는    
위와 같은 상황에서는 transaction이 커밋되고 나서야 
SQL 쿼리문이 나가게 된다.    
    
위와 같은 일이 가능한 이유는 EntityManager 단위로 영속 컨텍스트를 관리하기 때문이다.   

그래서 커밋 시점에 영속 컨텍스트의 변경 내용을 DB에 반영하게 되는 것이다(SQL쿼리 발생).
![persistence-context](/assets/images/persistence-context.png)

## 엔타티 조회, 1차 캐쉬
```java
Member member = new Member(); 
member.setUsername("회원1");
// 엔티티를 영속 
em.persist(member);
// 1차 캐쉬에서 조회
Member findMember = em.find(Member.class, member.getId());
```
![persistence-context](/assets/images/persistence-context-cache.png)

1. member1을 1차 캐쉬에 넣는다. 
2. 아직은 쿼리 날라가지 않는다.
3. find를 할 때 1차 캐쉬에 있는 member를 찾아온다.
4. 따라서 select 쿼리가 날라가지 않고 insert 쿼리만 날아가게 된다.

## 동일성 보장
```java
Member a = em.find(Member.class, "member1"); 
Member b = em.find(Member.class, "member1");
System.out.println(a == b); //동일성 비교 true
```