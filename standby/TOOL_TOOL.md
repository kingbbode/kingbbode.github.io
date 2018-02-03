TOOL TOOL DEV TOOL!
===================

-	좋은 환경에서 더 좋은 퍼포먼스를 내기 위한 기술부채 청산기

내가 이상적으로 바라던 그런 개발환경을 만나게 되었다. 그러나 나는 이 툴들을 쓸 줄 모른다. 결국 이전에 하던 귀찬은 방법들을 찾게 된다.

비효율적이지만 익숙한 방법을 고수하는 것, 이게 바로 `기술부채` 구나!

그래서 내가 접하게 된 툴들을 정리해본다.

---

### New Relic APM

`New Relic`은 `SaaS` 기반의 `APM(Application Performance Management)` 서비스를 제공하는 회사이다.

`New Relic` 의 다양한 서비스 중 내가 가장 잘 사용할 줄 알아야 할 서비스는 `APM`과 `INFRASTRUCTURE`다. `APM`은 어플리케이션에 대한 성능 모니터링을 제공한다. `INFRASTRUCTURE`는 서버에 대한 성능 모니터링을 제공한다.

예전에는 무료 버전이 있던 것 같지만, 홈페이지를 살펴보니 현재는 free tier(기간제)만 존재하는 것 같다. 전체적으로 New Relic이 비싸다는 글을 보았다. (사용할 수 있는 회사에 감사..)

#### UI

![newrelic_ui](/images/2018/TOOL/newrelic_ui.png)

New Relic 의 UI는 훌륭하다. 다양한 시각 자료를 통해 모니터링을 위한 자료를 보여주며, 시간 범위 설정 등 모니터링을 위한 기능들을 제공하고 있다.

(위 그림에서 INFRASTRUCTURE 는 빠져있다. 옛날 버전 캡쳐본인듯..?)

![custom_dashboard](/images/2018/TOOL/custom_dashboard.png)

사용자 정의 DashBoard도 제공한다. 매우 유용한 기능인 것 같다.

#### New Relic INFRASTRUCTURE

`INFRASTRUCTURE`는 16년 말에 오픈한 서비스로 이전에는 `New Relic Server`란 이름으로 `APM`에 내장된 서비스로 제공되었다가, 독립된 상품으로 전환되었다.

```
New Relic Servers is available for existing users, but not for New Relic accounts created after November 16, 2016. If a master account has access to Servers, any new sub-accounts created after November 16, 2016 will also have Servers access.
```

![newrelic_inst](/images/2018/TOOL/newrelic_inst.png)

-	Health
-	DNS
-	Process
-	Traffic
-	Disk
-	CPU
-	Memory

등의 모니터링을 지원한다.

#### New Relic APM

##### Ruby, Java, Node.js, PHP, .NET, Python, Go 기반의 어플리케이션 모니터링을 지원한다.

-	응답 시간, 처리량 및 오류 비율
-	종단 간 트랜잭션 추적
-	어플리케이션 히스토그램
-	JVM Performance Analyzer
-	Service Map

등의 모니터링 기능을 지원한다.

##### Database 의 모니터링도 지원한다.

-	호출 응답 시간 및 처리량
-	호출에 소요 된 시간
-	쿼리 분석
-	Slow Query

등의 모니터링 기능을 지원한다.

#### 알람

*이미지, 자료 등 출처 : https://docs.newrelic.com*

### PINPOINT

### logentries

### Jira

### Confluence

### Upsource

### IntelliJ
