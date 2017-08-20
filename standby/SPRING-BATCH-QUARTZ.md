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

즉, Database 를 기반으로 모든 Schedule에 대한 제어가 가능하다는 말 입니다.

Schedule 과 Trigger 를 가진 Batch Application 을 Grouping 할 수 있는 구조를 제공하여, 여러 Application을 가지고 있는 Server 들을 제어할 수 있으며, 그 Application 들을 Clustering 기능을 통해 적절히 분배하여 효율적으로 동작시킬 수 있으며, 서버 장애 상황에도 대응할 수도 있을 것 입니다.

간단히 예를 들자면,

특정 주제에 대해 여러 Job 을 가지고 있는 Batch Application이 Clustering이 없는 시스템에서 하나의 서버가 장애 상황에 발생했을 때

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

**현재는 Spring Boot에서 Quartz를 지원하진 않지만, 곧 나오게 될 Spring Boot 2.0에는 Quartz Starter Pack이 포함되어 있습니다.**

---

왜 Quartz + Spring Batch 조합인가?
----------------------------------

`Quartz` 가 그리 좋은데 왜 `Spring Batch` 가 필요한가? 라고 생각했을 수도 있지만, 역할을 나누어보면 굉장히 당연한 일 입니다.

제가 생각하는 `Quartz` 는 Scheduling 에 최적화 되어있을 뿐, Batch성 Job에 최적화 되어있지는 않습니다. 애초에 Project 이름이 `Quartz Scheduler` 이기도 합니다!

많지는 않지만, 여러 Batch Application을 개발하면서 Spring Batch에서 제공해주는

![spring-batch](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/spring-batch.png)

*출처: http://wiki.gurubee.net/pages/viewpage.action?pageId=4949437*

`Job` - `Step` - `Reader` - `Processor` - `Writer`

라는 구조가 정말 편하고 좋다고 생각했습니다.

Spring Boot에서도 Spring Batch에 대한 설정을 지원해줄 뿐만 아니라, Spring Batch를 위한 수많은 라이브러리가 있습니다.

제가 해야했던 것은 Batch Application 개발이였으니, Spring Batch는 포기할 수 없었습니다.

또한, Spring Batch 에서 Database 를 통해 제공하는 여러가지 정보들도 굉장히 유용히 쓸 수 있을 것 입니다.

그래서 제가 생각한 `Quartz 와 Spring Batch` 의 구조는,

![spring-batch](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/structure.png)

입니다!

Quartz vs Jenkins
-----------------

`Job Luncher` 역할을 하는 다른 도구들과 비교를 해보았습니다.

![job luncher](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/vsluncher.png)

그리고 그 중에서 제 주변에서는 Batch 를 수행할 때 가장 많이 쓰고 있다고 하는 `Jenkins` 와 비교하여 `Quartz` 의 장단점을 적어보겠습니다.

### Quartz 장점

-	Database 를 기반으로 Schedule 인스턴스들 간의 Clustering 이 가능합니다.

-	Application 이 구동된 상태를 유지하기 때문에, 비교적 정확한 시간에 Job 수행이 가능합니다.

### Quartz 단점

-	UI Admin 을 제공하지 않습니다.

### History 기능

**`Quartz` 자체는 Schedule 실행에 대한 History를 보관하지 않습니다.**

그럼 Job 에 대한 History는 어떻게?

![spring-batch-erd](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/spring-batch-erd.png)

바로 `Spring Batch` 에서 제공해주는 기능을 사용하면 됩니다. Spring Batch 는 다양한 테이블들로 작업에 대한 기록을 남깁니다.

각 Job 과 Step 에 대한 꽤 상세한 히스토리를 알 수 있습니다.

> `Jenkins` 를 사용하면 Job 에 대한 History 를 볼 수 있다라고 이야기를 하시는 분들이 계셨는데, `Jenkins` 를 사용한다고 해도 `Spring Batch` Job History 는 반드시 봐야한다고 생각합니다. 사실 우리가 알고 싶은 건 Job 에 대한 History 지 Scheduling 에 대한 History가 아니기 때문입니다.

### 결론

제가 생각하는 결론은 `Jenkins` 보다는 `Quartz` 입니다.

