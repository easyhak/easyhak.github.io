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

## @Table

# 데이터스키마 자동 생성
# 필드와 컬럼 매핑
# 기본 키 매핑
