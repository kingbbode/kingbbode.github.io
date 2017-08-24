---
layout:     post
title:      Spring Boot 와 Properties(or Yaml) Environment
author:     kingbbode
tags:       web spring boot springboot
subtitle:   Spring의 PropertySource 의 Yaml 미지원과 ConfigurationProeprties 의 locations Deprecated 의 배경을 알아보자!.
category:  posts
outlink: 0
---

Spring Boot 와 Properties(or Yaml) Environment
==============================================

Spring Boot 에서 properties 설정에 대한 깨달음을 얻어 정리하고자 글을 작성합니다.

몇 시간 전까지만 해도 이 글은 `@PropertySource Yaml 사용하기` 라는 글로 작성될 뻔 하였던 글 입니다.

제가 처음 위의 주제로 글을 작성하기로 마음 먹은 이유는

-	@PropertySource 의 Yaml 미지원
-	@ConfigurationProeprties 의 locations Deprecated

때문입니다.

까려고 찾아보다가, 내가 잘못 알았구나 하는 깨달음에 바로 글을 작성합니다.

---

### 발단

저의 10번째 블로깅이였던 [스프링 부트, YAML 적용](http://kingbbode.tistory.com/10) 이라는 블로그에서 소개하였던 `@ConfigurationProeprties` 의 `locations` 이 Spring Boot 1.4 를 이후로 Deprecated 되었습니다.

왜인지 알아보지도 않았었고, 그러려니 하다가

창천향로님의 멋진 버그 트래킹 일지인 [기억보단 기록을 : Spring OAuth + Spring Session시 HTTP URL must not be null 발생 원인 및 해결](http://jojoldu.tistory.com/171) 을 보고 `@PropertySource` 가 `Yaml` 을 지원하면 참 좋을텐데라는 생각을 해보았습니다.

![jira](/images/2017/2017-08-21-SPRING-BOOT-PROPERTIES/jira.png)

그래서 검색하다가 Spring Jira 를 보았고 짧은 영어 실력으로 잘못 해석하여 설레발 치다가 창피를 당했습니다..(제목 말고 댓글 중에 그런 부분이 정말 있습니다..;;)

![sulebal](/images/2017/2017-08-21-SPRING-BOOT-PROPERTIES/sulebal.png)

창피함을 극복하기 위해 `@PropertySource`에 있는 `Factory`를 이용하여 Yaml을 지원하도록 만들어보았습니다.

**해당 문서 맨 밑에 작성 코드는 공유합니다.**

코드를 작서해보고 느낀 것은 생각보다 너무 간단하다는 것 입니다...

아무리 생각해도 그들이 이걸 만들기 어려워서 안만들었을리가 없고,,

https://jira.spring.io/browse/SPR-13912

https://github.com/spring-projects/spring-boot/issues/6220

등등

관련된 이슈들을 다시 살펴보았습니다!

---

### 1. 처음에는 답정너인줄 알았음

처음에는 이슈들을 보면서 `무슨 말을 해도 답은 정해져 있다` 의 느낌을 받았습니다.

-	spring.config.location, spring.config.name
-	SpringApplicationBuilder
-	EnvironmentPostProcessor

`ConfigurationProeprties` 의 `locations` Deprecated 와 `@PropertySource` 의 Yaml 미지원에도 거의 비슷한 말만 계속..!

### 2. 나는 왜 `@PropertySource` 와 `@ConfigurationProeprties` 의 location 지정 기능에 집착을 했나?

*(@ConfigurationProeprties 의 location 기능이 @PropertySource 와 거의 유사하므로, @PropertySource 를 기준으로 작성)*

#### 보안을 위한 파일 분리

민감한 데이터나 정보를 저장소에 보관할 수는 없다는 이유가 있습니다.

```java
@PropertySource(value = {
    "classpath:/properties/example.properties",
    "file:/data/properties/example.properties"
})
```

`PropertySource` 의 특성상 나중에 로드된 파일로 설정을 `Override` 할 수 있습니다.

#### 환경에 따라 다른 값을 주입

```java
@PropertySource(value = {
    "classpath:/properties/example.properties",
    "classpath:/properties/example-${spring.profiles.active:default}.properties"
})
```

마찬가지로 설정이 `Override` 되는 것을 이용하여 프로필별로 다른 값을 주입받을 수도 있을 것 입니다.

#### 그냥 명시적으로 분리

```java
@PropertySource(value = {"classpath:/properties/foo.properties"})
class FooConfig {
  @ConfigurationProeprties(prefix="foo")
  Foo foo() {
    ...
  }
}

@PropertySource(value = {"classpath:/properties/bar.properties"})
class BarConfig {
  @ConfigurationProeprties(prefix="bar")
  Bar bar() {
    ...
  }
}
```

명시적인 설정을 선호하여 이렇게 분리할 수도 있을 것 입니다.

위의 내용들은 그동안 자주 보았고, 해보았던 설정들입니다. 이 외에도 많은 경우가 있을 수도 있지만, 제 기준으로는 일단 3가지를 예로 작성했습니다.

### 3. Spring Boot 는 `@PropertySource` 를 권장하지 않으며, `@ConfigurationProeprties` 의 locations 는 Deprecated 시켰다.

이슈를 보다가 Spring Boot 가 `@PropertySource` 를 권장하지 않는다는 것을 처음 알게 되었습니다.

```
using @PropertySource is definitely something that we do not recommend.

(@PropertySource를 사용하는 것은 우리가 권장하지 않는 것입니다.)

Easy doesn't necessarily means correct. In the context of Spring Boot, the whole use of @PropertySource (be it for properties or potentially yaml based format) is kind of inconsistent and we don't want to promote that feature.

(쉬운 것이 반드시 올바른 것은 아닙니다. Spring Boot의 맥락에서 @PropertySource (속성 또는 yaml 기반 형식 일 수도 있음)의 전체 사용은 일관성이 없으며 해당 기능을 승격하고 싶지 않습니다.)
```

*출처 : https://jira.spring.io/browse/SPR-13912*

스프링 부트 공식 문서에서도 그 내용을 찾을 수 있습니다.

```
While using @PropertySource on your @SpringBootApplication seems convenient and easy enough to load a custom resource in the Environment, we do not recommend it as Spring Boot prepares the Environment before the ApplicationContext is refreshed. Any key defined via @PropertySource will be loaded too late to have any effect on auto-configuration.

(@SpringBootApplication에서 @PropertySource를 사용하는 것이 환경에서 사용자 정의 리소스를로드하기에 편리하고 쉽지만 ApplicationContext가 새로 고쳐지기 전에 Spring 부트가 환경을 준비 할 때 권장하지 않습니다. @PropertySource를 통해 정의 된 키는 자동 구성에 영향을 미치기에는 너무 늦게로드됩니다.)
```

*출처 : http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-customize-the-environment-or-application-context*

#### Spring Boot의 Auto Configuration을 사용함에`@PropertySource` 는 너무 늦다고 말합니다.

제가 아는 Auto Configuration으로만 이해하려고 했어서 위의 내용을 이해데 한참 늦었습니다 (아직도..?). 저는 Auto Configuration 에서 `@Conditional*` 기반의 `ApplicationContext` 가 생성된 후의 일만을 생각했기 때문입니다.

```
Classes passed to the SpringApplication static convenience methods, and those added using setSources() are inspected to see if they have @PropertySources, and if they do, those properties are added to the Environment early enough to be used in all phases of the ApplicationContext lifecycle.

(
SpringApplication 정적 편리 메소드에 전달 된 클래스와 setSources ()를 사용하여 추가 된 클래스는 @PropertySources가 있는지 검사하여 ApplicationContext 라이프 사이클의 모든 단계에서 사용할 수있을만큼 일찍 환경에 추가됩니다 .)
```

*출처 : https://docs.spring.io/spring-boot/docs/current/reference/html/howto-properties-and-configuration.html#howto-change-the-location-of-external-properties*

실제로 `@PropertySource`는 느리지 않습니다. `ApplicationContext` 의 lifecycle 에서 매우 빠르다고 문서에서 말하고 있습니다.

**그렇다면 무엇이 느린 것인가!**

핵심 문장은 이것 입니다!

`Spring Boot prepares the Environment before the ApplicationContext is refreshed`

ApplicationContext 생성 전 Spring Boot가 준비하는 과정이라고 저는 해석을 했습니다.

아래가 스프링 부트 어플리케이션이 실행될 때 이벤트 순서입니다. 그리고 ApplicationContext가 생성되기 전에 실행되는 이벤트가 보입니다.

```
1. An ApplicationStartingEvent is sent at the start of a run, but before any processing except the registration of listeners and initializers.

2. An ApplicationEnvironmentPreparedEvent is sent when the Environment to be used in the context is known, but before the context is created.

3. An ApplicationPreparedEvent is sent just before the refresh is started, but after bean definitions have been loaded.

4. An ApplicationReadyEvent is sent after the refresh and any related callbacks have been processed to indicate the application is ready to service requests.

5. An ApplicationFailedEvent is sent if there is an exception on startup.
```

```
Some events are actually triggered before the ApplicationContext is created so you cannot register a listener on those as a @Bean.

(어떤 이벤트는 실제로 ApplicationContext가 생성되기 전에 트리거되어서 @Bean으로 리스너를 등록 할 수 없습니다.)
```

*출처 : http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-application-events-and-listeners*

#### Spring Boot 는 어디선가 ApplicationContext 가 생성되기도 전에 무엇인가를 하고 있구나 확신이 듭니다!

그럼 그 무엇인가는 무엇일까요?

Spring Boot 의 Auto Configuration의 핵심인 `EnableAutoConfigurationImportSelector`는 `SpringFactoriesLoader` 를 사용하고 있습니다.

`SpringFactoriesLoader`는 Class Path의 META-INF 경로 하위의 모든 `spring.factories` 로드하여 Spring Framework의 내부 사용을 위한 설정을 적용하는 클래스입니다.

spring.factories 의 대표적인 기능이 바로 Spring Framework 의 listener 를 등록하는 것 입니다.

```
If you want those listeners to be registered automatically regardless of the way the application is created you can add a META-INF/spring.factories file to your project and reference your listener(s) using the org.springframework.context.ApplicationListener key.

(응용 프로그램의 작성 방법에 관계없이 리스너를 자동으로 등록하려면 META-INF / spring.factories 파일을 프로젝트에 추가하고 org.springframework.context.ApplicationListener 키를 사용하여 리스너를 참조 할 수 있습니다.)
```

*출처 : http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-application-events-and-listeners*

그리고 아래는 Spring Boot Auto Configuration 프로젝트인 `spring-boot-autoconfigure` 의 `spring.factories` 내용의 일부입니다.

```
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnClassCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
```

일단 찾고 있던 Spring Framework 에게 ApplicationContextInitializer와 ApplicationListener 을 등록하는 것이 보입니다.

그리고 그 외에도 여러가지 Key 값이 보이며, 지난 스프링 캠프의 저의 발표에서도 언급했던 `EnableAutoConfiguration` 도 보입니다. Spring Boot는 `factories`를 통해 `EnableAutoConfiguration` 클래스 리스트를 주입받고, 어디에선가 사용을 하고 있습니다.

#### 현재까지 내용을 정리해보자면,

Spring Boot 는 `ApplicationContext` 가 생성되기 전(`@PropertySource`로 설정한 파일이 로드되기도 전)에도 Environment 를 사용한 `무엇인가`를 통해 Auto Configuration 에 영향을 미치고 있겠다고 생각이 됩니다.

그래서 Spring Boot는 @PropertySource를 권장하지 않겠구나라는 것을 대충 짐작으로 생각하게 되었습니다.

### 4. 그래서 무엇을 사용하라는 것인가?

그들이 제안하는 방식은 모두 동일합니다. 스프링이 로드되는 과정의 아주 앞단에서 properties 를 정의하는 방식입니다.

그들이 소개했던 방식을 옮겨 적습니다.

#### 1. spring.config 옵션

Spring Boot 에서는 spring.config 라는 옵션을 제공합니다.

`ConfigFileApplicationListener` class 를 통해 제공되는 옵션으로, `spring.config.name`은 파일 이름, `spring.config.location`은 경로를 입력받습니다.

> 해당 class 를 열어보시면 알겠지만, 저 옵션을 통해 `spring.config` 옵션과는 무관하게 `classpath:/,classpath:/config/,file:./,file:./config/` 와 그 경로에서 name이 `application`인 properties(or yaml) 파일은 항상 로드됩니다.

해당 옵션을 정의하는 방법을 두 가지 소개합니다.

##### Spring Boot 환경변수

jar나 war 를 구동할 때 삽입하는 옵션입니다.

```
java -jar kingbbode.jar --spring.config.name=kingbbode
```

##### SpringApplicationBuilder

`SpringApplicationBuilder` class는 우리가 스프링 프레임워크 어플리케이션의 main 에 항상 작성했던 `SpringApplication` class 의 Builder class 입니다. 빌더를 사용해 properties 를 주입합니다.

```java
public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class)
				.properties(
						"spring.config.location=" +
						"classpath:/kingbbode.yml" +
						", file:/data/kingbbode/" +
						", file:/data/test/kingbbode.yml"
				)
				.run(args);
	}
```

> location 옵션은 경로를 지정할 수 있지만, 단일 파일도 지정할 수 있습니다. ignoreResourceNotFound 옵션을 따로 지정할 필요없이 자동으로 적용이 됩니다.

#### 2. ApplicationEnvironmentPreparedEvents

Environment 구성 전에 이벤트를 만들어서 삽입하는 방식입니다.

```java
new SpringApplicationBuilder(SanityCheckApplication.class)
    .listeners(new LoadAdditionalProperties())
    .run(args);
```

```java
@Component
public class LoadAdditionalProperties implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {

    private ResourceLoader loader = new DefaultResourceLoader();

    @Override
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        try {
            Resource resource = loader.getResource("classpath:kingbbode.properties");
            PropertySource<?> propertySource = new PropertySourcesLoader().load(resource);
            event.getEnvironment().getPropertySources().addLast(propertySource);
        } catch (IOException ex) {
            throw new IllegalStateException(ex);
        }
    }

}
```

*출처 : https://github.com/spring-projects/spring-boot/issues/6220*

#### 3. EnvironmentPostProcessor

Listener 보다는 Spring 을 사용함에 꽤 친숙한 Class 입니다. 수많은 `PostProcessor` 중 `Environment` 의 `PostProcessor` 를 정의하는 것 입니다.

이 방식은 부트 공식 문서에서도 추천하는 방식입니다. 를 정의하고 `spring.factories` 를 통해 등록하는 방식입니다. 자세한 내용은 아래 출처로 작성한 공식 문서를 보시면 됩니다.

```java
public class EnvironmentPostProcessorExample implements EnvironmentPostProcessor {

	private final YamlPropertySourceLoader loader = new YamlPropertySourceLoader();

	@Override
	public void postProcessEnvironment(ConfigurableEnvironment environment,
			SpringApplication application) {
		Resource path = new ClassPathResource("com/example/myapp/config.yml");
		PropertySource<?> propertySource = loadYaml(path);
		environment.getPropertySources().addLast(propertySource);
	}

	private PropertySource<?> loadYaml(Resource path) {
		if (!path.exists()) {
			throw new IllegalArgumentException("Resource " + path + " does not exist");
		}
		try {
			return this.loader.load("custom-resource", path, null);
		}
		catch (IOException ex) {
			throw new IllegalStateException(
					"Failed to load yaml configuration from " + path, ex);
		}
	}
}
```

```
# META-INF/spring.factories
org.springframework.boot.env.EnvironmentPostProcessor=com.example.YourEnvironmentPostProcessor
```

*출처 : http://docs.spring.io/spring-boot/docs/1.5.x-SNAPSHOT/reference/htmlsingle/#howto-customize-the-environment-or-application-context*

---

### 마무리

![before](/images/2017/2017-08-21-SPRING-BOOT-PROPERTIES/before.jpg)

처음에는 `왜 이래, 뭐야` 했지만.

![after](/images/2017/2017-08-21-SPRING-BOOT-PROPERTIES/after.jpg)

이제는 대단하고 멋져보입니다. 피보탈 짱짱.

호기심을 쫓아 내용을 정리하였습니다. 조금은 불편하지만, Spring Boot 가 우리를 정말 편하게 해주고 있으니, 믿고 일단 따라가야겠습니다.

##### 정리를 하면서 알아두면 반드시 좋은 정보도 공유하겠습니다!

Spring Boot는 합리적인 값 `Override`를 허용하도록 설계된 **매우 특별한 PropertySource 순서** 를 사용한다는 내용입니다. Override 순서에 맞게 잘 작성하여, 삽질하는 일을 방지! 밑의 번호는 우선순위입니다. 밑으로 갈 수록 우선순위가 낮다는 말 입니다.

```
1. Devtools global settings properties on your home directory (~/.spring-boot-devtools.properties when devtools is active).
2. @TestPropertySource annotations on your tests.
3. @SpringBootTest#properties annotation attribute on your tests.
4. Command line arguments.
5. Properties from SPRING_APPLICATION_JSON (inline JSON embedded in an environment variable or system property)
6. ServletConfig init parameters.
7. ServletContext init parameters.
8. JNDI attributes from java:comp/env.
9. Java System properties (System.getProperties()).
10. OS environment variables.
11. A RandomValuePropertySource that only has properties in random.*.
12. Profile-specific application properties outside of your packaged jar (application-{profile}.properties and YAML variants)
13. Profile-specific application properties packaged inside your jar (application-{profile}.properties and YAML variants)
14. Application properties outside of your packaged jar (application.properties and YAML variants).
15. Application properties packaged inside your jar (application.properties and YAML variants).
16. @PropertySource annotations on your @Configuration classes.
17. Default properties (specified using SpringApplication.setDefaultProperties).
```

*출처 : https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html*

##### 그리고 한편으로는 그들에게 조금은 아쉬운 면도 있습니다.

`@PropertySource` 는 Spring Boot가 아닌 Spring Framework 가 제공하는 class 입니다.

```
Easy doesn't necessarily means correct. In the context of Spring Boot, the whole use of @PropertySource (be it for properties or potentially yaml based format) is kind of inconsistent and we don't want to promote that feature.

(쉬운 것이 반드시 올바른 것은 아닙니다. Spring Boot의 맥락에서 @PropertySource (속성 또는 yaml 기반 형식 일 수도 있음)의 전체 사용은 일관성이 없으며 해당 기능을 승격하고 싶지 않습니다.)
```

Spring Boot 를 위해 `properties` 지원하지만 더 이상 개발은 안하겠다는 것인데, Spring Framework 이면 무조건 Spring Boot 를 쓰라는 것 일까요? 이 부분에서는 아직 잘 모르겠습니다.

그리고 정리를 하면서 느낀 것은, 내가 설정할 properties 가 Spring Boot 의 자동 설정과 전혀 무관하며, 관계 없다고 여겨지는 것은 그냥 사용해도 괜찬다고도 생각되어 집니다. 이것은 개인적인 생각입니다!

그래서 `@PropertySource` Yaml 을 사용하는 코드는 공유를 하겠습니다.

---

**@PropertySource Yaml 사용하기**

```java
public class YamlPropertiesProcessor extends YamlProcessor {

    public YamlPropertiesProcessor(Resource resource) throws IOException {
        if(!resource.exists()){
            throw new FileNotFoundException();
        }
        this.setResources(resource);
    }

    public Properties createProperties() throws IOException {
        final Properties result = CollectionFactory.createStringAdaptingProperties();
        process((properties, map) -> result.putAll(properties));
        return result;
    }
}

public class YamlPropertySourceFactory implements PropertySourceFactory {
    private static final String YML_FILE_EXTENSION = ".yml";

    @Override
    public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {
        String filename = resource.getResource().getFilename();
        if (filename != null && filename.endsWith(YML_FILE_EXTENSION)) {
            return name != null ?
                    new YamlResourcePropertySource(name, resource) :
                    new YamlResourcePropertySource(getNameForResource(resource.getResource()), resource);
        }
        return (name != null ? new ResourcePropertySource(name, resource) : new ResourcePropertySource(resource));
    }

    private String getNameForResource(Resource resource) {
        String name = resource.getDescription();
        if (!StringUtils.hasText(name)) {
            name = resource.getClass().getSimpleName() + "@" + System.identityHashCode(resource);
        }
        return name;
    }
}

public class YamlResourcePropertySource extends PropertiesPropertySource {

    public YamlResourcePropertySource(String name, EncodedResource resource) throws IOException{
        super(name, new YamlPropertiesProcessor(resource.getResource()).createProperties());
    }
}
```

사용법

```java
@Configuration
@PropertySource(value = {"classpath:/test.yml","classpath:/test2.yml", "file:/data/test3.yml"}, ignoreResourceNotFound = true, factory = YamlPropertySourceFactory.class)
public class AppConfiguration {

...

}
```

---

**ConfigurationProeprties 의 locations 기능 구현하기**

링크로 공유드립니다!!

[fabiomaffioletti : http://fabiomaffioletti.me/blog/2016/12/20/spring-configuration-properties-handle-deprecated-locations/](http://fabiomaffioletti.me/blog/2016/12/20/spring-configuration-properties-handle-deprecated-locations/)

---

여기까지 파볼 수 있는 계기 마련해주신 [창천향로(갓천향로)](http://jojoldu.tistory.com/) 님 감사합니다!
