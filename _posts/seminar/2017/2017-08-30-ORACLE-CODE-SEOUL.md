---
layout:     post
title:      오라클 코드 서울 2017
author:     kingbbode
tags:       java oracle
subtitle:   대망의 오라클 코드 서울!
category:  seminar
outlink: 0
---


Session 1. When, Why, and How to CQRS
-------------------------------------

*Sebastian Daschner : 개발자, Java Champion 및 Java Rockstar Sebastian Daschner - IT-Beratung*

> 오늘날 대다수의 기업용 애플리케이션은 구현이 단순하고 간소한 생성-읽기-갱신-삭제의(CRUD) 데이터 모델에 기반하였습니다. 애플리케이션의 모델링을 위한 다른 방식은 명령과 쿼리의 역할을 구분하는, 즉 CQRS(Command-Query Responsibility Segregation)모델로, 특히 확장성에 대한 수요 증대와 관한 매우 흥미로운 솔루션과 유스 케이스(Use Case)를 가능케 합니다. 본 세션에서는 CQRS가 어떻게 궁극적 일관성, 도메인 중심 설계, 이벤트 소싱을 촉진하는가와 CQRS 애플리케이션의 개발 방법에 대하여 설명합니다. CQRS가 Java EE 기술과 함께 사용될 수 있는지, 이 프레임워크가 어떤 영역에서 이미 솔루션을 제공하고 있는지, 어떤 영역에서 보다 확장될 필요성이 있는지를 알아보세요. 본 세션은 라이브 코딩과 두 가지 접근법에 대한 세부 사항 점검으로 이루어질 것입니다.

### 전통적인 CRUD 방식의 문제

-	재현성

	-	이전 상태에 대한 값을 알 수 없음.

-	일관성

	-	Transaction 간의 경합 발생.

-	확장성

	-	단일 데이터베이스가 가지고 있는 확장성 문제.

### CQRS 로 갈 수 있는 동기

#### Event Sourcing

현재 **상태를 유추** 하는 방식.

-	atomic, immutable events, derived from use cases.

-	calculating state from events.

-	context & full History

#### Event-driven Architecture

**일관성을 보장** 하는 방식.

-	several systems involved.

##### Eventually Consistent Real-World

-	collaboration needed.

-	trust on (outdated) information.

-	compensating use-cases for "exception handling"

### CQRS

쓰기, 읽기를 분리할 수 있다.

-	Event Sourcing

-	Event-driven Architecture

#### 이점

-	scaling independently
-	each side optimized
	-	DB 의존성 해결
-	solves event sourcing scale problem
	-	저장된 상태 값이 있으므로, 매번 계산이 필요없음.
-	read side failover availability

#### 적용 고려

-	분산 시스템이 없다면 X
-	단일 데이터베이스라면(확장할 필요가 없다면) X
-	일관성이 필요 없다면 X

#### 데이터베이스 동기화 방법

-	데이터베이스는 동기화 하지 않는다.
-	Event Store 를 통해서만 접근한다.

