---
layout:     post
title:      스프링캠프 2018-Consumer Driven Contract
author:     kingbbode
tags:       spring think
subtitle:   Consumer Driven Contract 기법을 활용한 마이크로서비스 API의 진화
category:  seminar
outlink: 0
---

요즘 기존 프로젝트의 일부를 새로운 프로젝트로 떼어내는 작업을 하고 있다. 노후된 프로젝트인 점과 내가 알고있는 프로젝트가 아니라는 점이 지옥을 맛보게 해주고 있다. 정리를 하다보니 기능이 중복된, 프로젝트의 성격 이상으로 많은 정보를 제공하는, 불필요하게 많은 정보를 요구하는 등 이상한 End Point들이 발견되었다. 아마 오랜시간 요구사항이 점점 늘어나면서 불가피하게, 혹은 요구했었지만 이제는 다르게 사용하는 API가 되었지 않을까 생각한다. 

정리를 위해 **이 API들을 사용하는 서비스들에 대해서 전부 조사를 해야했고, 굉장히 많은 시간을 소비해야 했다.** 정리를 하고난 후 보니 중복된 것, 사용되지 않는 것, 불필요한 요구, 제공 스펙, 그리고 그것들을 위한 테스트 코드들이 보였다. 현재와 미래의 모든 클라이언트의 필요를 충족하는 API 만들려고 시도했기 때문일 것이다.

> 일부 서비스 개발자는 현재 및 미래의 모든 클라이언트 요구 사항을 해결하기 위해 시도하는 서비스 API를 작성하여이 문제를 해결하려고 노력했습니다. 더 자주는 아니지만, 이러한 노력은 누락 된 요구 사항이 확인되고 예기치 않은 변경 사항이 요청되기 때문에 쓸데없는 것으로 판명됩니다. 클라이언트가 API를 사용하는 방법을 이해하는 서비스 소유자는 큰 이점이 있습니다. 이들은 서비스 변경으로 인해 클라이언트가 중단되는시기를 식별하는 클라이언트 위주의 테스트를 작성할 수 있습니다. 그러나 서비스 소유자가 이러한 테스트를 작성하면 클라이언트가 서비스를 사용하는 방법에 대한 부정확 한 해석이 될 수 있습니다. 고객 소유자는 서비스 소유자가 기대를 문서화하여 서비스 사용 방법을 이해하도록 도울 수 있지만 이러한 문서가 코드와 스키마에서 생성되어 변경 될 때마다 재생성되지 않으면 신속하게 구식이 될 수 있습니다. 당사자들이 정말로 필요로하는 것은 일련의 **자동화 된 통합 테스트**입니다. 

