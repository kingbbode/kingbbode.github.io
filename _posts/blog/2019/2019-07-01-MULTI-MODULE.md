---
layout:     post
title:      멀티모듈 설계 이야기 with Spring, Gradle
author:     kingbbode
tags:       study
subtitle:   Spring Framework 와 함께하는 Gradle Multi Module 설계 경험기
category:  posts
outlink: 0
---


> 회사 블로그에 작성한 글 : [우아한형제들 기술블로그](http://woowabros.github.io/study/2019/07/01/multi-module.html)

## 멀티 모듈 프로젝트란?

 멀티 모듈 프로젝트를 처음 알게된 건 2017년 초였습니다. 당시에 저는 단일 프로젝트를 사용하고 있었습니다. 예를 들어 제가 회원 시스템을 개발 한다고 하면

 - member internal api
 - member external api
 - member batch

와 같이 서로 독립된 프로젝트 단위로 가지고 있었습니다. 이런 구조를 가지고 있을 때 가장 큰 문제점은 시스템의 중심 `Domain` 이 가져야할 구조와 규칙 등을 **동일하게 보장해주는 메커니즘** 이 없다는 것 입니다.

![독립 프로젝트의 모듈](/images/2019/MULTI-MODULE/sg-project.png){: style="width:75%; display:block; margin:40px auto 0;"}

개발자는 동일한 `Domain` 을 가지고 있는 위 3가지 어플리케이션을 열심히 복&붙하며 개발을 하게 됩니다. 매우 귀찮고 불안하게 말이지요.

멀티 모듈 프로젝트는 기존의 단일 프로젝트를 프로젝트 안의 모듈로서 갖을 수 있을 수 있는 구조를 제공합니다.

![멀티 프로젝트의 모듈](/images/2019/MULTI-MODULE/mt-project.png){: style="width:75%; display:block; margin:40px auto 0;"}

그래서 개발자는 하나의 시스템에서 중심 도메인을 모듈로 분리하여 위와 같은 보장 메커니즘을 제공받을 수 있게 됩니다.

*더 자세한 내용은 [기억보단 기록을:Gradle 멀티 프로젝트 관리](https://jojoldu.tistory.com/123) 블로깅을 참고하면 좋습니다.*

**"아! 멀티 모듈 프로젝트는 공통으로 사용하는 코드들을 모아놓고 같이 쓸 수 있게 해주는구나!"**

맞습니다. **맞지만**, 위와 같은 생각만 했었기때문에 저는 **많은 문제점** 을 맛보았고, **많은 시행착오** 를 하게 되었습니다.

## 실패한 멀티 모듈 프로젝트

 저는 아래와 같은 어플리케이션들을 가지는 시스템을 멀티 모듈로 바꾸어보았습니다.

 - Kingbbode System
   - internal api
   - external api
   - batch

공통으로 사용하는 코드들을 모았습니다. 모듈을 이름을 뭘로 하지? `common` 이 좋겠다! 그리고 `common` 의 저주가 시작되었습니다. 아래와 같은 멋진(?..) 구조가 완성되었습니다.

![실패한 프로젝트의 모듈](/images/2019/MULTI-MODULE/failed-project.png){: style="width:75%; display:block; margin:40px auto 0;"}

**"여러분. 이제 common 모듈에서만 수정하면 됩니다. 더 이상 노가다하지 않아도 됩니다."**

### 공통(Common) 모듈의 저주

공통 모듈에 대부분의 핵심 또는 공통 코드들이 다 들어가게 되었습니다. 그리고 많은 문제점들이 생기기 시작했습니다.

**A 어플리케이션** 에서 기능을 추가합니다. 이 때 코드는 어디에 작성되게 될까요? 개발자는 고민을 하게 되고, 코드는 어떠한 선택에 의해 빈번하든 아니든 **공통 모듈에 점점 추가** 가 됩니다. **B 어플리케이션** 에서도 마찬가지 일 것 입니다.

이제 **C 어플리케이션** 에서 기능을 추가합니다. 공통 모듈에 작성된 유용해보이는 코드들이 있습니다. 사용하게 됩니다. 사실 그 기능은 **A 어플리케이션** 을 위해 작성한 코드입니다.

위 과정이 반복이 되다보면 어느세 공통 모듈은 걷잡을 수 없이 커져있을 것 입니다. 그리고 어플리케이션에서 하는 일이 점점 줄어들고 공통 모듈에서 점점 더 많은 일을 하게 됩니다.

#### 1. 스파게티 코드

모든 프로젝트들은 주기적인 청소가 반드시 필요합니다. 그게 기능의 F/O든 자발적인 리펙토링이든 말이지요. 그러나 공통 모듈의 저주가 이를 방해합니다.

> A: "오래된 기능이 F/O 되었다. 코드를 정리해볼까?", "이쪽 흐름이나 코드가 효율적이지 않네. 리펙토링 좀 해볼까?"

> 관리자 : "영향 범위는?"

> A: "**시스템 전체..**"

![스파게티 코드](/images/2019/MULTI-MODULE/sp-code.png){: style="width:75%; display:block; margin:40px auto 0;"}

코드가 꼬리에 꼬리를 물고 결국 하나의 코드만 수정해도 전체가 영향받는 현상은 불가피해보입니다.

심지어 A를 B로, B를 C로 가공한 코드에서 다시 C에서 A로 가공한 코드도 본적이 있습니다. 아래와 같이 말이지요.

```java
class AService {
  private final ARepository aRepository;

  public A act(Long id) {
    ...
    return aRepository.findById(id);
  }
}

class BService {
  private final AService aService;

  public B act(Long id) {
    A a = aService.act(id);
    ...
    return mapToB(a);
  }
}

class CService {
  private final BService bService;

  public C act(Long id) {
    B b = bService.act(id);
    ...
    return mapToC(b);
  }
}

class DService {
  private final CService cService;

  //띠용!!
  public A act(Long id) {
    C c = cService.act(id);
    ...
    return mapToA(c);
  }
}
```

공통 모듈에 들어간 코드는 더 이상 처음 작성한 의도만을 위한 코드가 아니게 됩니다. 공통이기 때문입니다. 우리는 혼자 개발하는 것이 아닐 뿐더라 혼자 개발한다고 해도 이러한 실수는 반드시 생기게 됩니다.

#### 2. 의존성 덩어리

공통 모듈은 어떤 의존성을 갖게 될까요? 결국 전부라고 저는 확신합니다.

웹쪽 설정을 사용하니까 `spring-boot-starter-web`, JPA를 사용하니까 `spring-boot-starter-data-jpa`, REDIS도 사용하니까 `spring-boot-starter-data-redis` 등등등 계속 추가하게 됩니다. 전부가 아니더라 프로젝트에서 사용하는 대부분의 의존성이 공통 모듈로부터 시작될 것 입니다.

![의존성 덩어리](/images/2019/MULTI-MODULE/big-depend.png){: style="width:75%; display:block; margin:40px auto 0;"}

문제는 어플리케이션들이 사용하는 의존성은 다를 수도 있다는 것 입니다. 예를 들어 데이터베이스를 사용하지 않는 어플리케이션에서 공통 모듈을 사용하기 위해 데이터베이스와 커넥션을 맺게 됩니다. 그 외에 다른 의존성도 마찬가지겠지요.

스프링 부트는 의존성을 기반으로 발동되는 `AutoConfiguration` 들이 아주 많이 있지요. 의존성 덩어리인 공통 모듈로 인해 어디선가 발동되는 스프링 부트의 마법으로 어플리케이션은 최적화되어 돌아가지 못하게 될 것이고, 이것은 곧 장애 포인트가 될 것 입니다.

#### 3. 공통 설정

설정까지 공통 모듈로 몰게 되는 경우도 많이 보았습니다. 고정적으로 공통으로 사용되어야 하는 호스트 정보 등은 경우에 따라 공통으로 보아야할 수도 있지만, 그 외의 Thread Pool, Connection Pool, Timeout 등의 설정도 가장 민감하고 중요한 어플리케이션 기준으로 몰아들어갑니다.


![의존성 덩어리](/images/2019/MULTI-MODULE/common-setting.png){: style="width:75%; display:block; margin:40px auto 0;"}

이 때 발생하는 문제의 대표적인 예로 데이터베이스 커넥션이 있습니다. 모든 데이터베이스에는 가질 수 있는 최대 커넥션 개수가 정해져 있습니다. 데이터베이스를 사용하지 않는 어플리케이션에서 공통 모듈을 사용하기 위해 사용되는 커넥션으로 인해 실제로 데이터베이스를 사용하는 어플리케이션과 해당 어플리케이션까지 모두 문제가 생길 수가 있게 됩니다.

이 외에도 어플리케이션마다 다르게 작성되어야 할 설정을 한 곳으로 몰게 되었을 때 수많은 문제점들이 있겠지요?

---

왜 이런 문제점들이 생기게 되었을까요? 제 생각에는 **멀티 모듈 프로젝트는 독립적으로 실행 가능한 어플리케이션을 1개 이상 가질 수 있기 때문입니다.**

멀티 모듈 프로젝트는 하나의 시스템을 단위로 만들어집니다. 여기서 말하는 **시스템** 은 아래와 같은 정의를 말합니다.

![시스템](/images/2019/MULTI-MODULE/system.png){: style="width:75%; display:block; margin:40px auto 0;"}

[출처: 이벤트 기반 분산 시스템을 향한 여정 (Arawn Park)](https://www.slideshare.net/arawnkr/ss-94475606)

`멀티 모듈 프로젝트`는 독립적으로 실행가능한 어플리케이션 모듈을 1개 이상 가지고 있으며, 사용하는 인프라 자원 역시 1개 이상을 가지고 있습니다. **독립적으로 실행 가능** 한 어플리케이션들은 당연히 서로 다른 책임과 역할을 가지기 때문에 `스파게티 코드`, `의존성 덩어리`, `설정` 등의 문제를 피하기 위해 하위의 모듈들에 대한 의존성과 사용성에 대한 개방, 폐쇄를 철저히 해야만 했습니다.

## 멀티 모듈 구성하기

### **글을 읽기 전 주의사항**

아래 글에서 나오는 용어와 관점을 우리가 많이 바라보는 시점인 `레이어 아키텍처` 의 시점으로 바라보았을 때 혼동이 올 수 있습니다.

![수평 관점](/images/2019/MULTI-MODULE/before-prdm.png){: style="width:75%; display:block; margin:40px auto 0;"}

**아래 작성된 내용은 기존의 시점과 조금 다른 시점에서 어플리케이션을 바라본 점이라는 것을 인지하여 읽어주시면 감사하겠습니다**

![수직 관점](/images/2019/MULTI-MODULE/after-prdm.png){: style="width:75%; display:block; margin:40px auto 0;"}

---

사전마다 거의 재각각으로 설명이 되어있지만, 공통적으로 나오는 내용으로 정리했을 때 모듈이라는 개념은 **독립적으로 운영될 수 있는 의미를 가지는 구성요소 단위** 라고 정의가 됩니다.

> 모듈은 여러 가지로 정의될 수 있지만, 일반적으로 큰 체계의 구성요소이고, 다른 구성요소와 독립적으로 운영된다. <br>
[출처: wikipedia](https://ko.wikipedia.org/wiki/%EB%AA%A8%EB%93%88%EC%84%B1_(%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D))

모듈이라는 용어는 개발 영역 안에서도 여러 영역에서 쓰이고 있습니다. 그렇다면 설계해야 할 프로젝트 모듈의 예시가 무엇이 있을까요? 답은 굉장히 간단했고 가까운 곳에 많이 있습니다. 우리가 제공받고 있는 의존 라이브러리들이 모두 모듈입니다. `Spring Framework`, `타 회사 연동용 라이브러리` 등등등 우리는 굉장히 많은 모듈들을 사용하고 있습니다.

그래서 많은 인지도와 인기를 가지고 있는 여러 오픈소스 라이브러리들 중 멀티 모듈 구조를 갖는 프로젝트들이 어떠한 구조로 모듈을 사용하고 있는지 참고하기 위해 찾아보았습니다. 라이브러리(프레임워크)와 우리가 구성해야할 시스템의 모듈은 제공 정도의 목적이 다르기 때문에 그들의 구조를 참고하는데에는 많은 괴리감이 있었습니다. 그러나 모듈 자체에 대한 특징에서는 공통적으로 어떠한 특징을 찾아낼 수 있었습니다.

- 모듈은 독립적인 의미가 갖는다.
- 모듈은 어떠한 추상화 정도에 대한 계층을 가지고 있다.
- 계층 간 의존 관계에 대한 규칙이 있다.

이러한 특징들을 우리가 설계해야할 시스템에 적용하여 고민하여 아래와 같은 시스템에서 가져야할 계층 구조를 정의했습니다.

![모듈 계층](/images/2019/MULTI-MODULE/modules.png){: style="width:75%; display:block; margin:40px auto 0;"}

- 독립 모듈 계층
- 도메인 모듈 계층
- 내부 모듈 계층
- 공통 모듈 계층
- 어플리케이션 모듈 계층

이러한 정의된 계층 구조를 갖음으로써 우리는 모듈이 어디까지 책임과 역할 가질 수 있는지를 명확히 할 수 있고, 의존 관계 또한 최소화하여 최적화된 프로젝트를 만들어낼 수 있습니다. 위 계층에 어떠한 독립적으로 의미를 갖는 모듈들이 배치될 수 있는지 살펴보겠습니다.

---

### 1. 독립 모듈 계층

![독립 모듈 관계](/images/2019/MULTI-MODULE/dependency-relation1.png){: style="width:75%; display:block; margin:40px auto 0;"}

 시스템과 무관하게 어디에서나 사용 가능한 라이브러리 성격의 모듈을 이 계층에 배치하였습니다. 프로젝트 내에서 가장 프로젝트와 성격이 먼 모듈이 이 계층에 위치하게 됩니다. 시스템, 도메인의 비지니스와 전혀 별게로 자체 제작한 라이브러리 같은 모듈이 이 계층으로 들어갈 것이며, 경우에 따라 사내 레파지토리에 올라갈 수도 있을 것 입니다. 특별한 경우가 아니라면 이 계층에 모듈이 생길 일은 거의 없을 것 입니다.

 이 계층은 아래와 같은 원칙을 갖습니다.
 - 자체로서 독립적인 역할을 갖는다.

언제든 이 계층의 모듈을 대처할 탄탄한 라이브러리가 나오거나, 사내 레파지토리 등에 올라간다면 프로젝트에서 이 모듈은 제거 될 것이므로, 프로젝트 내에 의존 관계를 두지 않아야 합니다.

 ||어플리케이션 모듈|내부 모듈|도메인 모듈|독립 모듈 계층|공통 모듈|
 |--|--|--|--|--|--|
 |사용 가능한 모듈 여부|X|X|X|X|X|

저같은 경우 아래와 같은 모듈을 배치하였습니다.

![독립적으로 사용 가능한 모듈](/images/2019/MULTI-MODULE/indep-modules.png){: style="width:75%; display:block; margin:40px auto 0;"}

**yaml-importer**

> Spring Framework 의 EnvironmentPostProcessor Cycle에 특정 path 의 Yaml 기반으로 Configure Property 를 로드하는 기능을 하는 모듈입니다. 이 모듈은 Spring Framework 를 사용하는 어떠한 프로젝트에서도 사용할 수 있습니다. 왜 필요했는지는 아래에서 다시 언급하겠습니다.

**data-dynamo-reactive**
> Reactor 프로젝트에서 DynamoDB 를 사용하기 위해 nio를 지원하는 AWS SDK V2를 사용했습니다. 이 모듈에서는 DynamoDB Table 과 Java Object 에 대한 Mapping 을 책임지며, DDL, Memory Mode, CRUD 등등 환경과 기능을 제공합니다. 마찬가지로 Spring Framework를 사용하는 어떠한 프로젝트에서도 사용할 수 있습니다.

### 2. 공통 모듈 계층

![공통 모듈 관계](/images/2019/MULTI-MODULE/dependency-relation4.png){: style="width:75%; display:block; margin:40px auto 0;"}

공통 모듈이라니 이게 무슨 소리인가 할 수 있겠지만, 하나의 프로젝트에서 모든 모듈에서 사용될 수 있는 것들은 나올 수 밖에 없었습니다. 그래서 고민 끝에 계층을 정의하였고, 대신에 이 모듈은 가장 큰 제약사항을 두었습니다.

- Type, Util 등을 정의한다.
- 가능하면 사용하지 않는다.

마찬가지로 시스템 내 모든 모듈들이 의존할 수 있을만큼 얇은 의존성을 제공해야 하기 때문에 프로젝트 내 어떠한 모듈도 의존하지 않아야 합니다.

||어플리케이션 모듈|내부 모듈|도메인 모듈|독립 모듈|공통 모듈|
|--|--|--|--|--|--|
|사용 가능한 모듈 여부|X|X|X|X|X|

심지어 저는 이 모듈에서는 외부의 의존 관계도 갖지 않았습니다.

```
bootJar { enabled = false }
jar { enabled = true }

dependencies {
}
```

그 말은 즉 순수 Java Class 만 정의할 수 있다는 이야기입니다. 생성된 모듈에서는 시스템에서 정말 많이 쓰이는 Type 성격의 DTO나 기본적인 Util Class만 배치하였습니다. 그렇다고 모든 Type이나 Util 격 Class가 이 모듈로 배치되는 것은 아닙니다. 가능하다면 이 모듈을 사용하지 않는다는 원칙을 두고 있습니다.

### 3. 도메인 모듈 계층

![도메인 모듈 관계](/images/2019/MULTI-MODULE/dependency-relation2.png){: style="width:75%; display:block; margin:40px auto 0;"}

 시스템의 중심 도메인을 다루는 모듈을 이 계층에 배치하였습니다. 아래의 `내부 모듈 계층` 과 비슷한 성격의 계층이지만, 저장소와 밀접한 중심 도메인을 다루는 계층은 더 견고하고 특별하게 격리되고 관리되어야 하기 때문에 반드시 분리되어야 합니다.

 이 계층은 아래와 같은 원칙을 갖습니다.

 - 서비스 비지니스를 모른다.
 - 하나의 모듈은 최대 하나의 인프라스트럭처에 대한 책임만 갖는다.
 - 도메인 모듈을 조합한 더 큰 단위의 도메인 모듈이 있을 수 있다.

 이 계층에는 프로젝트 안의 어떠한 실행 가능한 어플리케이션에서도 사용 가능한 모듈이 위치해야 합니다. 그러기 위해서는 저장소 외 시스템 특성을 알지 않아야 하기 때문에 `내부 모듈` 계층을 의존하지 않도록 구성했습니다.

 ||어플리케이션 모듈|내부 모듈|도메인 모듈|독립 모듈|공통 모듈|
 |--|--|--|--|--|--|
 |사용 가능한 모듈 여부|X|X|O|O|O|

 이 계층의 모듈은 오로지 도메인에 집중합니다. 어떠한 도메인이든 그 도메인이 가져야할 서비스와 무관한 도메인의 비지니스가 있습니다. 도메인 중심 설계에 대한 내용으로 견고한 도메인에서부터 프로젝트를 만들어 올라가면 자연스럽게 이러한 구조가 나올 수 있기도 합니다.

 하나의 모듈은 최대 하나의 인프라스터럭처를 갖는 것은 의존성의 전파를 방지하기 위해서입니다. 사용하지 않는 인프라스트럭처에 대한 설정까지 해줘야 했던 실패한 멀티 모듈에서 배운 경험으로 인프라스트럭처 단위로 최대한 작은 단위부터 작성되야 한다는 것을 알게 되었습니다.

 저는 아래와 같은 구성을 하였습니다.

#### 단일 인프라스트럭처 사용 모듈

가장 흔한 케이스로 RDBMS가 중심 도메인을 품고 있는 프로젝트는 이런 도메인 모듈이 하나만 만들어집니다. 보통 네이밍은 `xxx-domain` 혹은 `xxx-core` 로 작성했습니다.

도메인 모듈은 아래와 같은 책임을 갖습니다.

![도메인 모듈 계층](/images/2019/MULTI-MODULE/domain-layer.png){: style="width:75%; display:block; margin:40px auto 0;"}

**첫번째는 `Domain` 입니다.** Java Class 로 표현된 도메인 Class들이 이곳에 위치합니다. JPA 를 기준으로 한다면 테이블과 맵핑되는 Class 들 입니다. 도메인 모듈 내부에서는 이곳에 위치한 도메인들을 사용하여 대화를 하게 됩니다.

**두번째는 `Repository` 입니다.** 도메인의 `조회`, `저장`, `수정`, `삭제` 역할을 합니다. 여기서 주의할 점은 모든 `조회`, `저장`, `수정`, `삭제` 역할을 이곳에서 하는 것이 아니라는 점 입니다. 이 모듈은 시스템에서 가장 보호받아야하며 가장 견고해야 할 모듈이므로, 이 모듈에서 `조회`, `저장`, `수정`, `삭제` 에 대한 정의를 작성할 때 많은 고민을 해야합니다. <br> 예를 들어 Admin Application 에서 시스템 도메인에 대한 통계를 기능을 추가한다고 했을 때를 가정해보겠습니다. 이 때에 통계 조회에 대한 정의는 시스템이 갖는 중심 역할에 따라 달라지게 됩니다. 만들고 있는 시스템에서 통계라는 기능이 중심 역할로 볼 수 있다면 도메인 모듈에 작성할 것이고, 그렇지 않다면 사용을 하는 측에서 작성되어야 합니다. 만일 이 시스템이 `주문`이라는 중심 도메인을 갖는다면 저는 통계 기능은 사용하는 측(Application Module)에 작성할 것 입니다.

**세번째는 `Domain Service` 입니다.** 이 계층은 도메인의 비지니스를 책임집니다. 그렇기 때문에 도메인이 갖는 비지니스가 단순하다면 이 계층은 생기지 않을 수도 있습니다. 이 계층에서 저는 '트랜잭션의 단위'를 정의했고, '요청 데이터를 검증' 했으며, '이벤트를 발생'하는 일 등 도메인의 비지니스를 작성했습니다.

#### 다중 인프라스트럭처 사용 모듈

도메인 계층에서 여러 인프라스트럭처를 사용해야할 때 꼬이는 의존 관계를 많이 보았습니다.

예를 들어 RDBMS를 사용하는 'A'라는 도메인 모듈이 있습니다. 시스템의 요구사항으로 'A'의 도메인의 삽입이 발생할 때 'B'라는 도메인으로 가공하여 임시 저장시켜야 하는 요구가 생겼습니다.

이 때 RDBMS 를 사용하는 도메인 모듈에 `Redis` 의존성을 추가한다면 의존 지옥을 맛볼 가능성이 큽니다. `A` 도메인에 대한 조회성을 위해서만 도메인 모듈을 사용해야하는 상황에서도 `Redis`에 대한 설정과 의존을 설정해주어야 합니다. 앞으로 더 다양한 `인프라스트럭처`를 사용하게 될 때 점점 더 지옥이 열릴 것을 확신합니다.

그래서 하나의 모듈은 하나의 `인프라스트럭처`만 책임지도록 모듈을 작성하는 것을 추천합니다. 그리고 위와 같은 두 인프라스트럭처 사이의 관계가 생길 때 두 모듈을 품는 모듈을 작성하거나 두 인프라스트럭처 모듈을 품는 어플리케이션에서 처리하시길 바랍니다.

![도메인 모듈 계층](/images/2019/MULTI-MODULE/multi-infra.png){: style="width:75%; display:block; margin:40px auto 0;"}

결국 하나의 모듈이 두 인프라스트럭처를 책임지는 것이 아니냐고 생각할 수 있지만, 두 인프라스트럭처를 모듈을 품는 모듈에게는 인프라스트럭처에 대한 책임이 주어지지 않습니다.

`A`, `B` 도메인이 다른 인프라를 갖지만 절대 떨어질 수 없다고 단정 지을 수 있다면 단일 모듈로도 작성할 수도 있을 것 입니다. 위와 같이 분리된 모듈로 작성하는 것은, 추후 분리와 확장을 더 용이하도록 하기 위한 기초 작업입니다.

### 4. 내부 모듈 계층

![내부 모듈 관계](/images/2019/MULTI-MODULE/dependency-relation3.png){: style="width:75%; display:block; margin:40px auto 0;"}

저장소, 도메인 외 시스템에서 필요한 모듈들은 이 계층에 속하게 됩니다. 독립 모듈 계층은 시스템에도 전혀 관여되지 않았다면, 이 계층은 시스템과는 연관이 있는 모듈을 말합니다.

이 계층은 아래와 같은 원칙을 갖습니다.

- 어플리케이션, 도메인 비지니스를 모른다.

이 계층은 시스템 전체적인 기능을 서포트하기 위한 기능 모듈이 만들어질 수 있습니다. 이 계층 역시 프로젝트 안의 어떠한 실행 가능한 어플리케이션에서도 독립 사용 가능한 모듈이 위치되어야 하므로, `도메인 계층`을 의존하지 않습니다.

||어플리케이션 모듈|내부 모듈|도메인 모듈|독립 모듈|공통 모듈|
|--|--|--|--|--|--|
|사용 가능한 모듈 여부|X|O|X|O|O|

저는 아래와 같은 모듈들을 배치했습니다.

![클라이언트들](/images/2019/MULTI-MODULE/clients.png){: style="width:75%; display:block; margin:40px auto 0;"}

**core-web**
> web 설정을 사용하는 프로젝트에서 사용할 수 있는 모듈이다. 주로 Web Filter 를 이용한 보안, 로깅 등으로 활용되며, 웹에 대한 필수적인 공통 설정을 하기도 한다.

**xxx-client**
> 외부의 xxx 시스템과 통신을 책임지는 모듈이며 각 외부 시스템별로 따로 모듈을 만들었습니다. 이 모듈은 비지니스와 관계없이 요청과 응답을 할 수 있는 사용성을 제공하고, 요청에 대한 설렁과 스팩을 책임집니다. 어플리케이션 모듈에서는 사용하는 외부 시스템 모듈만 사용하게 됩니다.

**xxx-event-publisher**
> 특정 이벤트에 대한 처리를 담당합니다. 여기서 말하는 이벤트는 Spring ApplicationEvent 를 말하며, 이벤트가 발생했을 때 `SQS` 로 이벤트를 전송하거나, 로그를 남기는 등 특정 행위를 처리합니다. 이 모듈 또한 하고 있는 주요 행위의 범위에 따라 따로 모듈이 생성될 수 있습니다.

### 5. 어플리케이션 모듈 계층

![어플리케이션 모듈 관계](/images/2019/MULTI-MODULE/dependency-relation5.png){: style="width:75%; display:block; margin:40px auto 0;"}

이 계층은 우리가 잘 알고 있는 독립적으로 실행 가능한 어플리케이션 모듈 계층입니다.

어플리케이션 모듈은 하위 설계 했던 모듈들을 조립하여 서비스 비지니스를 완성시킵니다.

||어플리케이션 모듈|내부 모듈|도메인 모듈|독립 모듈|공통 모듈|
|--|--|--|--|--|--|
|사용 가능한 모듈 여부|X|O|O|O|O|

저는 아래와 같은 모듈을 주로 배치하고 있습니다.

- xx-app-batch
- xx-app-worker
- xx-app-internal-api
- xx-app-external-api

이 계층의 모듈에게는 'app' 이라는 네이밍을 넣어서 모듈 내의 실행 가능한 어플리케이션이라는 것을 명확하게 했습니다. 어플리케이션 모듈은 `독립 모듈 계층`, `도메인 모듈 계층`, `내부 모듈 계층`, `공통 모듈 계층` 을 사용성에 따라 의존성을 추가하여 사용하게 됩니다.

### 모듈 계층의 의존 관계 흐름

![모듈 관계](/images/2019/MULTI-MODULE/dependency-relation.png){: style="width:75%; display:block; margin:40px auto 0;"}

---

## 멀티모듈 구성 효과

### 1. 명확한 추상화 경계

계층에 따라 분리된 모듈로 인해 추상화 레벨을 맞출 수 있었습니다.

![추상화 경계](/images/2019/MULTI-MODULE/abstraction.png){: style="width:75%; display:block; margin:40px auto 0;"}

기존에는 하나의 모듈에서 다양한 어플리케이션 레이어와 의존성을 책임지고 있었지만, 계층화된 모듈 레이어로 분리됨으로서 아래와 같이 역할과 책임의 선을 명확하게 할 수 있었고, 그로인해 개발의 생산성을 향상시킬 수 있었습니다.

![추상화 경계](/images/2019/MULTI-MODULE/module-refactor.png){: style="width:75%; display:block; margin:40px auto 0;"}

기존의 들쭉날쭉했던 기능의 제공 정도와 추상화 정도의 경계가 계층 설계로인해 명확해짐으로써 얻는 이득이 많았습니다.

- 각 모듈이 갖는 책임과 역할이 명확하여 리펙토링, 기능의 변경의 영향 범위를 파악하기 용이합니다.
- 경계가 명확해짐으로써 기능의 제공 정도를 예측 가능하여 스파게티 코드 발생 가능성이 줄어들었습니다.
- 역할과 책임에 대한 애매함이 없어짐으로써 어떤 모듈에서 어느정도까지를 개발되야할지 명확해졌습니다.

위 세가지의 효과로 개발자는 더 빠르게 책임과 역할에 집중하여 확신을 가지고 개발을 할 수 있습니다.

### 2. 최소 의존성

각 모듈들이 각자 필요한 의존성만 가지고 있기때문에 최소한의 의존성을 갖을 수 있게 됩니다.

의존성 관리가 그렇게 중요할까?

자바에서 성능적으로 본다면 사실 그렇게 큰 성능 차이는 아닐 것 입니다. 그러나 좋은 개발환경이란 측면과 `Spring Boot` 를 사용하는 측면에서는 손해를 볼 수 있습니다. 여러가지 손해들이 있겠지만 아래 두 가지를 짚어보았습니다.

**첫번째로는 여지의 개방으로 개발 생산성을 떨어트립니다.** 무엇이든 끌어다쓸 수 있게 된다는 측면에서는 초기에는 빠르게 개발을 할 수도 있지만, 그로인해 만들어지는 스파게티 의존으로 점차 개발 생산성이 떨어지게 됩니다.

**두번째로 `Spring Boot` 를 사용하고 있기 때문에 예상치 못했거나, 불필요한 설정이 동작하게 될 수 있습니다.** `Spring Boot` 는 Class 의 존재만으로도 작동되는 설정들이 아주 많이 있습니다.

![class 컨디셔널](/images/2019/MULTI-MODULE/conditional.png){: style="width:75%; display:block; margin:40px auto 0;"}

이로인해 예상치 못한 설정이 로드될 수도 있으며, 혹은 위 잘못된 멀티모듈에서 보았던 사용하지 않는 자원에 대한 설정을 해줘야하는 일들도 발생할 수 있습니다.

역할과 책임이 명확해진 모듈 설계로 위 두 가지 측면에서도 이득을 볼 수 있을 것 입니다.

---

## 의존관계 완성하기

설계는 완료되었지만, 몇 가지 마음에 안드는 점들이 있었습니다. 그래서 아래와 같은 방식으로 제가 원하는 완성된 멀티모듈 프로젝트를 만들 수 있었습니다.

### 1. 모듈 Configuration Property Load

`Spring Boot` 는 몇 가지 규칙에 의해 `configuration property` 를 로드하도록 되어있습니다.

![properties](/images/2019/MULTI-MODULE/properties.png){: style="width:75%; display:block; margin:40px auto 0;"}

[출처 : Spring Framework docs](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)

그래서 대부분 모듈에 속해있는 properties 파일을 작동시키기 위해 `application-${profile}`이 로드되는 방식을 많이들 이용하고 있습니다. 모듈에는 `application-xxx.yml` 과 같은 네이밍을 사용하여 변수를 셋팅하고 어플리케이션 모듈의 `application.yml` 에서

```
spring.profiles.include=xxx
```

와 같이 `xxx` profile을 추가하여 파일을 로드하는 방식입니다.

저는 이 방식이 마음에 들지 않았던 여러 이유가 있었지만, 가장 큰 것은 모듈 사용의 번거로움입니다. 모듈 의존성을 추가한 뒤에도 properties 혹은 yaml 에 profile을 설정해주어야만 정상적으로 동작하는 번거로움이 마음에 들지 않았습니다. 저는 모듈 의존성만 추가하면 되는 사용성을 만들고 싶었습니다.

그래서 특정 path에 있는 yaml 을 Spring Boot 의 적절한 사이클에 밀어넣을 수 있는 `yaml-importer` 모듈을 개발했습니다. 그렇게 저는 모듈 의존성만 추가하면 되는 사용성을 만들었습니다.

### 2. 모듈 Component Scan

모듈에 있는 `Bean(Configuration, Service, Repository 등)`을 로드할 수 있는 방법이 무엇이 있을까요?

**첫번째로는 `spring.factories`를 통한 로딩 방식입니다.** `Spring Framework` 는 모듈에 있는 `META-INF/spring.factories` 파일을 통해 특정 사이클에서 로드될 Class 를 설정해줄 수 있습니다.

```
org.springframework.boot.env.EnvironmentPostProcessor=\
com.baemin.xxx.processor.ThirdPropertiesEnvironmentPostProcessor
```

위에서 언급한 `yaml-importer` 모듈은 `EnvironmentPostProcessor`라는 스프링 사이클에 설정을 로드하도록 되어있습니다.

`spring.factories` 를 사용해도 되겠다고 생각할 수 있지만, 이 방식은 시스템과 관련없는 위와 같은 모듈에 대한 설정을 로딩할 때만 사용하는 것이 맞습니다.

**두번째 방식은 `base package` 의 확장입니다.** `Spring Boot`는 아래와 같이 scan 되는 package를 확장할 수 있습니다.

```
@SpringBootApplication(scanBasePackages = "com.baemin.xxx.domain.shop")
public class XXXApplication {
```

이 방식이 스프링에서 권장하는 방식이며, 이 방식을 사용하는 것이 스프링의 의도와 잘 맞는 것 같습니다. 그러나 이 방식을 사용할 때 역시 저는 불편함을 느꼈습니다. 모듈 의존성을 추가하고 `scanBasePackages`를 추가로 작성하는 것이 번거롭다고 생각했습니다.

**그래서 제가 선택한 방식은 Application 클래스를 한 패키지 상위에 작성하는 관례는 두는 것 입니다.** 프로젝트 안에서 모든 모듈은 공통적으로 `com.baemin.xxx` 라는 상위 공통 패키지를 갖고 어플리케이션 모듈에서는 `com.baemin.xxx` 패키지에 Application 클래스를 배치합니다.

**어플리케이션 모듈**

![package app](/images/2019/MULTI-MODULE/package-app.png){: style="width:75%; display:block; margin:40px auto 0;"}

**하위 모듈**

![package redis](/images/2019/MULTI-MODULE/package-redis.png){: style="width:75%; display:block; margin:40px auto 0;"}

![package dynamo](/images/2019/MULTI-MODULE/package-dynamo.png){: style="width:75%; display:block; margin:40px auto 0;"}

이렇게 했을 때 모듈 내의 모든 `Bean` 을 모듈 의존성 추가만으로도 사용할 수 있었습니다.

### 3. 접근 개방/폐쇄

설계와 개발을 하다보면 개방과 폐쇄에 대해서 많은 고민들을 하고 있을 것 입니다. A라는 코어가 있고 A는 보호받아야하기 때문에 B라는 서비스를 만들게 되는 경우가 있습니다.

이럴 때 우리가 할 수 있는 방법 중 **첫번째는 접근제한자의 활용입니다.** 패키지나 상속 수준의 접근제한을 설정하여 A라는 코어를 보호할 수 있습니다. 그러나 A와 B를 모듈 차원에서 보았을 때는 쉽지 않습니다. 모듈이란 것은 독립적인 의미를 갖어야하므로, 이 둘 모듈의 내용을 제한할 수 없습니다.

**두번째는 관례입니다.** 함께 개발하는 사람들끼리 함께 지켜야하는 관례를 만들어내어 문서, 코드리뷰 등을 통해 새로운 구성원들에게도 계속 전파되게끔 노력해야하는 방법입니다. 많이 사용할 수 밖에 없는 방법이지만, 이보다 좋은 매커니즘이 있다면 꼭 사용하고 싶어집니다.

**그래서 제가 사용하는 것이 세번째 방법인 `gradle` 의 `implementation`, `api` 의 활용입니다.** `gradle` 이 업데이트되면서 기존 `compile` 이 `deprecated` 되고 두 가지 방식을 제공하도록 바뀌었습니다. 이 두 가지는 매우 유용합니다. `api` 는 기존의 `compile` 과 거의 같다고 보면 됩니다. 주목해야할 것은 `implementation` 입니다.

`implementation` 는 하위 의존 관계를 숨겨줍니다. 숨겨준다는 것은 즉 하위 의존에 대한 접근을 제한한다는 것 입니다.

**A Module**

```
public class A {}
```

**B Module**

```
api project(':A')
```

**C Module**

```
implementation project(':B')
```

```
public class C {
  public void act() {
    new A() <-- compile error
  }
}
```

`implementation` 은 하위 의존을 숨겨주므로, 우리는 보호받아야할 하위 모듈에 대한 접근을 폐쇄할 수 있습니다. 그래서 저는 `어플리케이션 모듈 계층` 에서는 `implementation` 을 사용하고 그 외 계층에서는 `api` 방식을 사용하여 개발을 하였습니다.

### 4. README

다양한 Module 을 갖게 될 때 Naming 만으로는 모듈의 의도를 파악하기 힘듭니다. 모든 Module에 `README.md` 를 반드시 작성하시길 바랍니다.

저는 모든 모듈에

- 역할과 책임
- 실행방법(사용방법)
- 관례

등을 작성하고 있습니다. 자동화된 매커니즘만으로 제공될 수 없는 위의 이야기를 이 프로젝트를 개발하게 될 모든 개발자를 위해 친절하게 꼭 작성하시길 당부드립니다. README를 통해 더 원활한 개발환경을 만들 수 있습니다.


### 5. 확장 및 사용자 정의 여지 개방

하위 모듈에서 정의한 `Bean` 을 사용하는 모듈에서 확장하거나 변경하고 싶을 때가 있습니다. 그렇기 때문에 하위 모듈을 개발할 때도 필요한 설정에 한하여 여지를 개방해둘 수 있습니다.

이럴 때 활용할 수 있는 첫번째 방법은 `@ConditionalOnMissingBean` 입니다. `Spring Framework` 에서도 이 방식으로 많은 여지를 개방해주고 있습니다. `@ConditionalOnMissingBean` 은 `Bean` 이 없을 때 발동합니다. 이 어노테이션을 하위 모듈에서 설정해놓음으로서 상위 모듈에서 대체할 수 있는 여지를 줄 수 있습니다.

**하위 모듈**
```java
@Conficuration
public class Config {
  @Bean
  @ConditionalOnMissingBean
  public A a() {
    return new A("하위"); //로드되지 않음
  }
}
```

**상위 모듈**
```java
@Conficuration
public class Config {
  @Bean
  public A a() {
    return new A("상위"); //로드
  }
}
```

두번째 방법은 `Customizer` 방식입니다.

```java
@Bean
public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
    return jacksonObjectMapperBuilder -> jacksonObjectMapperBuilder.featuresToEnable(MapperFeature.DEFAULT_VIEW_INCLUSION);
}
```

`Bean` 으로 설정된 `Customizer`를 `Configuration` class가 주입받아 생성 과정에 관여를 합니다.

```java
@Bean
@ConditionalOnMissingBean
public Jackson2ObjectMapperBuilder jacksonObjectMapperBuilder(
		List<Jackson2ObjectMapperBuilderCustomizer> customizers) {
	Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();
	builder.applicationContext(this.applicationContext);
	customize(builder, customizers);
	return builder;
}

private void customize(Jackson2ObjectMapperBuilder builder,
		List<Jackson2ObjectMapperBuilderCustomizer> customizers) {
	for (Jackson2ObjectMapperBuilderCustomizer customizer : customizers) {
		customizer.customize(builder);
	}
}
```

위와 같은 방법으로 필요한 곳에 한하여 확장과 변경에 대한 여지까지 제공하면서 더 유연한 모듈이 될 수 있었습니다.

---

## 마무리

이렇게 멀티모듈 프로젝트를 설계한 경험을 작성해보았습니다. `배달의민족` 서비스를 개발하기 이전부터도 이러한 원칙을 점차 만들어나갔고 그 과정에 함께한 `배달의민족`의 프로젝트들이 `결제`, `주문`, `회원`, `프론트`, `검색` 프로젝트입니다. 재각기 서로 다른 비지니스를 가지고 있고 서로 다른 도메인의 성격, 인프라 자원을 가지고 있었기에 매번 많은 고민과 시도를 해야만 했습니다. 그러한 고민과 시도를 통해 얻게 된 경험을 정리해보았습니다.

아직 접해보지 못한 다양한 비지니스들이 많을 것이기에, 이 경험을 정리한 글을 정답이라고 말할 순 없을 것 입니다. 그러나 모듈에 대해 다시 한번 생각해보게 될 수 있고, 그 생각들의 기반이 되어 정답에 가까워질 수 있는 시작이라고 봐주시고 많은 피드백을 부탁드리겠습니다.

더 좋은 개발환경을 위해, 더 좋은 프로젝트를 위한 멀티 모듈 프로젝트 설계 이야기였습니다. 긴 글 읽어주셔서 감사합니다.
