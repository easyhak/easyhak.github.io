---
title: "JPA 엔티티 매핑(Entity Mapping)"
categories:
  - JPA
tags:
  - [study, java]
toc: true
toc_sticky: true
sidebar: 
  nav: "docs"
---
JPA를 사용하면서 가장 중요한 작업 중 하나는 엔티티와 DB테이블을 정확히 매핑해주는 것이다.   
엔티티는 쉽게 이야기하면 DB테이블에 대응되는 하나의 클래스이다.   
JPA에서는 다양한 매핑 어노테이션들을 지원해주는데 그것들을 잘 숙지해줘야 한다.
# 객체와 테이블 매핑
## @Entity
@Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다.   
JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수이다.  

<span style="font-size:1.1em">
**주의**
</span>

- 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)로 만들어야 한다.
- final 클래스, enum, interface, inner 클래스 사용하지 말아야한다.
- 저장할 필드에 final 사용하지 말아야한다.   

<span style="font-size:1.1em">
**속성**
</span>

- name: JPA에서 사용할 엔티티 이름 지정, 지정하지 않으면 클래스 명을 사용, 가급적이면 기본값을 사용하도록 한다.

## @Table
엔티티와 매핑할 테이블을 지정한다.   
애노테이션을 생략하면 클래스 이름과 동일한 이름에 매핑한다.    

<span style="font-size:1.1em">
**속성**
</span>

- name: 테이블 이름, 생략하면 클래스 이름과 동일
- catalog: 카탈로그 이름 ex) MySQL, Oracle 등 DB이름
- schema: 스키마 이름 ex) Oracle Schema 이름
- uniqueConstraints: DDL 생성 시에 유니크 제약 조건 생성

## 데이터스키마 자동 생성
- DDL을 애플리케이션 실행 시점에 자동 생성
-  테이블 중심 -> 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한
DDL 생성
- 이렇게 생성된 DDL은 개발 장비에서만 사용
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬
은 후 사용   
    
<span style="font-size:1.1em">
**옵션**
</span>
- create: 기존테이블 삭제 후 다시 생성 (DROP + CREATE)
- create-drop: create와 같으나 종료시점에 테이블 DROP
- update: 변경분만 반영(운영DB에는 사용하면 안됨)
- validate: 엔티티와 테이블이 정상 매핑되었는지만 확인
- none: 사용하지 않음
<span style="font-size:1.1em">
**주의**
</span>
- 운영 장비에는 절대 create, create-drop, update 사용하면
안된다.
- 개발 초기 단계는 create 또는 update 
- 테스트 서버는 update 또는 validate 
- 스테이징과 운영 서버는 validate 또는 none

# 필드와 컬럼 매핑

## 매핑 애노테이션
### @Column(칼럼 매핑)
<span style="font-size:1.1em">
**속성**
</span> 
- name: 필드와 매핑할 테이블의 컬럼 이름 / default: 객체의 필드 이름
- insertable, updatable: 등록, 변경 가능 여부 / default: TRUE
- nullable(DDL): null 값의 허용 여부를 설정한다. false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다.
- unique(DDL): @Table의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다.
- columnDefinition(DDL): 데이터베이스 컬럼 정보를 직접 줄 수 있다. ex) varchar(100) default ‘EMPTY' 
- length(DDL): 문자 길이 제약조건, String 타입에만 사용한다. // default: 255
- precision, scale(DDL): BigDecimal 타입에서 사용한다(BigInteger도 사용할 수 있다). precision은 소수점을 포함한 전체 자 릿수를, scale은 소수의 자릿수
다. 참고로 double, float 타입에는 적용되지 않는다. 아주 큰 숫자나
정 밀한 소수를 다루어야 할 때만 사용한다.    
 // default: precision=19, scale=2

### @Temporal(날짜 타입 매핑)
- java.util.Date, java.util.Calander 매핑
- 자바 8 시간/날짜 타입 등장 이후로는 거의 쓰지 않는다.

### @Enumerated(enum 타입 매핑)
<span style="font-size:1.1em">
**속성**
</span> 
- value(default: EnumType.ORDINAL)
  - EnumType.ORDINAL: enum 순서를 데이터베이스에 저장
  - EnumType.STRING: enum 이름을 데이터베이스에 저장

__주의:__
ORDINAL을 사용하지 않도록 한다. 중간에 값이 추가가 되면 수정하기 어려워진다.

### @Lob(BLOB, CLOB 매핑)
- @Lob에는 지정할 수 있는 속성이 없다.
- 매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑한다.
  - CLOB: String, char[], java.sql.CLOB
  - BLOB: byte[], java.sql. BLOB 

### @Transient(특정 필드를 컬럼에 매핑하지 않음)
- 필드 매핑을 하지 않는다.
- 데이터베이스에 저장, 조회를 하지 않는다.
- 주로 메모리상에서만 임시로 어떤 값을 보관 할 때 사용한다.

## 기본 키 매핑
```java
@Id @GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```
### @Id
- 기본키 식별자로 매핑할 때 사용한다.

### @GeneratedValue
**IDENTITY**
```java
@Id 
@GeneratedValue(strategy = GenerationType.IDENTITY) 
private Long id; 
```
  - 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용한다. (예: MySQL의 AUTO_ INCREMENT)
  - JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행한다.
  - AUTO_ INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있다.
  - IDENTITY 전략은 em.persist() 시점에 즉시 INSERT SQL 실행하고 DB에서 식별자를 조회한다.
    
**SEQUENCE**
- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트
- 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용
- @SequenceGenerator 필요
```java
@Entity 
@SequenceGenerator( 
name = “MEMBER_SEQ_GENERATOR", 
sequenceName = “MEMBER_SEQ", //매핑할 데이터베이스 시퀀스 이름
initialValue = 1, allocationSize = 1) 
public class Member { 
  @Id 
  @GeneratedValue(strategy = GenerationType.SEQUENCE, 
  generator = "MEMBER_SEQ_GENERATOR") 
  private Long id; 
```
     
**TABLE**
- 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략이다
- 장점: 모든 데이터베이스에 적용 가능하다.
- 단점: 성능

```sql
create table MY_SEQUENCES ( 
  sequence_name varchar(255) not null, 
  next_val bigint, 
  primary key ( sequence_name ) 
)
```
    
```java
@Entity 
@TableGenerator( 
  name = "MEMBER_SEQ_GENERATOR", 
  table = "MY_SEQUENCES", 
  pkColumnValue = “MEMBER_SEQ", allocationSize = 1) 
  public class Member { 
  @Id 
  @GeneratedValue(strategy = GenerationType.TABLE, 
  generator = "MEMBER_SEQ_GENERATOR") 
  private Long id; 
```


**AUTO**
- 방언에 따라 자동 지정, 기본값

