---
layout: post
title: W3C HTML5 Conference 2016
categories: [seminar]
tags: [seminar, javascript]
fullview: false
comments: true
published: true
outlink: 0
description : 조금 밖에 못들은.. W3C HTML5 Conference 2016.. 
---

Session 4. Angular1 + ES6
-------------------------

*늦게 도착하여 거의 못들음..*

### Build & Bundling

-	`Webpack` 사용

### Controller

-	Controller `as` 문법 사용
	-	`$scope`를 분리

### Service

-	Class로 선언
-	static을 붙이지 않음

### Scope

-	ES6에서는 `$scope`를 사용하지 않는 추세
-	`this` 사용

### Directive

-	Class로 생성
-	객체를 넘기지 않음

	-	가독성
	-	상속 가능

### Component

-	Angular 1.6부터 Component가 생김

	-	Directive와는 다름
	-	Class를 사용할 수 없음

### BDD

-	`babel` + `browserfy` + `karma`
-	http.flush()
-	test 파일 동시 실행
	-	개별로 돌릴 때 문제 없던 문제가 동시 실행시 에러
	-	inject시 다른 객체를 들고오는 문제가 있음

Session 5. Universal Rendering
------------------------------

### 유니버셜 렌더링의 정의

*서버-사이드 렌더링과 클라이언트-사이드 렌더링을 동시에 지원하는 것*

-	두 언어가 다른 경우 변환 또는 재작성이 필요

### 장점과 단점

#### 장점

-	빠르다.
	-	(인지성능) : 사용자는 눈에 컨텐츠 데이터가 보이느냐 안보이냐에 따라 성능을 인지
	-	사용자 경험이 향상될 수 있다
-	서버 사이드 렌더링이 더 안정적

```
    i'd argue that.......
```

(Malte Ubl : Google AMP 프로젝트의 Tech lead)

-	옳다

	-	웹의 이념인 "웹 페이지는 문서"라는 생각을 지지한다면 각각의 주소에 해당하는 데이터에 유리하다.

-	SEO

	-	검색엔진 최적화 측면에서 유리

#### 단점

-	업무부담

	-	같은 코드를 써도 테스트는 각각 해야함
	-	고려할 점이 많고 더 수고롭다.
	-	유지보수는 점점 어려워진다.
	-	그 시간에 다른거 하는게 더 이득이다.

-	필요없다

	-	필요없는 서비스가 많다
	-	구글에서만 검색되면 된다

*용도에 맞게 선택 사용하라*

### 구성 방법

**1. 웹서버 + Node 렌더링**

-	인스타그램에서 사용했던 방식(Django + Node)

**2. Node 웹 서버 + 다른 언어 API 서버**

**3. 다른언어 웹서버 + 자바스크립트 확장 모듈**

### Best Practices

-	npm 버전보다는 최소화 버전의 성능이 더 좋다
-	노드 환경은 프로덕션 환경으로
-	HTML 결과물은 가능하다면 캐시
-	JS 모듈보다는 Node 서버 사용
	-	React 렌더링 성능에 있어 Java8의 Nashorn 엔진은 Node보다 **9배 가량 느리다.**\([벤치마킹](goo.gl/ZoCCwp)\)
	-	최근 핀터레스트가 Node 렌더링 서버를 선택
	-	JS 모듈은 프로토타이핑을 빠르게 할 수 있는 방법이고 별도의 렌더링 서버를 두면 관리 비용은 증가

[발표자님 홈](https://taegon.kim/)
