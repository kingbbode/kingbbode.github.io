---
layout: post
title: 도메인 드리븐 정기 세미나 - TDD
categories: [seminar]
tags: [seminar]
fullview: false
comments: true
published: true
outlink: 0
---

세미나 정리 및 후기.

<br />

# **Domain Driven TDD**
>2016년 7월 23일

### 재미있게 개발하기

* 왜 TDD?
    - 다이나믹 개발-검증-개발-검증
    - 마음 놓고 리팩토링
    - 비지니스 로직에 집중

* 왜 Spring Boot?
    - 왠만한건 @Enable ~~
    - 설정보다 비즈니스 로직에 집중
    - Spring-data 시리즈와 연동 편리

* 왜 JPA?
    - CRUD 편하게?
    - 객체간의 관게만 신경쓰면 대부분 해결
    - 개발시에 테이블 마음대로 부시기
    - 비지니스 로직에 집중
    - 리팩토링에 유리

* 카카오..
    - 코드 리펙토링에 자유롭,,
    - 설계하고 개발하면서 디자인 패턴 적용하는 것보다 개발하고 리펙토링하면서 적용하는게 더 효율.

* 테스트 코드 중요
    - 에자일 사례..

### 개발팁

* Gradle -> Maven으로 바꿨음.. (Gradle offline mode이면 빠르긴한데 그렇지않은 경우 Maven이 월등히 빠름)
* Setter 안쓰고 생성자를 사용.(객체적인 접근)
* findAll(pageable)
* JPA 장점으로 @PrePersist, @PreUpdate를 통해 테스트, 리펙토링 편함. 
* Jenkins
    - Cobertura 등 사용
    - 셋팅은 디폴트에 커버리지만 80으로 올림.
* ModelMapper로 데이터 Translation(왜?)
        
### TDD

* Git :  https://github.com/changhwa/tdd-example
* 패턴
    - Given
    - When
    - Then
* Helper 사용