#### UI 가 제공된다는 것은 분명히 굉장히 크지만,

반론을 해보자면

1.	`Jenkins` 를 사용한다고 해도 `Spring Batch` History 를 모니터링하기 위한 도구가 필요합니다. 즉, 어차피 Jenkins 만으로는 안된다는 것 입니다.

2.	관계가 복잡하지 않은 테이블 3-4개를 컨트롤 하는 Application 을 만드는게 어렵진 않을 것이라고 생각합니다. 어느 정도 Quartz Table 들에 대한 사전지식이 있어야겠지만, 너무 간단해서 학습비용은 적을 것 입니다. (그래도 만들어져 있는 것이 없고, 누군가 만들어야 한다는게 크지만.. 그 누군가가 제가 될 듯 합니다.)

#### Quartz 가 갖는 장점인,

Clustering 기능과 초기 구동 속도가 필요없는 정확한 시간의 Job 수행도 매력적이고 필요한 기능입니다.

#### Jenkins는 CI가 목적인 툴이라는 것도,

제가 `Jenkins` 를 Batch Job Luncher로서 꺼려하는 굉장히 큰 이유입니다.

Jenkins가 CI 를 목적으로 하다보니, Batch Scheduling 을 위한 기능도 다 가지고 있게 되었고, 그렇다보니 편하기 때문에 `Jenkins` 를 유두리 있게 사용하는 것도 센스있고 좋은 시도인 것은 분명합니다.

그러나 저는 이건 분명 `개발부채` 라고 생각합니다. `Jenkins` 가 영원할지 아닐지는 아무도 모르고, 굉장히 엄청난 CI 툴이 나올 수도 있고 등등 만에 하나 CI 툴로 `Jenkins`를 사용 안하게 되었을 때?

Batch 를 위하여 Jenkins 시스템을 남겨둘 것인가. 새로운 CI 툴에도 Batch Application 을 컨트롤할 수 있는지를 체크할 것인가.

뭔가 목적에 맞지 않고 억지로 껴맞추어진 느낌이라면, 결국엔 언젠가 값을 치루어야할 `개발부채` 일 것 입니다.

#### 그래서 제 결론은 Jenkins 보단 Quartz 입니다.

---

Quartz + Spring Batch 조합
--------------------------

**Quartz 개발과 Spring Batch 개발에 대한 내용은 생략하겠습니다.**

