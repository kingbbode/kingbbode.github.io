Spring 에서 @ConditionalOnClass, @ConditionalOnBean 사용할 때 주의할 점
=======================================================================

Spring Boot 기반의 자동 설정을 위한 AutoConfiguration 클래스를 만들다가 발생한 이슈를 작성합니다.

@Conditional 이란?
------------------

스프링4에서 도입된 어노테이션으로 조건부로 Bean을 스프링컨테이너에 등록하는 역할을 합니다. 이 어노테이션은 Condition Interface 사용하여 특정 조건부로 등록되도록 만들 수 있습니다. 그리고 현재의 스프링 프레임워크에서는 미리 정의된 Condition Interface 구현체를 가지고 있는 @Conditional 어노테이션의 파생 어노테이션들이 있습니다.

주의점을 적을 어노테이션은 아래 어노테이션들 입니다.

@ConditionalOnClass : 특정 Class 파일이 존재하면 Bean 을 등록. @ConditionalOnBean : 특정 Bean 이 존재하면 Bean 을 등록.

**두 어노테이션의 특징은 어노테이션 메타데이터로 Class 를 입력받는다는 것 입니다.**(두 어노테이션 말고도 더 있을 수도 있음)

```java
@ConditionalOnClass(Test.class)
...

@ConditionalOnBean(Test.class)
...
```

언제 주의해야 하는가?
---------------------

`@ConditionalOnClass` 가 작성되는 목적이기도 합니다.

`@ConditionalOnClass` 는 해당 Class 가

ArrayStoreException : TypeNotPresentExceptionProxy
--------------------------------------------------

### ArrayStoreException

can not load configuration(ArrayStoreException, TypeNotPresentExceptionProxy)

https://github.com/spring-projects/spring-boot/issues/9870

아까비...ㅜㅜ