[GIT : https://github.com/sdaschner/scalable-coffee-shop](https://github.com/sdaschner/scalable-coffee-shop)

---

Session 2. API, 마이크로서비스, 챗봇을 사용한 모던 애플리케이션의 개발
----------------------------------------------------------------------

*Kevin Walsh : 수석부사장, Oracle R&D*

> 빈번한 개발, 배포, 그리고 반복. 오늘날 개발자는 매일, 또는 매 시간마다 다수의 릴리즈를 개발 및 배포합니다. 이에 따라 아키텍처 설계도 단일(monolithic) 애플리케이션에서 작은 단위의 경량 프로토콜을 사용하는 마이크로 서비스로 변화하고 있습니다. 모바일, 챗봇, 가상 현실까지 망라하는 다양한 채널에서 고객이 참여하는 모던 애플리케이션이 개발되고 있습니다. API 우선 접근법은 이 모든 것을 연결하고 클라우드 네이티브 애플리케이션이 데이터와 프로세스에 접근하게 함으로써 프론트-엔드와 백-엔드 개발자 간 협업을 가능하게 해줍니다. 모던 애플리케이션 개발 플랫폼 상에서 개발자는 어떠한 디바이스에서도 모든 웹과 모바일 기반 애플리케이션을 쉽게 개발, 연결, 탄력적으로 확장할 수 있습니다. 마이크로서비스와 챗봇은 모든 기업이 API 우선 전략을 채택하도록 하는 촉매제가 되고 있습니다.

**소프트웨어는 더 이상 하나의 monolithic 시스템이 아니다! atomic 시스템의 조합이다!**

### API First : Design and Development

API 관리 플랫폼이 필요하다.

-	기능성 향상

-	Security

-	가시성 확보

등을 고려해야 한다.

#### Design

-	얼마나 쉽게 공유를 할 수 있을까

-	publish 했을 때 얼마나 정확한 정보를 가지고 있는가

-	얼마나 쉽게 개발할 수 있는가

apiary 에서는 마크다운으로 문서를 만들면 Reference, Test 등 모든게 자동으로 만들어진다.

#### Development

-	Version Management
-	Build
-	CI 고려
-	이슈 관리

apiary에서는 모든 것을 제공한다.

### 챗봇

메신저는 대부분의 사람들이 쓰며, 이것을 통해 API 관리 플랫폼과 통신할 수 있다면 매력적일 것이다.

---

Session 3. Java 9과 Spring 5로 바라보는 Java의 변화와 도전
----------------------------------------------------------

*Toby Lee : 개발자, 저자, KSUG 설립자*

> 올 가을, Java 9과 함께 이를 공식적으로 지원하는 Spring 5 버전이 공개됩니다. Java 9은 Java 8이 가져온 혁신적인 언어의 변화 위에 개발자들이 원하는 많은 기능을 대폭 추가한 최신 버전입니다. 또한 Spring 5는 Java 8을 기반으로 재작성되고, Java 9의 최신 기술을 지원하며 새로운 프로그래밍 모델을 도입했습니다. Java 9과 Spring 5를 살펴보면 더욱 복잡해지는 개발환경과 기술 요구사항에 맞설 수 있는 새로운 변화와 전략의 공통점이 보입니다. 지금까지 Java와 오픈소스 기술이 다양한 기술 변화와 그에 따른 위기에 어떻게 대응해왔고 어떤 결과를 가져왔는지, 새롭게 도전하는 Java 9와 Spring 5의 변화를 어떻게 바라봐야 할 지, 그리고 개발자들은 어떤 준비가 필요할 지 이야기합니다.

### Coming Soon !

**2017년 9월 21일**

-	JAVA 9

-	SPRING Framework 5

### Java id DEAD ?

-	언어가 죽었다는 언급은 모든 언어에서 많이 나타난다.

-	TIOBE Index 2017년 8월에 JAVA 가 1등.

-	indeed.com 에 구인공고에서 JAVA 의 비중이 월등히 높다.

**자바는 여전히 가장 인기있고, 많이 쓰이는 언어다.**

### 자바의 위기

#### 웹 어플리케이션의 급격한 발전

급격한 발전으로 과도한 Over Engineering 기술들이 나오게 되었다.

ex) EJB

Spring Framework 의 등장으로 결국 EJB 는 망.

**해결**

*자바와 객체지향의 기본으로 돌아간다*

#### 언어 발전의 요구와 호환성

경쟁 언어의 등장으로 여러 요구사항들이 나오게 되었다.

선발 언어이므로 아무래도 밀리는 것이 있었다.

**해결**

*언어의 발전과 함께 코드 호환성도 지킨다.*

-	Generic
-	Lambda

#### 간결한 코드와 관례로 무장한 언어 등장?

ex) 루비

-	간결하고 짧음.
-	설정 최소화.
-	엔터프라이즈 개발.

**해결**

*애노테이션 기반의 메타프로그래밍과 영리한 디폴트로 무장한 관례의 적극 도입*

-	JavaEE
-	Spring
-	Lombok

#### 함수형 프로그래밍과 비동기 논블록킹 개발의 도전

-	함수형 프로그래밍을 지원하는 언어가 인기.

**해결**

*함수형 프로그래밍이 가능하도록 처리*

*스프링 프레임워크에서 비동기 지원*

### 자바 9 와 스프링 5

**새로운 위기**

