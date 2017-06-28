---
layout:     post
title:      [Fast-Campus] Java Web Programming Camp 4일차
author:     kingbbode
tags:       spring
subtitle:   강의 정리
category:  study
outlink: 0
---

네번째 강의
===========

배움 관련 이야기
----------------

-	가장 좋은 배움은 나의 색깔, 나의 속도로 꾸준히 가는 것이 가장 빠른 성장으로 가는 길
-	주변에 휘둘리지말고 가보는 것이 좋다

**책 추천**

`이너게임` - 배움에 대하여 다른 관점을 느낄 수 있는 책

`몰입의 즐거움` - 행복한 삶을 살기 위해 몰입하는 것이 얼마나 중요한 것인지에 다루고 있는 책

*3년차 전까지는,, 개발 서적이나 열심히..! 그 후 여유가 되면 위 책을 보자*

배운 것
-------

### 객체지향

-	객체는 상태 값뿐만 아니라 행위도 가진다.
-	상태 데이터를 가지고 있는 객체에게 무슨 일을 시킬 수 있을까를 고민해야 한다
-	하나의 메서드는 하나의 행위만 해야한다.
-	한 줄 짜리 메서드라도 역할을 명시하는 것이 중요하다.

### HTTP

-	무상태(stateless) 프로토콜
-	요청,응답 간의 관계를 위해 Session, Cookie를 Header에 실어 사용한다.

### JPA

#### Lob Annotation

`@Lob`

대용량 데이터 타입을 선언한다.

ex)

```java
@Lob
int idx // bigint

@Lob
String contents // clob
```

#### ORM 객체와 DB 필드명 관계

DB의 underscore 방식과 자바의 CamelCase 방식이 자동 양방향 컨버팅

ex)

데이터베이스 : `abc_abc_abc`<br> 자바 : `abcAbcAbc`

### else 사용을 자제

ex)

```java
if(a>1){
  if(b>3){
    return false;
  }else{
    return true;
  }
}else{
  return true;
}
```

보다는 아래 방식이 깔끔

```java
if(a>1 && b>3){
  return true
}

return false;
```

느낀점
------

코딩, 스타일과 규칙은 사람마다 다르지만, 더 깔끔하고 가독하기 쉬운 스타일이 분명히 있다. 그런 스타일을 작성하기 위하여 끊임없이 노력해야겠다고 느꼈다.<br> Hibernate만 사용하다보니 JPA의 기본을 모르는 것이 많았다. ORM을 이해하기 위하여 최대한 빨리 ORM 책을 꼭 정독을 해야겠다.
