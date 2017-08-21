Spring Boot 와 Properties(or Yaml)
==================================

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

**@PropertySource Yaml 사용하기**

(결론이야 어찌되었든 만들었으니 공개..)

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

생각보다 너무 간단한데..?

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

Easy doesn't necessarily means correct. In the context of Spring Boot, the whole use of @PropertySource (be it for properties or potentially yaml based format) is kind of inconsistent and we don't want to promote that feature.
```

*출처 : https://jira.spring.io/browse/SPR-13912*

스프링 부트 공식 문서에서도 그 내용을 찾을 수 있습니다.

```
While using @PropertySource on your @SpringBootApplication seems convenient and easy enough to load a custom resource in the Environment, we do not recommend it as Spring Boot prepares the Environment before the ApplicationContext is refreshed. Any key defined via @PropertySource will be loaded too late to have any effect on auto-configuration.
```

*출처 : http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-customize-the-environment-or-application-context*
