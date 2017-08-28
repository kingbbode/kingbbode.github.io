Spring 에서 @ConditionalOnClass, @ConditionalOnBean 사용할 때 주의할 점
=======================================================================

Spring Boot 기반의 자동 설정을 위한 AutoConfiguration 클래스를 만들다가 발생한 이슈를 작성합니다.

주의할 점을 보기 전 알아야 할 것!
---------------------------------

### @Conditional 이란?

스프링4에서 도입된 어노테이션으로 조건부로 Bean을 스프링컨테이너에 등록하는 역할을 합니다. 이 어노테이션은 Condition Interface 사용하여 특정 조건부로 등록되도록 만들 수 있습니다. 그리고 현재의 스프링 프레임워크에서는 미리 정의된 Condition Interface 구현체를 가지고 있는 @Conditional 어노테이션의 파생 어노테이션들이 있습니다.

주의점을 적을 어노테이션은 아래 어노테이션들 입니다.

-	@ConditionalOnClass : 특정 Class 파일이 존재하면 Bean 을 등록.

-	@ConditionalOnBean : 특정 Bean 이 존재하면 Bean 을 등록.

**두 어노테이션의 특징은 어노테이션 메타데이터로 Class 를 입력받는다는 것 입니다.** (두 어노테이션 말고도 더 있을 수도 있음)

#### Class 를 직접 입력받는 ConditionalOn* 어노테이션을 주의하세요!!

```java
@ConditionalOnClass(Test.class)
...

@ConditionalOnBean(Test.class)
...
```

### @ConditionalOnClass 을 왜 쓰는지를 집고 넘어가자!

`@ConditionalOnClass` 는 해당 Class 가 있는지를 검사하는 조건 어노테이션 입니다.

Class 가 있는지 왜 검사를 할지 생각하면 답이 나옵니다!

#### 1. 이 어노테이션이 작성되는 클래스는 외부 라이브러리로 제공되는 프로젝트 이다.

하나의 독립 실행형 프로젝트의 Configuration 에 ConditionalOnClass 작성할 일은 없습니다.

**일단 의존성을 독립 실행형 프로젝트에서 이미 확정 짖기 때문에, 클래스가 없을 일이 없습니다.**

그리고, **독립 실행형 프로젝트에서 ConditionalOnClass에 Class를 지정하는데, Class 가 없다면 Compile Error로 빌드부터 문제입니다.**

#### 2. 이 어노테이션이 작성되는 클래스는 의존성에 대한 제어권을 갖지 않습니다.

해당 어노테이션이 작성된다는 것은 **해당 클래스가 사용되는 프로젝트에서 해당 Class 가 있을 수도 있고 없을 수도 있다는 의미가 됩니다.**

이것은 해당 클래스가 작성되는 프로젝트는 해당 프로젝트를 참조하는 프로젝트의 의존성에 영향을 주지 않는다는 말 입니다.

> Spring Boot 의 AutoConfiguration 을 살펴보시면, 모든 선택적 의존성에 Dependency Optional True 가 붙어 있는 것을 확인할 수 있습니다. Gradle 에서는 compileOnly 를 사용할 수 있습니다. 대부분의 선택적 의존성들은 Starter Pack 에 의하여 주입됩니다.

---

@ConditionalOn* 어노테이션에서 지정한 Class 가 없을 때 주의해야 할 점 입니다!
-----------------------------------------------------------------------------

> @ConditionalOnClass 를 제외한 다른 @ConditionalOn* 어노테이션에서 Class 가 없는 상황은 ConditionalOnClass 자체가 작성되는 것이 누락되었을 가능성이 있으며, 아래 설명드릴 주의사항과 해결 방법이 동일하므로 모든 @ConditionalOn* 어노테이션에서 동일하므로 ! @ConditionalOnClass 어노테이션을 중심으로 작성합니다.

**ConditionalOnClass 가 Class 가 없을 때를 위한 조건 어노테이션인데, 문제가 있다니! 저도 어이가 없었지만 사실 입니다.**

### Method 에 ConditionalOnClass를 사용할 때 Exception!

```java
//TestBean 클래스가 없을 때
@Configuration
public class TestConfiguration {
  @Bean
  @ConditionalOnClass(TestBeab.class)
  public TestBean testBean() {
    return new TestBean();
  }
}
```

위 코드는 `ClassNotFoundException` 이 발생합니다.

```java
//TestBean 클래스는 있고 Test 클래스가 없을 때
@Configuration
public class TestConfiguration {
  @Bean
  @ConditionalOnClass(Test.class)
  public TestBean testBean() {
    return new TestBean();
  }
}
```

위 코드는 `ArrayStoreException` ( `TypeNotPresentExceptionProxy` ) 이 발생합니다.

### 왜 Exception이 발생하는가?

`ConfigurationClassPostProcessor` 의 동작과정을 이해하면 답은 간단합니다. `ConfigurationClassPostProcessor` Class 는 @Configuration 어노테이션이 적용된 빈 객체에서 @Bean 어노테이션이 적용된 메서드로부터 빈 객체를 가져와 스프링 컨테이너에 등록하는 역할을 합니다.

`ConfigurationClassPostProcessor` 는

1.	`ConfigurationClassParser`를 통해 먼저 Class 의 생성 여부를 Conditional 어노테이션들을 포함한 특수항 과정을 통해 정의합니다.

2.	위 과정을 통과한 Configuration Class 는 `ConfigurationClassEnhancerConfiguration` 를 통해 Enhance 라는 ByteCode를 읽어들이는 과정을 거쳐 Class 를 로드하게 됩니다.