-	애노테이션과 메타프로그래밍, 관례의 범람

#### 애노테이션의 한계와 부작용

-	컴파일러에 의해 검증 가능한 타입이 아님
-	코드의 행위를 규정하지 않음
-	상속, 확장 규칙의 표준이 없음
-	이해하기 어려운 코드, 오해하기 쉬운 코드 가능성
-	테스트가 어렵거나 불가능함
-	커스토마이징하기 어려움

#### 리플렉션과 런타임 바이트코드 생성의 문제

-	성능 저하
-	타입정보 제거
-	디버깅 어려움
-	툴이나 컴파일러에 의한 검증 불가

### 함수형 프로그래밍의 도입이 시작된 업그레이드된 자바의 기본으로 돌아가자!!!

-	애노테이션과 리플렉션 제거
-	명시적인 코드로 모든 기능을 표현
-	불변객체 사용
-	함수의 조합을 이용
-	리액티브

### 스프링 5.0

함수형 프로그래밍을 적용한 함수형 웹 어플리케이션을 지원.

-	서블릿의 의존성 제거
	-	서블릿 컨테이너를 비동기 HTTP서버로 활용 가능
-	새로운 HTTP 요청과 응답의 추상화 - 불변 객체

	-	ServerRequest
	-	ServerResponse

-	두 개의 새로운 함수

	-	Mono
	-	Flux

-	독립형 애플리케이션

-	스프링 컨테이너

#### 함수형 웹 어플리케이션 구성요소

-	핸들러 함수
-	라우터 함수
-	HTTP 서버
-	DI 컨테이너

#### 함수형 웹 개발

-	간결한 코드
-	람다식-메소드 상호 호환 - 메소드 레퍼런스
	-	핸드러를 람다식 대신 메소드로 작성할 수 있음
	-	기능 종류에 따라 구조적으로 변경 가능

#### Mono / Flux

-	정보를 전달할 때 컨테이너 사용
-	데이터 가공, 변환 조합을 위한 다양한 연산 제공
-	리액티브 프로그래밍 기반
	-	ProjectReactor
	-	RxJava
-	함수형 웹 개발 외의 전 계층에 적용 가능
	-	서비스, 리포, API 호출
-	비동기 논블록킹 방식 스타일 작성 가능

#### 다양한 방식으로 개발 가능

-	순수 함수형 독립형
-	순수 함수형 애플리케이션 코드 + 자바 설정(@)
-	하이브리드 함수형 애플리케이션 코드(@)

### JavaSE 9 - Flow API와 ReactiveStreams

-	Flow API 는 ReactiveStreams 표준과 호환
-	스프링 Mono/Flux - Java 9 의 Flow APi를 함께 사용해서 개발 가능
-	Flow API가 많은 기술과의 브릿지 역할

---

Session 4. Java 8의 멋진 기능과 Java 9의 신기능 알아보기
--------------------------------------------------------

*David Buck : Principal Member of Technical Staff Oracle*

> 기존 Java 프로그래밍 모델의 현격한 업그레이드로써 JVM, Java 개발언어, 라이브러리의 종합적인 진전을 보인 Java 8에 이어, 2017년 오라클은 Java 9의 출시를 통해 Project Jigsaw의 모듈성 지원을 비롯하여 개발자가 고려해야 하는 다양한 주요 개선점과 발전 사항을 새롭게 선보일 것입니다. 본 세션에서는 Java 8이 애플리케이션 개발에 가져온 혁신과 Java 9의 주요 변화에 대하여 살펴보는 시간이 될 것입니다.

### Java 8

-	Lambda Expressions
-	Default Method
-	Method References
-	Date Time APIs - JSR 310

