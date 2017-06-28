---
layout:     post
title:      [Fast-Campus] Java Web Programming Camp 2일차
author:     kingbbode
tags:       spring
subtitle:   강의 정리
category:  study
outlink: 0
---

두번째 강의
===========

학습 방법
---------

`코딩을 지탱하는 기술` 책 추천!

-	필요한 부분부터 흡수!
	-	처음부터 완독보다는, 필요한 부분을 먼저 학습한다.
-	대략적인 부분을 잡아서 조금씩 상세화!
	-	큰 그림을 그릴 줄 알아야 한다.
-	끝에서부터 차례대로 베껴간다!
	-	잘 이해가 가지않으면 반복적으로 베껴서라도 학습한다.

<br>

한 내용
-------

### HTTP

-	GET : 서버의 데이터를 요청할 떄
-	POST : 서버의 상태를 변경할 때

### MVC

-	MVC를 이해하자!
	- Model
	- View
	- Controller

### Java Bean 규약

Setter, Getter는 꼭 필요한 것은 아니다.
Reflection을 통해 getter 없이도 private 접근이 가능하다!

### Controller Mapping

- @GetMaping
	- 스프링 4 이상부터 지원하는 어노테이션으로 RequestMapping Method Type을 Get으로 가지고 있어서 편함!
- @PostMaping
	- 스프링 4 이상부터 지원하는 어노테이션으로 RequestMapping Method Type을 POST로 가지고 있어서 편함!
	- Key, Value를 값은 일반 매개변수로도 맵핑이 가능하지만, Json 같은 데이터 가공이 필요한 데이터는 @RequestBody를 통해 Mapping이 가능하다.

수업 후 느낀 점
---------------

이미 편리하게 작성된 프레임워크의 기능들과 여러 Annotation들을 사용해왔다. 그렇지만 이것들이 정작 어떤 원리와 구조로 돌아가는지 모르고 있는 것이 너무 많았다.

저번 시간부터 느끼는 거지만 각종 툴을 사용하고 편리한 것들을 사용만 해왔기때문에 기본이 되는 것들을 모르는게 너무 많다!

앞으로 더 깊이 알아가보자!