---
title: "JPA란?"
categories:
  - JPA
tags:
  - [study, java]
toc: true
toc_sticky: true
sidebar: 
  nav: "docs"
---
## JPA(Java Persistenc API, Jakarta Persistence API)
- 스프링이 제공해주는 것이 아닌 자바에서 제공해주는 `ORM` 표준 기술이다.
- `Hibernate`, `Spring JPA`, `EcliplseLink` 등 과 같은 구현체가 있고 이것의 표준 인터페이스가 JPA이다.
- JPA 3.0 버전부터는 JCP에서 이클립스 재단으로 이관이 진행 되면서 Jakarta Persistence API로 이름이 바뀌었다.

## ORM(Object Relation Mapping)
- Object-relation mapping(객체 관계 매핑)
- 객체는 객체대로 설계한다.
- 관계형 데이터베이스는 관계형 데이터 베이스대로 설계한다.
- ORM 프레임워크가 중간에 매핑한다.
- 대중적인 언어에는 ORM기술이 존재(`python`, `java`, `Node.js`...)

## 동작과정
<img src="/assets/images/jpa-working-flow.png" alt="jpa-working-flow">
JPA는 애플리케이션과 JDBC 사이에서 동작한다.

## JPA의 장점
- 특정 데이터베이스에 종속되지 않는다.
  * 처음에 h2 driver로 개발하다가 중간에 oracle또는 MySQL로 DB를 변경해야한다고 해보자.  
  만약 JDBC를 이용해서 개발한다면 query 각각의 방언에 맞게 바꿔줘야하는 번거로움이 있을 것이다. 하지만 JPA를 사용한다면 설정파일에서 어떤 DB를 사용하는지만 말해주면 된다.
- SQL중심적인 개발에서 객체 중심으로 개발할 수 있다.
- 생산성이 향상된다.
- 유지보수하기 쉽다.
  * 만약 `Member`라는 엔티티에 `tel`이라는 속성이 추가된다고 해보자 우리가 만약 JDBC를 사용한다면 필드 수정에 큰 어려움을 가지게 될 것이다.  
  그러나 JPA를 사용한다면 우리는 단순히 필드만 추가해주면 된다. 나머지 SQL은 JPA가 처리해줄 것이다.
- 페더라임의 불일치 해결
  * JPA와 상속
  * JPA와 연관관계를 지원한다(1:N, N:1,N:M,1:1)
  * JPA와 객체 그래프 탐색
- 성능 최적화 기능
  * 1차 캐시와 동일성(identity)을 보장해준다.
  ```java
  String memberId = "100";
  Member m1 = jpa.find(Member.class, memberId); //SQL
  Member m2 = jpa.find(Member.class, memberId); //캐시
  System.out.println(m1 == m2) //true
  // SQL이 한번만 시행된다.
  ```
- 애노테이션을 이용한 매핑 설정을 한다. (xml파일로도 매핑 설정은 가능하다.)
- String, int, LocalDate 등 기본적인 타입에 대한 매핑을 지원해준다.
- 커스텀 타입 변환기를 지원해준다. (내가 만든 money타입을 db칼럼에 매핑이 가능하다.)
  

  * 트랜잭션을 지원하는 쓰기 로딩을 지연해준다.  

  `insert`
  ```java
  transaction.begin(); // [트랜잭션] 시작
  em.persist(memberA);
  em.persist(memberB);
  em.persist(memberC);
  //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
  //커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
  transaction.commit(); // [트랜잭션] 커밋
  ```

  `update`
  ```java
  transaction.begin(); // [트랜잭션] 시작
  changeMember(memberA); 
  deleteMember(memberB); 
  비즈니스_로직_수행(); //비즈니스 로직 수행 동안 DB 로우 락이 걸리지 않는다. 
  //커밋하는 순간 데이터베이스에 UPDATE, DELETE SQL을 보낸다.
  transaction.commit(); // [트랜잭션] 커밋
  ```

  * 지연 로딩과 즉시로딩  

  `지연 로딩:`객체가 실제 사용될 때 로딩된다.

  ```java
  Member member = memberDAO.find(memberId); // select * from member 
  Team team = member.getTeam();
  String teamName = team.getName(); // selct * from team
  ```

  `즉시 로딩:` JOIN SQL로 한번에 연관된 객체까지 미리 조회한다.
  ```java
  Member member = memberDAO.find(memberId);
  Team team = member.getTeam();
  String teamName = team.getName(); // select M.*, T.* from member join team
  ```
  