[www.oracle.com/java8launch](http://www.oracle.com/java8launch)

### Java 9

약 3주 후 릴리즈!!

`JEP`란 자바에 대한 제안서

-	JDK9 에 대한 JEP 를 검색해보면 많이 볼 수 있다.
-	특정 몇 가지를 소개

#### Behind the scenes improvements

성능 적인 향상

##### 1. JEP 250 : Store Interned Strings in CDS Archives

-	String class data 에 대한 공유.

##### 2. JEP 254 : Compact Strings

-	UTF-8 인코딩을 사용하지 않음.
-	String을 char 배열이 아닌 encoding flag를 포함한 byte 배열로 해석.

#### New features and functionality

##### 1. Project Jigsaw

Class Library 의 Module화!

JEP - 261, 200, 201, 220, 260, 282

Java Custom Runtime

-	Includes the Modular Application
-	꼭 필요한 것만 포함할 수 있음

##### 2. JEP 277 : Enhanced Deprecation

##### 3. JEP 269 : Convenience Factory Methods for Collections

##### 4. JEP 222 : jshell : The Java Shell (Read-Eval-Print Loop)

##### 5. JEP 226 : UTF-8 Property File

##### 6. JEP 249 : OCSP stapling.

##### 7. JEP 110 : HTTP/2 Client

등등 엄청 많음.. 너무 빨라서 다 못봄.. 없어지고 금지된 것도 많음.. 찾아보길!

---

Session 5. Docker 초보자들이 듣는 Container Cloud Service를 활용한 애플리케이션 컨테이너화
------------------------------------------------------------------------------------------

*Jongmin Lim : 컨설턴트 한국오라클*

> 본 세션에서는 GitHub에 기반하여 개발자들이 애플리케이션 스택을 구축하는 방법과 CLI 접속 없이 Container Cloud Service를 통한 애플리케이션 컨테이너화 방법을 알아봅니다.

### Docker 정의 및 특징

사용자가 필요로 하는 **소프트웨어를 이미지 형태로 패키지화** 해주고 **실행을 위해서 동일한 환경을 제공** 해주는 유틸리티입니다.

-	일관된 환경 제공(DevOps 플랫폼으로 적합)
-	개발자에 친화적
-	소스파일 빌드 및 테스트가 아닌 이미지 빌드/테스트
-	뛰어난 이식성

#### Docker Architecture

-	Docker Cleint
	-	CLI
-	Docekrfile
	-	Docker 컨테이너 구성 내용을 요약기술하는 텍스트파일
-	Image
-	Container - Docker run 명령어를 사용하여 Instance를 실행
-	Registry - 이미지 저장소

#### DockerHub

랩탑에서 이미지 공유 및 이동 가능

-	Docker Inc.
	-	Repository
	-	public and private

#### Docker Compose

-	멀티 컨테이너 도커 애플리케이션 정의 및 실행하는 툴
-	yaml에 정의된 참조 파일들
	-	docker-compose.yml

### OCCS Overview

OCCS (Oracle Container Cloud Service)

### OCCS를 활용한 App 배포 및 관리방법

---

Session 6. 누가 테스트를 원하는가? - 개발자의 불편함을 최소화하는 Java EE 애플리케이션 테스트 방법
--------------------------------------------------------------------------------------------------

*Sebastian Daschner : 개발자, Java Champion 및 Java Rockstar Sebastian Daschner - IT-Beratung*

> 기업에서 개발 시 테스트는 관련성이 적은 것으로 간주되고 있습니다. 그러나 수준의 애플리케이션이 아니고서는 테스트는 단지 하면 좋은 것이 아니라 제대로 구동되는 소프트웨어를 위한 필수 조건입니다. 본 세션은 Java EE 애플리케이션을 생산적이고도 포괄적인 방식으로 테스트하는 방법, 시스템 테스트가 필수불가결한 이유, Arquillian과 같은 통합 테스트 프레임워크가 대부분의 유스 케이스에 과대평가되는 이유를 조망합니다. 세션은 주로 라이브 코딩과 라이브 테스팅으로 구성됩니다.

**Live Coding**

### dependencies

`javax:javaee-api:7.0`, `junit4`, `mokito`, `assert`

### 일반적인 테스트

#### Mocking

내부적인 관계는 `mokito` 로 해결할 수 있음.

#### Parameterized

```java
@RunWith(Parameterized.class)
...

@Parameterized.Parameters
public static Collection<Object[]> parameter(){
  return asList(
    new Object[]{"kingbbode", "kingbbode@gmail.com"}
  );
}

@Prameterized.Parameter(0)
public String id;

@Prameterized.Parameter(1)
public String email;

@Test
...

```

**라이브 코딩을 감상 했음..**
