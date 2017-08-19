Quartz + Spring Batch 조합하기
==============================

`Zum` 에서 [BeyondJ2EE 김태기](https://beyondj2ee.wordpress.com/) 팀장님과 표준화 프로젝트를 진행하며, `Zum` 에서의 `Batch` 에 대한 표준을 작성하며 알게 된 `Quartz Framework` 의 매력과 직접 개발해본 `Spring 과의 조합 및 궁합`을 소개해보려고 합니다.

Quartz란?
---------

![logo](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/logo.png)

[Quartz Scheduler](http://www.quartz-scheduler.org/) 는 거의 모든 Java 응용 프로그램에 통합 할 수 있는 풍부한 기능 의 오픈 소스 작업 스케줄 라이브러리 라고 소개하고 있습니다.

```
스프링에서 Unix의 Cron 처럼 특정시간 혹은 몇분 혹은 몇시간마다 동작해는 스케쥴러를 구현해야 했다.

그래서 찾아보게 된게 Spring + Quartz Scheduler 조합의 활용이었다.

하지만 Spring 3.1 버전 부터는 Quartz Scheduler를 사용하지 않고도 Scheduler를 통한 Job을 실행할 수 있게 되었다.
```

*출처: http://javafactory.tistory.com/1386 [FreeLife의 저장소]*

라고 합니다. 위 말 처럼 우리는 이미 `Spring Framework` 안에서 `@Scheduled` 를 통해 과거 Quartz 에서 제공해주었던 `cron schedule` 기능 등을 사용하고 있습니다.

[FreeLife의 저장소 : What is Quartz](http://javafactory.tistory.com/272) 의 글을 보면 2001년 봄에 해당 라이브러리가 등장하였다고 소개하고 있는만큼 많은 경험과 경력을 가지신 개발자 분들은 `Quartz` 에 대해 많이 들어보셨을 것이라고 생각합니다.

> 그래서 제가 추측하건데, 위 글을 종합하였을 때 `Quartz` 는 과거 Cron 과 같은 스케줄링을 위해 많이 쓰이던 라이브러리였지만, 스프링 3.1 버전 후부터 그 기능을 `Spring Framework` 에서 제공해주기 때문에 조금은 `Spring Framework` 와 멀어졌던 라이브러리라고 생각합니다.

---

왜 Quartz 인가?
---------------

제가 Quartz 에 매력을 느끼게 된 가장 큰 이유 중 2가지만 설명하려 합니다.

### 첫번째는 `Clustering` 기능 입니다.

![cluster](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/cluster.png)

*출처 : http://www.quartz-scheduler.org/documentation/quartz-2.x/configuration/ConfigJDBCJobStoreClustering.html*

Schedule에 대한 Clustering 기능입니다.

Quartz 는 Database 기반으로 Scheduler Key 와 Trigger Key 를 통해 Schedule 을 Clustering 합니다.

즉, Database 를 기반으로 한 모든 Schedule에 대한 제어가 가능하다는 말 입니다.

Schedule 과 Trigger 를 가진 Batch Application 을 Grouping 할 수 있는 구조를 제공하여, 여러 Application을 가지고 있는 Server 들을 제어할 수 있으며, 그 Application 들을 Clustering 기능을 통해 적절히 분배하여 효율적으로 동작시킬 수 있으며, 서버 장애 상황에서 대응할 수도 있을 것 입니다.

간단히 예를 들자면,

특정 주제에 대해 여러 Job 을 가지고 있는 Batch Application 에서 Clustering이 없는 시스템에서 하나의 서버가 장애 상황에 발생했을 때

![non-clustering](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/non-clustering.png)

위와 같이 Job 이 진행 될 수 없지만, Clustering 기능이 있다면

![Clustering](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/clustering.png)

같은 Group 군 안에 있는 정상 서버의 Application 들이 나머지 Job 들을 적절히 분배하여 실행할 수 있으므로 효율적인 Batch 수행이 가능할 것이라고 생각 합니다.

위 예가 적절하지 않을 수도 있지만, Batch Application 에서도 Clustering 이 갖는 이점이 많겠다고 생각했습니다.

### 두번째는 Scheduling 을 위해 최적화된 구조를 제공한다는 점 입니다.

![quartzscheduler](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/quartzscheduler.jpg)

Quartz 는 `Scheduler`, `Trigger`, `JobDetail` 등의 구조를 통해 Scheduling 에 최적화된 구조를 제공합니다.

다양한 Java 라이브러리를 통해 해당 설정들을 쉽게 지원하기 위한 도구들도 많이 있습니다.

-	Scheduler : 스케줄러와 상호 작용하기위한 주요 API 입니다.
-	Trigger : 주어진 작업이 실행될 일정을 정의하는 구성 요소입니다.
-	JobDetail : 작업의 인스턴스를 정의하는 데 사용됩니다.

**현재는 Spring Boot에서 Quartz를 지원하진 않지만, Spring Boot 2.0에는 Quartz Starter Pack이 포함되어 있습니다.**

---

왜 Quartz + Spring Batch 조합인가?
----------------------------------

Quartz 가 그리 좋은데 왜 Spring Batch 가 필요한가? 라고 생각했을 수도 있지만, 역할을 나누어보면 굉장히 당연한 일 입니다.

제가 생각하는 `Quartz` 는 Scheduling 에 최적화 되어있지, Batch성 Job에 최적화 되어있지는 않습니다. 애초에 Project 이름이 `Quartz Scheduler` 이기도 합니다!

많지는 않지만, 여러 Batch Application을 개발하면서 Spring Batch에서 제공해주는

![spring-batch](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/spring-batch.png)

*출처: http://wiki.gurubee.net/pages/viewpage.action?pageId=4949437*

`Job` - `Step` - `Reader` - `Processor` - `Writer`

라는 구조가 정말 편하고 좋다고 생각했습니다.

Spring Boot에서도 Spring Batch에 대한 설정을 지원해줄 뿐만 아니라, Spring Batch를 위한 수많은 라이브러리가 있습니다.

제가 해야했던 것은 Batch Application 개발이였으니, Spring Batch는 포기할 수 없었습니다.

또한, Spring Batch 에서 Database 를 통해 제공하는 여러가지 정보들도 굉장히 유용히 쓸 수 있을 것 입니다.

그래서 제가 생각한 둘 간의 구조는,

![spring-batch](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/structure.png)

입니다!

Quartz vs Jenkins
-----------------

Quartz + Spring Batch 조합 개발하기
-----------------------------------
