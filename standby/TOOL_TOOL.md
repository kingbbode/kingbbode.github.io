내가 바라던 모니터링 환경!
==========================

내가 이상적으로 바라던 그런 모니터링 환경을 만나게 되었다.

![monitoring](/images/2018/TOOL/monitoring.png)

그러나 나는 이 툴들을 쓸 줄도, 어디까지 해주는지, 뭘 해주는지도 잘 모른다. 결국 이전에 하던 귀찬은 방법들을 찾게 된다.

비효율적이지만 익숙한 방법을 고수하는 것, 이게 바로 `기술부채` 구나!

그래서 내가 처음 접하게 된 모니터링 툴들을 정리해본다.

---

### 1. New Relic

`New Relic`은 `SaaS` 기반의 `APM(Application Performance Management)` 서비스를 제공하는 회사이다.

`New Relic` 의 다양한 서비스 중 내가 가장 잘 사용할 줄 알아야 할 서비스는 `APM`과 `INFRASTRUCTURE`다. `APM`은 어플리케이션에 대한 성능 모니터링을 제공한다. `INFRASTRUCTURE`는 서버에 대한 성능 모니터링을 제공한다.

예전에는 무료 버전이 있던 것 같지만, 홈페이지를 살펴보니 현재는 free tier(기간제)만 존재하는 것 같다. 전체적으로 New Relic이 비싸다는 글을 보았다. (사용할 수 있는 회사에 감사..)

#### UI

![newrelic_ui](/images/2018/TOOL/newrelic_ui.png)

New Relic은 하나의 전용 페이지를 제공하며, 각각의 상품들은 상단 탭으로서 존재한다.

다양한 시각 자료를 통해 다양한 통계 모니터링을 보여준다.

(위 그림에서 INFRASTRUCTURE 는 빠져있다. 옛날 버전 캡쳐본인듯..?)

![custom_dashboard](/images/2018/TOOL/custom_dashboard.png)

사용자 정의 DashBoard도 제공한다. 중점적으로 모니터링해야할 내용으로 DashBoard 를 구성하여 효율적으로 모니터링을 할 수 있도록 돕는다.

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

![newrelic_apm](/images/2018/TOOL/newrelicapm.png)

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

특정 환경에 대한 정책, 임계 값 설정 등으로 경고 알림을 받을 수 있다.

![newrelicalert](/images/2018/TOOL/newrelicalert.png)

다양한 메신저들과의 쉬운 연동을 제공하면서, Web Hook을 제공하기 때문 다른 곳에 연동에도 크게 어려움이 없을 것이다.

![newrelicslack](/images/2018/TOOL/newrelicslack.png)

빠른 알림 시스템을 제공하고, 해당 시간에 대한 통계 페이지를 제공해주어, 빠르게 대처할 수 있게 도와준다.

*이미지, 자료 등 출처 : https://docs.newrelic.com*

---

### 2. PINPOINT

PINPOINT는`Java`로 작성된 대규모 분산 시스템 용 `오픈 소스 APM`이다.

![naver_pinpoint](/images/2018/TOOL/naver_pinpoint.png)

`Naver`에서 만든 자랑스런 국산 오픈소스로 작년 말 `5000 스타`를 돌파했고, 현재도 계속 상승 중인 네이버의 대표적인 오픈 소스 프로젝트이다.

#### UI

![pinpointui](/images/2018/TOOL/pinpointui.png)

서비스의 흐름과 실시간 상태를 보기 굉장히 좋은 UI를 제공하고 있다.

#### APM

제공되는 대표적인 기능들은 아래와 같다.

-	ServerMap
-	Realtime Active Thread Chart
-	Request/Response Scatter Chart
-	CallStack
-	Inspector

#### 알람

![pinpointalarm](/images/2018/TOOL/pinpointalarm.png)

핀포인트 역시 특정 임계치를 설정하여 알람을 받는 방식이 가능하다.

그러나 실제 알람을 받기 위해서는 AlarmMessageSender 인터페이스의 구현체를 작성하여야 한다.

```Java
public interface AlarmMessageSender {
    void sendSms(AlarmChecker checker, int sequenceCount);
    void sendEmail(AlarmChecker checker, int sequenceCount);
}
```

[Github Source](https://github.com/naver/pinpoint/blob/master/web/src/main/java/com/navercorp/pinpoint/web/alarm/AlarmMessageSender.java) 에서 인터페이스를 보면, 현재 공식적으로는 sms와 email만을 지원하는 듯 하다. (위 안에 억지로 구현은 할 수 있겠지만..)

*이미지, 자료 등 출처 : https://naver.github.io/pinpoint*

---

### 3. logentries

logentries는 실시간 로그 관리, 분석 서비스이다. `Saas` 기반의 서비스로 `Agent`로부터 수집되는 `로그`에 대한 검색, 시각화, 분석을 제공한다.

![logentries](/images/2018/TOOL/logentries.png)

즉 이 서비스를 이용하면, 더 이상 모든 서버에 들어가 로그를 확인해야하는 노가다를 안할 수 있다.

-	실시간으로 수집되는 로그를 볼 수 있다.
-	특정 패턴에 대한 시각화를 할 수 있다.
-	수집된 로그에서 검색을 할 수 있다.

등의 아주 편리한 사용성을 제공한다. 이 편리한 사용성을 잘 사용하기 위해서는 효율적인 로그의 검색을 위해서 각 서비스에 맞게 로그의 구조에 대한 정책이 선행되는 것이 좋다고 한다. https://docs.logentries.com/docs/best-practices-logs

#### 알람

![logenttag](/images/2018/TOOL/logenttag.png)

특정 패턴에 대한 `Tag`를 만들 수 있는데, 이를 이용한 경고 알림이 가능하다.

![logentalert](/images/2018/TOOL/logentalert.png)

logentries 역시 다양한 메신저들과의 쉬운 연동을 제공하면서, Web Hook을 제공하기 때문 다른 곳에 연동에도 크게 어려움이 없을 것이다.

*이미지, 자료 등 출처 : https://docs.logentries.com*

---

### 마무리

로그를 찾기 위해 모든 서버를 들어가지 않는다. 서버 모니터링을 위해 16분할의 터미널을 띄워놓지 않는다. 서버의 상황에 따라 그때그때 여러가지 시도를 해볼 수 있다. 등등 위 시스템들이 제공해주는 여러 편리함들이 있다. (상황에 따라 필요한 경우도 있지만)

아직도 무슨 상황이 되면 터미널 키는 것을 먼저 생각하는 것이.. 적응이 덜 되었지만, 이제는 이것들이 무엇을 해주고 있는지, 날 어떻게 더 편하게 해주는지, 내가 뭘 이것들을 통해 봐야하는지를 대략 알 것 같다.

이러한 시스템들의 보호는 나의 시간을 절약시켜주고, 내가 더 개발에 집중할 수 있게 해주고 있다.

---
