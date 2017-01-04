Spring - Open Session In View
=============================

`Spring Boot`를 사용하면서 Boot가 제공해주는 수많은 자동 설정은 우리에게 많은 편리함을 느끼게 합니다. **그러나 이 편리함 뒤에는 개발자에게 치명적인 함정이 있습니다.**

`Spring Boot`에서 지원해주는 `default` 설정들은 사실 이전에는 개발자들이 직접 작성해줘야 했던 설정이기 때문입니다. 저는 서블릿이 등장하고, 프레임워크가 등장하면서 겪었던 함정과 같다고 생각하고 있습니다.

본 내용은 [Open Session In View Pattern](http://pds19.egloos.com/pds/201106/28/18/Open_Session_In_View_Pattern.pdf)의 내용을 기반으로 Spring 사용에 집중하여 내용을 재정리하였습니다.(본 내용을 이해하는데 `레이어 아키텍쳐`를 알면 더 이해가 빠를 수 있습니다.)

이번 포스팅에서는 `Spring Boot`에서 `Default`로 설정해주고 있는 `Open Session In View Pattern`에 대하여 설명해보도록 하겠습니다.

해당 패턴은 객체-관계 매핑(ORM, Object-Relational Mapping)의 사용으로 등장하게 된 패턴이며, 예제를 구성하는데에 JPA(Java Persistence API)의 구현체 프레임워크인 Hibernate를 사용할 것 입니다.

시작 전 `Open Session In View`에 대해 미리 정리를 하자면, `Open Session In View`는 `영속성 컨텍스트를 뷰 렌더링이 끝나는 시점까지 개방한 상태로 유지하는 것`입니다.

아래 내용으로

-	먼저 몇 가지 알아야 할 내용을 먼저 설명하겠습니다.
-	그리고 어떤 이유로 이러한 패턴이 등장하게 되었는지를 예제로 살펴볼 것이고,
-	과거에는 어떻게 해결하였는지를 볼 것이며,
-	Spring Boot에서는 무엇을 해주고 있는지 알아보겠습니다.

영속성 컨텍스트
---------------

Hibernate는 도메인 레이어 객체들이 하부의 데이터 저장소와 영속성 메커니즘에 대해 알지 않아도 되는 `투명한 영속성(transparent persistence)`을 제공하는 `비침투적인(nonintrusive)` 프레임워크입니다.

영속성과 관련된 모든 관심사는 도메인 객체로부터 격리된 채 관리자 객체에 의해 투명하게 처리되는데, 이 관리자 객체의 기능을 담당하는 객체가 Hibernate의 `Session 객체`이며, `Session 객체`는 `영속성 컨텍스트`를 포함합니다.

영속성 컨텍스트를 이해하기 위해서는 `Transaction`을 이해해야 합니다. `Transaction`는 쪼갤 수 없는 업무처리의 단위로 작업의 원자 단위라고 보면 되며, 영속성 컨텍스트는 `Transaction`과 1:1로 연결됩니다.

하나의 `Transaction`동안 수정된 객체의 모든 상태는 영속성 컨텍스트 내에 저장되어, `Transaction`이 종료될 때 데이터 저장소에 동기화(flushing)됩니다. 따라서 하이버네이트 Session을 하나의 작업 단위 동안 생성 및 조회, 수정, 삭제되는 객체의 상태를 보관하는 일종의 캐시로 간주해도 무방합니다.(실제로 하이버네이트 Session을 “1 차 캐시”라고 부르기도 합니다)

> MemberService.Java

```java
@Transactional
public void moveTeam(Long memberIdx, Team team){
    Member member = memberRepository.findOne(memberIdx);
    member.moveTeam(team);
}
```

위 moveTeam 메서드가 실행될 때 @Transactional 어노테이션에 의해 자동으로 Session이 열리게 됩니다.

조회(findOne)를 통해 가져온 Member 객체는 영속성 콘텍스트에 캐쉬되고, `Transaction`동안 Member 객체의 변경 사항이 추적됩니다. 이 때 Member 객체는 영속성으 갖는다고 말합니다.

moveTeam에 의해 영속성을 갖는 Member 객체가 변경된 것을 영속성 컨텍스트가 감지하게 됩니다. `Transaction`이 종료되는 시점인 메서드 종료 시점에 영속성 컨텍스트가 캐시하고 있는 객체의 변경 상태를 저장소와 동기화하게 되며, 이때 update 쿼리가 자동으로 실행되게 됩니다.

영속성을 갖는 객체의 상태
-------------------------

영속성을 갖는 객체는

-	Persistence
-	Detached
-	Transient
-	Removed

위 4 가지 상태를 가집니다.

![출처:하이버네이트 완벽 가이드](../images/2016/2016_12_28_OPEN_SESSION_IN_VIEW/status.png)

[출처 : 하이버네이트 완벽 가이드]

**Persistence**

데이터베이스 식별자(주키)를 가지며 작업 단위 내에서 영속성에 의해 관리되는 상태

> Transaction 안에서 조회, 저장, 수정 등을 통해 가져온 영속성이 부여된 상태를 Persistence 상태라 하며, Transient 상태의 객체도 강제로 Persistence 상태로 전환시킬 수 있습니다.

**Detached**

데이터베이스 식별자를 가지지만 영속성 컨텍스트로부터 분리되어 더 이상 데이터베이스와의 동기화가 보장되지 않는 상태

> Session.close() 메소드는 Session 을 닫고 영속성 컨텍스트를 포함핚 모든 자원을 반환하며, 관리하에 있는 모든 영속 인스턴스를 Detached 상태로 변경시킵니다. 하이버네이트는 Detached 상태의 객체에 대해서는 변경사항을 추적하지 않으며, 따라서 데이터베이스와의 동기화 또한 수행하지 않습니다.

**Transient**

아무 상태도 가지지 않는 상태(무 상태를 정의하기 위한 상태)

> new 연산자를 사용하여 생성한 객체는 곧바로 Persistence 상태가 되지 않고 Transient 상태를 갖게 됩니다.

**Removed**

제거될 상태

> Transaction이 종료되는 시점에 삭제될 객체는 Removed 상태를 갖습니다.

지연 로딩(Lazy Loading)
-----------------------

Open Session In View Pattern의 등장을 설명하기 앞서 중요한 개념이 `지연 로딩(Lazy Loading)`입니다.

그리고 `지연 로딩`을 설명하기 앞서 알아야 할 개념은 JPA의 `지연 페치` 전략과 `프록시`의 개념입니다.

**지연 페치(Lazy Feth)**

지연 페치 전략이란 기본적으로 쿼리 수행과 관련된 객체를 로드하고 해당 객체와 연관 관계를 맺고 있는 객체나 컬렉션은 로드하지 않는다는 것을 의미합니다. JPA는 모든 ENTITY와 컬렉션에 대해 `지연 페치(Lazy Fetch)` 전략을 적용합니다.

**프록시**

`지연 페치` 전략에 따라 모든 연관 관계를 맺고 있는 객체와 컬렉션에는 실제 ENTITY 대신 실제 객체처럼 위장한 `프록시(Proxy)` 객체가 생성됩니다.

**프록시 객체의 지연 페치**

JPA는 `프록시` 객체가 `Fetch`되는 시점을 갖는 타입을 두 가지로 나누었고, 관계의 종류에 따라 기본으로 갖는 `Fetch` 전략을 다르게 설정해두었습니다.

-	즉시로딩(FetchType.EAGER)

	-	쿼리 실행 후 즉시 로딩
		-	`@ManyToOne`
		-	`@OneToOne`

-	지연로딩(FetchType.LAZY)

	-	실제로 사용되는 시점에 로딩
		-	`@OneToMany`
		-	`@ManyToMany`

대부분의 사람들이 `지연 페치 전략`과 `지연 로딩`을 같은 것으로 생각하고 있습니다. `즉시로딩(EAGER)`과 `지연로딩(LAZY)` 모두 `지연 페치 전략`의 페치 방법이며, 쿼리 실행 후 프록시 객체를 Fetch하는 시점을 구분합니다.

아래는 1:N 관계를 갖는 Team과 Member입니다.

![M-T-ERD](../images/2016/2016_12_28_OPEN_SESSION_IN_VIEW/mt_erd.png)

Team 도메인은 Member와 `@OneToMany`의 관계를 맺고 있습니다.

```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    private Long idx;

    @Column(nullable = false, length = 20)
    private String name;

    @OneToMany(mappedBy="team")
    private List<Member> members;
```

JPA의 기본 Select를 통하여 Team을 조회하였을 때, 연관관계를 맺고 있는 members List 객체는 지연페치의 대상이되어, Proxy 객체로 채워질 것 입니다.

![M-T-ERD](../images/2016/2016_12_28_OPEN_SESSION_IN_VIEW/mt_proxy.png)

`@OneToMany`의 Default는 LAZY이기 때문에 Team 객체를 통해 Member List를 사용하려고 할 때 `Fetch`가 되어 쿼리가 실행되게 됩니다.

```java
team.getMembers().iterator().next()
```

*getMembers를 통해 객체를 호출해왔을 때가 아닌 내부의 값을 호출할 때 Fetch된다는 것!*

Open Session In View Pattern의 등장
-----------------------------------

### Open Session In View Filter

### Open Session In View In Spring Boot