**아래 내용에서는 Quartz와 Spring Batch의 연동을 위한 핵심 내용만 기록할 것이며, 그 외의 설정들은 [Github](https://github.com/kingbbode/spring-batch-quartz) 소스를 참고 부탁드립니다.**

전반적인 Spring Batch와 Quartz 조합 개발은 [javacodegeeks:Quartz Spring Batch Example](https://examples.javacodegeeks.com/enterprise-java/spring/batch/quartz-spring-batch-example/) 을 참고하시면 됩니다.

위 내용은 매우 잘 정리되었지만, Database 기반으로 `Clustering` 환경을 구성하기에는 어려운 부분이 있습니다. 그 부분을 고쳐가며, 보다 안정적인 환경으로 개발을 해보겠습니다.

### QuartzJobLauncher

Quartz 와 Spring Batch 를 구성함에 **핵심** 은 Quartz의 Job(Spring Batch Job이 아닌.. 둘 다 Job이라 조금 혼돈이 올 수 있습니다)에서 사용되는 Class가 단 하나라는 것 입니다. 그 Class 는 `QuartzJobLauncher` 라는 파라미터를 기반으로 Spring Batch의 Job을 구동시키는 역할을 합니다.

![springquartzbatch](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/springquartz.png)

기존 Spring Framework + Quartz 가 위와 같고,

![springquartzbatch](/images/2017/2017-08-20-SPRING-BATCH-QUARTZ/springquartzbatch.png)

Spring Framework + Quartz + Spring Batch 조합은 위와 같습니다.

#### 문제점

[javacodegeeks:Quartz Spring Batch Example](https://examples.javacodegeeks.com/enterprise-java/spring/batch/quartz-spring-batch-example/) 의 `QuartzJobLauncher` Class가 Database 기반에서 효율적인 못한 문제점은 아래 부분입니다.

```java
public class QuartzJobLauncher extends QuartzJobBean {
    private String jobName;
    private JobLauncher jobLauncher;
    private JobLocator jobLocator;

```

```java
@Bean
public JobDetailFactoryBean jobDetailFactoryBean() {
    JobDetailFactoryBean factory = new JobDetailFactoryBean();
    factory.setJobClass(QuartzJobLauncher.class);
    Map map = new HashMap();
    map.put("jobName", "fxmarket_prices_etl_job");
    map.put("jobLauncher", jobLauncher);
    map.put("jobLocator", jobLocator);
    factory.setJobDataAsMap(map);
    factory.setGroup("etl_group");
    factory.setName("etl_job");
    return factory;
}
```

Spring Batch Job을 실행 시키기 위한 `JobLauncher`와 `JobLocator` 를 `QuartzJobBean`를 이용하여, Parameter 기반으로 주입받고 있습니다. 이것이 문제가 되는 것은 Database를 사용할 때는 Parameter가 데이터베이스의 필드에 저장되야 하기 때문입니다.

그래서 `QuartzJobBean`를 사용하는데신, 필요한 Bean의 Injection을 Spring에게 위임하여야 합니다.

#### AutowireCapableBeanJobFactory

> BatchConfiguration.java

```java
@Bean
public JobFactory jobFactory(AutowireCapableBeanFactory beanFactory) {
    return new SpringBeanJobFactory(){
        @Override
        protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {
            Object job = super.createJobInstance(bundle);
            beanFactory.autowireBean(job);
            return job;
        }
    };
}
```

`Quartz` Job Class 의 auto-wiring 을 지원하는 JobFactory를 만들어 줍니다. `Spring Boot 2.0` 에서는 `AutowireCapableBeanJobFactory` 라는 이름의 Class로 제공될 예정입니다. 이 Class를 통해 앞으로 Quartz Job 클래스가 만들어질 때는 Quartz 내부 로직이 아닌 Spring을 통해 필요한 의존성을 주입할 수 있습니다.

`QuartzJobLauncher`도 다시 작성하게 되면

```java
public class BatchJobExecutor implements org.quartz.Job {
    @Autowired
    private JobLocator jobLocator;

    @Autowired
    private JobLauncher jobLauncher;
```

우리가 잘 알고 있는 `@Autowire` 를 통해 Injection 받을 수 있게 작성합니다.

### ADMIN UI를 만들기 위해 필요한 것

아직 저도 ADMIN UI를 직접 만들진 않았지만, 여기서는 Quartz Table 과 Spring Batch Table 을 어떻게 연결할 것인지 고민했던 내용을 기록합니다.

일단 이 내용이 없어도 가능한 것을 먼저 보겠습니다.

-	Spring Batch Job 모니터링 : 테이블 정보만 출력해주면 되므로..
-	Quartz Schedule Control : 테이블 정보만 업데이트해주면 되므로..

그리고 현재 상황에서는 불가능한 것이 Spring Batch Job이 어떤 Schedule Instance와 Trigger에 의해서 실행되었는지 모니터링 입니다.

불가능한 이유는 현재 Spring Batch Job 과 Quartz Schedule 간의 연관관계를 기록한 내용이 없기 때문입니다.

그래서 저는 이렇게 해결하려고 합니다.

```java

@Override
    public void execute(JobExecutionContext context) throws JobExecutionException {

...

JobParameters jobParameters = new JobParametersBuilder(jobName)
.addString(JOB_PARAMETERS_QUARTZ_KEY,
  context.getScheduler().getSchedulerName()
          + TOKEN + context.getJobDetail().getKey().getGroup()
          + TOKEN + context.getJobDetail().getKey().getName())
.addLong("timestamp", System.currentTimeMillis())
      .toJobParameters();

...

jobLauncher.run(jobLocator.getJob(jobName), jobParameters);
```

`Quartz` Job을 만들면 주입받는 Quartz `JobExecutionContext`을 통해 Spring Batch Job으로 Schedule에 대한 정보를 Parameter로 공급하는 것 입니다.

이렇게 되면 어떤 형태로든, Quartz와 Spring Batch 간의 관계를 찾을 수 있지 않을까 합니다!
