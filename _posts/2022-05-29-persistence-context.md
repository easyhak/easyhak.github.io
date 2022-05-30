---
title: "JPA 영속성 컨텍스트(Persistence Context), 플러쉬(flush)"
categories:
  - JPA
tags:
  - [study, java]
toc: true
toc_sticky: true
sidebar: 
  nav: "docs"
---

# 영속성 컨텍스트(Persistence Context)?
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
1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭
션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공해준다.

## 트랜잭션을 지원하는 쓰기 지연
```java
transaction.begin(); // [트랜잭션] 시작
em.persist(memberA);
em.persist(memberB);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
//커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```
![persistence-context](/assets/images/persistence-context-delay1.png)
*em.persist(memberA);*
{: .text-center}
![persistence-context](/assets/images/persistence-context-delay2.png)
*em.persist(memberB);*
{: .text-center}
![persistence-context](/assets/images/persistence-context-delay3.png)
*transaction.commit();*
{: .text-center}
위와 같은 방식으로 트랜잭션이 커밋 될 때 쿼리가 한번에 나가게 되는 것이다.

## 변경 감지(Dirty Chceking)
```java
transaction.begin(); // [트랜잭션] 시작
// 영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");
// 영속 엔티티 데이터 수정
memberA.setUsername("hi");
memberA.setAge(10);
//em.persist(member) 이런식으로 해야되나??
transaction.commit(); // [트랜잭션] 커밋
```
![dirty-checking](/assets/images/persistence-context-dirtychecking.png)
진짜 이 부분을 배울 때는 좀 충격(?)적이였다.  
{: .text-center}
entity를 수정할 때 update나 다시 persist를 해야하나 생각했지만 그럴 필요가 없다.   
영속성 컨텍스트에는 스냅샷이라는 것이 있어서 변경이 되면 서로 비교하여      
transacion이 commit될 때 자동으로 update하게 해준다.
따라서 update함수를 만들어주거나 persist를 다시 해줄 필요가 없다.

## 엔티티 삭제(remove)

```java
Member memberA = em.find(Member.class, “memberA");
em.remove(memberA); //엔티티 삭제
```

삭제는 이런식으로 하면 된다. 엄청 쉽다.
{: .text-center}
# 플러쉬(flush)?
영속성 컨텍스트에 있는 내용을 데이터베이스에서 반영하는 것을 말한다.   
보통은 transaction commit이 일어날 때 flush가 동작하게 된다.   
이 때 update, delete, insert 쿼리가 나가게 된다.    
- 플러쉬가 되었다고 해서 영속성 컨텍스트가 비워지는 것은 아니다.   
- 영속성 컨텍스트의 변경내용을 DB에 동기화 하는 것이다.
- 트랜잭션이라는 작업 단위가 중요하다. 커밋 직전에 동기화 하는 것이다.

## 영속성 컨텍스트를 플러쉬(flush)하는 방법
- em.flush() - 직접 호출
- 트랜잭션 커밋 - 자동 호출
- JPQL 쿼리 실행 - 자동 호출

## 플러쉬 모드 옵션
- FlushModeType.AUTO 커밋이나 쿼리를 실행할 시 플러쉬(default)
- FlushModeType.COMMIT 커밋할 때만 플러쉬

# 준영속 상태
영속 상태의 엔티티가 영속성 컨텍스트에서 분리 된 것   
이러면 영속성 컨텍스트가 제공하는 기능을 사용하지 못하게 된다.
## 준영속 상태로 만드는 방법
- __em.detach(entity)__   
특정 엔티티만 준영속 상태로 전환
- __em.clear()__   
영속성 컨텍스트를 완전히 초기화한다.
- __em.close__   
영속성 컨텍스트 종료한다.