위 예시에서 작성하였던 `Exception` 은 모두 이 과정에서 발생합니다.

#### ClassNotFoundException

먼저 말씀드리자면, 이 Enhance 과정에서 Annotation 내부에 정의된 Class는 문제가 되지 않습니다.

이 Exception 은 필드를 해석하는 과정에서 알 수 없는 Class 가 존재하기 때문에 발생합니다. 우리가 Compile 할 때 보는 Exception 과 같은 Exception 입니다.

#### ArrayStoreException - TypeNotPresentExceptionProxy

이 Exception 은 어노테이션에 정의해놓은 존재하지 않는 Class 때문에 발생합니다.

!? 위에서 제가 분명 enhance 과정에서 annotaion 은 문제가 되지 않는다고 했는데!?

더 자세히 들어가서 보면 이 Exception 은 Reflection 의 isAnnotationPresent 때문에 발생합니다.

`ConfigurationClassPostProcessor` 는 Class 를 enhance 하고 난 후 Bean 을 등록하기 위해 Method 를 enhance 하는 과정을 거칩니다. 이 때 해당 Method 가 Bean 객체를 등록하기 위한 Method 인지를 확인하기 위하여 `isAnnotationPresent(Bean.class)` 를 사용합니다.

그리고 `isAnnotationPresent` 은 해당 Method 의 전체 정보를 조사하게 되고 그 과정에서 `ArrayStoreException` 이 발생하게 되는 것 입니다.

#### ConfigurationClassPostProcessor 가 원인이 아닌 사례도 있습니다.

Configuration Class 자체가 WebApplicationInitializer 를 implements 하여 SpringServletContainerInitializer 가 Class 전체를 조사하여 위와 같은 Exception 이 발생한 사례도 있습니다.

[github.com/spring-projects/spring-boot commit : 2b7bf3e73359e0b5d7e31b899f638df2c9d30c43 ](https://github.com/spring-projects/spring-boot/commit/31874090b8e9b55ff6edfefdaac889a1a5188825)

### 해결방법

#### 1. Configruation Class 내부 코드에서 사용될 선택적 의존성 Class 는 반드시 Class 에 작성

```java
import com.external.lib.TestBean;

@Configuration
@ConditionalOnClass(TestBean.class)
public class TestConfiguration {
  @Bean
  public TestBean testBean() {
    return new TestBean();
  }
}
```

#### 2. ConditionalOn* 에서 사용될 Class가 선택적 의존성 Class이면 Class 에 작성

전체 설정에 동일하게 적용될 수 있는 조건이라면 Class 레벨로 옮기는 방법이 있습니다.

```java
import com.mysource.TestBean;
import com.external.lib.TestCondition;

@Configuration
@ConditionalOnClass(TestCondition.class)
public class TestConfiguration {
  @Bean
  public TestBean testBean() {
    return new TestBean();
  }
}
```

전체 설정에 동일하게 적용될 수 없다면, 내부 혹은 외부 Class 로 분리하는 것을 생각해볼 수도 있습니다.

```java
import com.mysource.TestBeanA;
import com.external.lib.TestConditionA;

@Configuration
@ConditionalOnClass(TestCondition.class)
public class TestAConfiguration {
  @Bean
  public TestBeanA testBeanA() {
    return new TestBeanA();
  }
}

import com.mysource.TestBeanB;
import com.external.lib.TestConditionB;

@Configuration
@ConditionalOnClass(TestConditionB.class)
public class TestBConfiguration {
  @Bean
  public TestBeanB testBeanB() {
    return new TestBeanB();
  }
}
```

#### 3. Class 지정 대신 String 을 사용

Class를 찾지 못하는 것만 우회하면 되므로, String 으로 작성하는 방법이 있습니다. 실제 많은 스프링 부트 이슈에서 이 방법으로 해결을 하고 있는 것을 확인했습니다.

```java
import com.mysource.TestBeanA;
import com.mysource.TestBeanB;

@Configuration
public class TestConfiguration {
  @Bean
  @ConditionalOnClass(name = "com.external.lib.TestConditionA")
  public TestBeanA testBeanA() {
    return new TestBeanA();
  }

  @Bean
  @ConditionalOnClass(name = "com.external.lib.TestConditionB")
  public TestBeanB testBeanB() {
    return new TestBeanB();
  }
}
```

---

마무리
------

발생한 이슈 해결을 글로 옮기면서 더 많은 것을 배울 수 있었습니다.

위의 내용을 종합해보았을 때 Class 를 받는 ConditionalOn* 은 Method 레벨에서는 반드시 String 을 작성하여야 하기 때문에 Annotation 이 Method Target을 지원하지 않고 Method Target 을 지원하는 Condition 은 따로 분리되야 한다고 생각했습니다.

이슈 재기 해야겠다! 라고 생각했지만,, 딱 한달 전에 피보탈 개발자가 이슈를 등록해놓았습니다 ㅜㅜ

![late](/images/2017/2017-08-26-SPRING-CONDITIONAL/late.png)

https://github.com/spring-projects/spring-boot/issues/9870

그래서 댓글로 다리만 살짝 걸치며,, 마무리..

현재는 여러 서비스들에서 자사 라이브러리를 Autu Configuration 화 시키는 사례가 잘 없지만, 점점 부트가 잘 보급된다면 언젠가 반드시 만나게 될 이슈라고 생각하여 이슈를 정리하여 작성합니다.
