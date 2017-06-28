---
layout:     post
title:      Fast-Campus Java Web Programming Camp 3일차
author:     kingbbode
tags:       spring
subtitle:   강의 정리
category:  study
outlink: 0
---

세번째 강의
===========

개발자 관련 단상
----------------

-	좋은 개발자란 어떤 개발자일까?
	-	사람을 중심에 두면서 기술을 사용하는 개발자?
	-	고민에 방향을 사람으로 향하면 그 속에 답이 있다..

한 내용
-------

### Template Engine

-	JSP
-	Freemaker
-	Mustache
-	Handlebars
-	Thymleaf

`isomorphic`은 계속 개발전할 것이지만, 아직은..!

### 응답 방식

-	forwarding
-	redirect

경우에 따른 forwarding 방식의 위험성을 주의!

### 객체지향적인 개발

객체에게 데이터가 아닌 메시지를 던져라!

ex)

```java
user.setName(userPojo.getName()) // X 데이터를 던지는 것은 객체지향적이 아닌 절차지향!

//객체지향적 개발
user.update(userPojo)

//User.class
void update(User user){
  this.name = user.name;
}
```

아래 참고!

[창천향로님의 객체지향적인 블랙잭 게임 구현](http://okky.kr/article/362491)

[OKKY에 fender님이 남겨주신 초보 개발자에게 권장하는 객체지향 모델링 공부 방법](http://okky.kr/article/358197)

수업후 느낀점
-------------

객체지향적인 개발에 대한 설명을 들으며, 많은 충격을 받았다. JAVA를 사용하면서 내가 객체지향적인 개발을 하지 않았다니!<br>그동안 객체지향에 대한 의구심과 개발에서의 모호함 등이 객체지향적인 개발에서 모두 설명이 되는 듯 하다. 객체지향을 의식하고 연습하며, 객체지향적인 개발을 하도록 노력해야겠다.
