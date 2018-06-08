---
layout:     post
title:      스프링캠프 2018-Consumer Driven Contract
author:     kingbbode
tags:       spring think
subtitle:   Consumer Driven Contract 기법을 활용한 마이크로서비스 API의 진화
category:  seminar
outlink: 0
---

요즘 기존 프로젝트의 일부를 새로운 프로젝트로 떼어내는 작업을 하고 있습니다. 노후된 프로젝트인 점과 내가 알고있는 프로젝트가 아니라는 점이 지옥을 맛보게 해주고 있습니다. 정리를 하다보니 기능이 중복된, 프로젝트의 성격 이상으로 많은 정보를 제공하는, 불필요하게 많은 정보를 요구하는 End Point들이 발견되었습니다. 아마 오랜시간 요구사항이 점점 늘어나면서 불가피하게, 혹은 요구했었지만 이제는 다르게 사용하는 API가 되었지 않을까 생각합니다.

**결국 저는 이 API들을 사용하는 서비스들에 대해서 전부 조사를 해야했고, 굉장히 많은 시간을 소비했습니다.**

이런 상황에서 떠오른 것은 `CDC(Consumer Driven Contract)` 패턴을 소개하며, `Spring Cloud Contract` 라는 프로젝트를 소개해주었던 `Spring Camp 2018` 피포탈 김민석님의 세션이였습니다.

### "`소비자(Consumer)`가 서비스를 어떻게 사용하고 있는지 `제공자(Producer)`가 알 수 있다면! `제공자(Producer)`의 변경을 `소비자(Consumer)` 중심의 테스트로 확인할 수 있다면! 그리고 이것들이 자동화될 수 있다면!"

`Consumer Driven Contract 기법을 활용한 마이크로서비스 API의 진화` 라는 주제로 발표되었던 이 세션을 조금은 늦게 정리하여 봅니다.

# Consumer Driven Contract : Spring Cloud Contract

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

#### Spring Cloud Contract 목표

1. Contract를 공유하는 메커니즘을 제공

2. Contract를 적용할 때 자동화를 통해 확신을 주자

Spring Cloud Contract 는 목표를 이루기 위해 아래와 같은 기능들을 제공합니다.

#### 컨트랙트 공유 메커니즘 제공:자동으로 Mock 환경 생성

`제공자(Producer)`는 `Contract` 기반으로 자동으로 Mock 테스트를 수행할 수 있습니다. `Rest 기반의 Http 통신` 혹은 `PUB/SUB QUEUE 기반 Message 통신`을 모두 지원합니다.

#### 자동화된 최신화 유지:자동으로 Stub 생성

 `제공자(Producer)` 가 정의한 테스트 Mock을 제공하여, `소비자(Consumer)`가 테스트에서 사용 가능합니다.

 `제공자(Producer)`가 정의한 테스트를 기반으로 동작하는 application jar 제공하기 때문에 어플리케이션 서버를 띄워서도 `소비자(Consumer)`가 테스트할 수도 있습니다.

#### Contract 작성 DSL 제공

정의하고 읽기 쉬운 형태로 `Contract` 작성이 가능하도록 DSl을 제공합니다. 이 DSL 은 `groovy` 이나 `yml` 로 작성 가능합니다.

#### Spring Cloud Contract Plugin

쉽게 사용할 수 있도록 `Maven`, `Gradle` Plugin 을 제공합니다.

- contract verifier : `제공자(Producer)`용
- contract stub runner : `소비자(Consumer)`용

#### Spring Cloud 연동

- Spring cloud를 연동하는 프로젝트 지원
- Spring cloud eureka에 stub 자동등록 지원

#### Spring Cloud Contract Workflow

![workflow](./images/2018/SPRING-CAMP-CDC/workflow.jpeg)

(이것만 보아도 대략 내용을 알 수 있을 것 같습니다)

---

##### 데모샘플 : https://github.com/myminseok/spring-cloud-contract-beer-sample