[출처 : servicedesignpatterns-Consumer-Driven Contracts](http://www.servicedesignpatterns.com/WebServiceEvolution/ConsumerDrivenContracts)

이런 상황에서 떠오른 것은 `CDC(Consumer Driven Contract)` 패턴을 소개하며, `Spring Cloud Contract` 라는 프로젝트를 소개해주었던 `Spring Camp 2018` 피포탈 김민석님의 세션이였다. 이 날 가장 재미있게 들었던 세션이였던 이 발표 후기가 보이지 않아 기억을 끄집어내어 정리해본다. 

---

# Consumer Driven Contract 기법을 활용한 마이크로서비스 API의 진화 ( Spring Cloud Contract )

**2018-04-21 pivotal 김민석님**

## 새로운 문제들

마이크로 서비스에서의 API 는 `Bounded Context` 로부터 생겨나고, `제공자(Producer)` 와 `소비자(Consumer)`의 합의를 통해 스펙이 결정되는 라이프사이클을 갖고 있으며, `Rest 기반의 Http 통신` 혹은 `PUB/SUB QUEUE 기반 Message 통신` 기반의 통신을 하고 있습니다.

그리고 애자일, MSA 시대를 맞으면서 아래와 같은 변화가 있습니다.

- Service가 점점 작아지고 많아짐
- Api contract 개선이 더 잦아짐(Agile)
- 다른 시간대(Timezone)의 팀간의 협업이 요구됨

점점 더 빠르게(자주) 변경되는 어플리케이션이 생기고 있고, 어플리케이션은 점점 더 작은 단위의 어플리케이션으로 분리되고 있습니다.

그리고 이로인해 생기고 있는 새로운 문제들은

- `소비자(Consumer)`에 영향없이 `제공자(Producer)` 서비스에 새로운 것을 추가하거나 삭제할 수 있을까?
- `소비자(Consumer)`가 서비스를 사용하고 있는지 `제공자(Producer)`가 알 수 있는가?
- `소비자(Consumer)`가 원하는 서비스를 제공하고 있는지 `제공자(Producer)`가 알 수 있는가?
- 얼마나 자주 배포할 수 있는가?

그리하여 안전한 API를 위하여 아래와 같은 진화를 위한 패턴들이 등장하였습니다.

- Single Message Argument
- Dataset Amendment
- Tolerant Reader
- Schema Versioning
- Extension Points
- Consumer-Driven Contracts

(이 발표에서 다루는 것은 이 중 `Consumer-Driven Contracts` 입니다.)

## 소비자 주도 계약 패턴(Consumer-Driven Contracts)

`CDC(Consumer-Driven Contracts)`는 `소비자(Consumer)` 의 요구사항 중심으로 `제공자(Producer)` 서비스를 진화시키기 위한 협업 패턴으로 **소비자(Consumer)와 `제공자(Producer)`의 협업이 많아지는 사상** 입니다.

- `제공자(Producer)`는 불필요한 서비스 개발을 줄일 수 있음 (make the right thing)
- `제공자(Producer)`는 개선에 대한 통찰을 얻을 수 있음
- Agile 실현을 위한 자동화를 가속

`CDC`가 성공하려면 `스펙 정의`, `문서화`, `테스트`, `스펙과 일치하는 테스트코드 유지`, `모니터링/제어`, `Gateway` 등이 필요합니다. 즉 서로 간의 커뮤니케이션이 핵심이 됩니다. 그래서 우리는 `apigee`, `aws api gateway`, `kong`, `swagger` 등 다양한 도구들을 사용하고 있습니다. 하지만 아직 부족한 것이 있습니다.

#### 내가 작성한 코드가 스펙과 일관성 있는가? 변화하는 API에 대해서 어떻게 대응할 수 있는가?

이러한 점을 만족 시키기 위해 우리는 테스트를 합니다.

**E2E(End to End)** 테스트전략은 확실하지만 실제 환경과 유사한 환경을 만들어 내야하며, 많은 시간과 테스트 자원을 소모하기 때문에 비효율적입니다.

그래서 **Mock** 을 활용하는 테스트 전략이 일반적입니다. 전체를 구성하지 않고 흉내를 내는 `Mock` 을 활용한 테스트는 작은 테스트 자원으로 빠른 피드백을 받을 수 있습니다. 그러나 **테스트가 통과하더라도 실제 서비스에서 실패** 할 수 있으며 **소비자가 요구하는 스펙과 일관성을 유지하기 어려운 것** 은 마찬가지 입니다. 테스트를 만들 때 임의의 케이스를 만들어 성공하게 하는데, 이 때 만들어진 케이스의 정합성이 맞는지 보장하는 메커니즘이 없으며, 스펙으로 정의해놓은 데이터 파일의 최신 관리가 어렵기 때문입니다.

결국 CDC를 성공시키기 위해서는 **Contract 정합성 유지** 와 **스펙과 일치하는 테스트코드** 가 필요합니다. 그리고 이를 해결할 수 있도록 도와주는 것이 `Spring Cloud Contract` 입니다.

## Spring Cloud Contract

### Spring Cloud Contract 목표

1. Contract를 공유하는 메커니즘을 제공

2. Contract를 적용할 때 자동화를 통해 확신을 주자



### Spring Cloud Contract Workflow

![workflow](/images/2018/SPRING-CAMP-CDC/workflow.png)

1. `소비자(Consumer)` 는 `Contract` 를 정의하여 `제공자(Producer)` 에게 요청합니다. (Pull Request)

2. `제공자(Producer)` 는 `Contract` 를 구현합니다. 

3. `제공자(Producer)` 의 `Contract` 구현으로 생성된 `stub.jar` 를 업로드합니다. (Nexus or Spring Cloud Eureka 등)

4. `소비자(Consumer)` 는 자신이 요청한 `Contract` 기반으로 구현된 `stub.jar` 를 다운로드 받아 테스트합니다. 

### Spring Cloud Contract 는 목표를 이루기 위해 아래와 같은 편의 기능들을 제공

#### API 스펙 지원

`Rest 기반의 Http 통신` 혹은 `PUB/SUB QUEUE 기반 Message 통신`을 모두 지원합니다.

#### Contract 작성 DSL 제공

정의하고 읽기 쉬운 형태로 `Contract` 작성이 가능하도록 DSl을 제공합니다. 이 DSL 은 `groovy` 이나 `yml` 로 작성 가능합니다.

#### Spring Cloud Contract Plugin

쉽게 사용할 수 있도록 `Maven`, `Gradle` Plugin 을 제공합니다.

- contract verifier : `제공자(Producer)`용
- contract stub runner : `소비자(Consumer)`용

#### 컨트랙트 공유 메커니즘 제공:자동으로 Mock 환경 생성

`제공자(Producer)`는 `Contract` 기반으로 자동으로 Mock 테스트를 수행할 수 있습니다.

#### 자동화된 최신화 유지:자동으로 Stub 생성

`제공자(Producer)` 가 `Contract` 기반으로 정의한 테스트 Mock을 제공하여, `소비자(Consumer)`가 테스트에서 사용 가능합니다. `application jar`도 제공하기 때문에 쉽게 Mock 어플리케이션 서버를 띄울 수도 있습니다.

#### Spring Cloud 연동

- Spring cloud를 연동하는 프로젝트 지원
- Spring cloud eureka에 stub 자동등록 지원

---

데모를 제외한 발표는 여기까지였다.

##### 더 자세한 내용은 제공 Slide 자료로! : https://www.slideshare.net/MinseokKim4/consumerdrivencontract-with-spring-cloud-contract-at-spring-camp-2018

##### 데모샘플 : https://github.com/myminseok/spring-cloud-contract-beer-sample

아래는 발표 이후 추가로 궁금해져 찾아본 내용이다.

---

### Document와 Spring Cloud Contract는 별게여야 하는가?

요즘 `Swagger`의 어노테이션 지옥을 맛본 후 `Spring Rest Docs`로 넘어오게 되었다. 문서 작업을 위해 조금의 번거로운 작업을 해줘야 하지만, 이러한 문서 자동화 도구를 통해 문서와 코드를 일치시킬 수 있다.

#### 그렇다면 스펙과 코드를 일치시키는 `Spring Cloud Contract`는 `Spring Rest Docs`와 어떻게 사용하여야 할까?

역시나 `Spring Cloud Contract`에서 지원을 해주고 있다. [Spring Cloud Contract with Rest Docs](http://cloud-samples.spring.io/spring-cloud-contract-samples/tutorials/rest_docs.html)에 잘 설명하고 있다.

`Rest Doccumentation` 테스트 코드에 Contract 를 끼워넣어 검증을 수행한다.

```java
.andDo(MockMvcRestDocumentation
    .document("shouldRejectABeerIfTooYoung", SpringCloudContractRestDocs.dslContract()));
```

### 이미 만들어진 RestDocs 에 Contract 를 끼워넣었는데, 역으로 Contract 기반으로 RestDocs를 생성할 순 없을까?

역시나 이미 진행 중이였다! 

[Spring Cloud Contract 이슈](https://github.com/spring-cloud/spring-cloud-contract/issues/613) 에 등록이 되어있었다. 이 이슈는 받아들여져 `2.0.0.RC2` 버전에 들어갔다. 빠른 시일 내에 정식버전에서 사용할 수 있지 않을까 생각한다.

## 마무리

요즘 세미나들에서 자사 제품 끼워팔기를 많이 보았다. 이 세션 역시 그렇지 않을까 우려하면서도 듣게 되었다. 

**약팔이 당하진 않을꺼야!**

그러나

![넘어감](/images/2018/SPRING-CAMP-CDC/kakaotalk.png)

넘어가버렸다. 요즘 같은 시대의 흐름에서 반드시 생기고 고민하게 되는 내용에 대한 솔루션이였기 때문이다.

모두가 `소비자(Consumer)` 이자 `제공자(Producer)` 가 되는 구조에서 도입은 쉽지 않을 것이다. 한쪽이 일방적으로 제공하는 것이 아닌, 서로 모두가 추구해야하기 때문이다. 쉽지 않지만 언젠가 반드시 필요한 기술이 아닐까 생각한다.
