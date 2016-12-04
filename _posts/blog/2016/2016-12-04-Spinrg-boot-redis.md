---
layout: post
title: Spring Boot에서 Redis 사용하기
categories: [blog]
tags: [spring boot,redis,nosql]
fullview: false
comments: true
published: true
outlink: 0
---

Redis를 쓰고 있으면서도 잘 몰랐습니다. 그래서 정리했습니다!

Spring Boot에서 Redis 사용하기
==============================

Redis란?
--------

![Redis](/images/2016/2016_12_04_Spring_Boot_Redis/redis.png)

-	`Remote Dictionary Server`의 약자 
- 오픈 소스 소프트웨어 
- `휘발성`이면서 `영속성`을 가진 `key-value` 저장소

### Redis는 NoSQL

**NoSQL**은 데이터 간의 관계를 정의하지 않고 고정된 스키마를 갖지 않는 새로운 형태의 데이터베이스로서, **관계형 데이터베이스(RDBMS)를 경량화한 데이터베이스** 입니다. 관계형 데이터베이스의 특징 을 제거하고 만들어진 다른 모든 형태의 DBMS를 칭하 기도 하며, SQL 계열 질의어를 사용할 수 있다는 사실 을 강조한다는 면에서 “Not Only SQL"로 불리기도 합니다.

*Redis는 이러한 NoSQL의 종류 중 하나입니다.*

### 데이터 모델

NoSQL이 가지고 있는 대표적인 데이터 모델은 아래와 같습니다.

-	Key-Value
	-	하나의 Key에 하나의 Value를 갖는 데이터 모델
	-	Key로만 접근 가능
-	Column
	-	하나의 Key에 여러개의 Value를 갖을 수 있는 데이터 모델
	-	중첩된 HashMap 구조
-	Document
	-	Value가 Json이나 XML Document를 갖는 데이터 모델
	-	Value의 일부로 질의하고 일부만 가져올 수 있음
-	Graph
	-	관계에 특화된 모델
	-	노드와 간선에 대한 정보

*이 중 Redis는 Key-Value 데이터 모델을 사용합니다. <br>단순한 형태의 Key-Value 방식으로 빠른 속도를 지원할 수 있습니다.*

### 데이터 타입

Redis는 key-value 데이터 모델에서 Value가 가질 수 있는 자료 구조를 제공합니다.

제공되는 자료구조를 아래와 같습니다.

-	String
	-	Command Docs : http://redis.io/commands#string
	-	Text, 숫자 등 일반적인 Value
-	Set
	-	Command Docs : http://redis.io/commands#set
	-	Value가 Set 자료구조가 되므로, 하나의 Key 값에 여러 값을 Set 자료구조를 통해 저장 가능
	-	Set간의 집합 연산을 제공
-	Sorted Set
	-	Command Docs : http://www.redis.io/commands#sorted_set
	-	score라는 필드가 추가되어 정렬의 기준이 됨.
	-	score를 통해 질의 가능
	-	그 외에는 위의 Set과 같음
-	Hashes

	-	Command Docs : https://redis.io/commands#hash
	-	Value가 Map 자료구조와 같은 Key/Value 형태가 됨
		-	하나의 Key에 여러개의 필드를 갖는 구조가 됨

-	List

	-	Command Docs : https://redis.io/commands#list
	-	Value가 linkedList 자료구조가 됨
		-	Push/Pop 연산 가능
		-	index 활용 질의 가능

### 휘발성

스티븐 옌은 자신의 블로그의 글 "NoSQL is a Horseless Carriage"에서 Redis를 Store가 아닌 Cache로 구분했습니다.[(A Yes for a NoSQL Taxonomy 참고)](http://highscalability.com/blog/2009/11/5/a-yes-for-a-nosql-taxonomy.html)

*Reids는 디스크 기반이 아닌 메모리에 데이터를 read/write하는 in-memory 솔루션이기 때문입니다.*

메모리 기반이기 때문에 갖을 수 있는 장점은 아래와 같습니다.

-	메모리에 데이터를 read/write 하기 때문에 매우 빠른 속도를 보장할 수 있습니다.
-	모든 데이타가 메모리 안에 있기 때문에 캐시 관점에서 매우 유용합니다.
-	Cache 방식을 통한 DB 부하 감소

### 영속성

레디스는 지속성을 보장하기 위해 데이터를 DISK에 저장할 수 있습니다. 서버가 내려가더라도 DISK에 저장된 데이터를 읽어서 메모리에 로딩을 합니다.

데이터를 DISK에 저장하는 방식은 크게 두 가지 방식이 있습니다.

-	snapshotting (RDB) 방식
	-	순간적으로 메모리에 있는 내용을 DISK에 전체를 옮겨 담는 방식
-	AOF(Append On File) 방식
	-	redis의 모든 write/update 연산 자체를 모두 log 파일에 기록하는 형태

---

Spring Boot 시작
----------------

### Dependency

> build.gradle

```
compile("org.springframework.data:spring-data-redis:1.7.2.RELEASE")
compile("redis.clients:jedis:2.8.1")
```

-	spring-data-redis : 스프링에서 공식 지원하는 Dependency로 Redis Client와 연동 가능한 높은 레벨의 `RedisTemplate` 추상화를 제공합니다.

-	jedis : 가장 많이 사용되고 있는 Java Client로 가장 활발한 오픈소스이며, 레디스에서 공식 추천하고 있는 Client 중 하나입니다. [[참고]](http://redis.io/clients#java)

### Java Configuration

> RedisConfig.java

```java
@Configuration
public class RedisConfig {
    @Bean
    public JedisConnectionFactory jedisConnectionFactory() {

        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory();
        jedisConnectionFactory.setHostName("xxx.xxx.xxx.xxx");
        jedisConnectionFactory.setPort(6379);
        jedisConnectionFactory.setTimeout(0);
        jedisConnectionFactory.setUsePool(true);

        return jedisConnectionFactory;
    }

    @Bean
    public StringRedisTemplate redisTemplate() {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setConnectionFactory(jedisConnectionFactory());
        return stringRedisTemplate;
    }
}
```

Spring Data에서 제공하는 만큼 기존 JDBC 설정 지원과 거의 유사함을 볼 수 있습니다.

#### Operations

Operation은 사용할 데이터 타입에 따라 다르게 사용해야 합니다. Operation들은 `@Resources` 어노테이션을 통해 선언해놓은 `RedisTemplate`로부터 주입 받을 수 있습니다.

**String**

```java
@Resource(name="redisTemplate")
private ValueOperations<String, String> valueOperations;
```

**Set**

```java
@Resource(name="redisTemplate")
private SetOperations<String, String> setOperations;
```

**Sorted Set**

```java
@Resource(name="redisTemplate")
private ZSetOperations<String, String> zSetOperations;
```

**Hashes**

```java
@Resource(name="redisTemplate")
private HashOperations<String, String, String> hashOperations;
```

**Hashes**

```java
@Resource(name="redisTemplate")
private ListOperations<String, String> listOperations;
```

#### Operation 사용 예제

적절하지는 않지만, 다양한 데이터 타입을 사용하는 간단한 예제를 만들어보았습니다.

Redis에 간단한 자기소개 정보를 저장해두고, 특정 사건이 발생함에 따라 변하는 변화를 Redis Operation으로 풀어보았습니다~

특정 사건은 선임의 이직!

![bye](/images/2016/2016_12_04_Spring_Boot_Redis/bye.gif)

> SpringBootRedisApplicationTests.java

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringBootRedisApplication.class)
public class SpringBootRedisApplicationTests {

    @Resource(name="redisTemplate")
    private ValueOperations<String, String> valueOperations;

    @Resource(name = "redisTemplate")
    private ListOperations<String, String> listOperations;

    @Resource(name = "redisTemplate")
    private HashOperations<String, String, String> hashOperations;

    @Resource(name = "redisTemplate")
    private SetOperations<String, String> setOperations;

    @Resource(name="redisTemplate")
    private ZSetOperations<String, String> zSetOperations;
```

테스트가 아니라 간단히 예제를 보여줄 예정이여서 `Application`의 `Config`를 그대로 사용하려고 합니다.

#### Data 삽입

```java
@Before
public void init() {
    //list put
    listOperations.rightPush("test:task", "자기소개");
    listOperations.rightPush("test:task", "취미소개");
    listOperations.rightPush("test:task", "소망소개");
    listOperations.rightPush("test:task", "선임이직");
    //hash put
    hashOperations.put("test:user:kingbbode", "name", "권뽀대");
    hashOperations.put("test:user:kingbbode", "age", "28");
    //set put
    setOperations.add("test:user:kingbbode:hobby", "개발");
    setOperations.add("test:user:kingbbode:hobby", "잠");
    setOperations.add("test:user:kingbbode:hobby", "옷 구경");
    //zset
    zSetOperations.add("test:user:kingbbode:wish", "배포한 것에 장애없길", 1);
    zSetOperations.add("test:user:kingbbode:wish", "배포한거 아니여도 장애없길", 2);
    zSetOperations.add("test:user:kingbbode:wish", "경력직 채용", 3);
    zSetOperations.add("test:user:kingbbode:wish", "잘자기", 4);
}
```

`List`는 `Push`를 통하여 데이터를 저장합니다. push에는 leftPush와 rightPush가 있습니다. Stack이나 Queue로 활용 가능합니다.

`Hash`는 Map 자료형과 사용이 거의 비슷합니다. `key-field-value`를 작성하여 데이터를 삽입합니다.

`Set`은 `같은 Key에 여러 Value`를 작성할 수 있습니다.

`ZSet`은 `Value에 Score를 포함`하여 데이터를 삽입하였습니다.

*모두 네이밍을 통해서 알만한 메서드를 operation에서 제공해주고 있습니다.*

#### 데이터 조회

```java
@Test
public void redisTest1() {
    String task = listOperations.leftPop("test:task");
    StringBuilder stringBuilder = new StringBuilder();
    while (task != null) {
        switch (task) {
            case "자기소개":
                Map<String, String> intro = hashOperations.entries("test:user:kingbbode");
                stringBuilder.append("\n******자기소개********");
                stringBuilder.append("\n이름은 ");
                stringBuilder.append(intro.get("name"));
                stringBuilder.append("\n나이는 ");
                stringBuilder.append(intro.get("age"));
                break;
            case "취미소개":
                Set<String> hobbys = setOperations.members("test:user:kingbbode:hobby");
                stringBuilder.append("\n******취미소개******");
                stringBuilder.append("취미는");
                for (String hobby : hobbys) {
                    stringBuilder.append("\n");
                    stringBuilder.append(hobby);
                }
                break;
            case "소망소개":
                Set<String> wishes = zSetOperations.range("test:user:kingbbode:wish", 0, 2);
                stringBuilder.append("\n******소망소개******");
                int rank = 1;
                for (String wish : wishes){
                    stringBuilder.append("\n");
                    stringBuilder.append(rank);
                    stringBuilder.append("등 ");
                    stringBuilder.append(wish);
                    rank++;
                }
                break;
            case "선임이직":
                stringBuilder.append("\n!!! 믿었던 선임 이직");
                zSetOperations.incrementScore("test:user:kingbbode:wish", "경력직 채용", -1);
                listOperations.rightPush("test:task", "소망소개");
                break;
            default:
                stringBuilder.append("nonone");

        }
        task = listOperations.leftPop("test:task");
    }
    System.out.println(stringBuilder.toString());
}
```

`List`는 `Pop`를 통하여 좌측 노드부터 데이터를 꺼냈습니다. 데이터를 rightPush 했으므로 leftPop을 사용함으로써 Queue처럼 사용하고 있습니다.

`Hash`는 `entries`를 통해 데이터를 Map 자료형으로 가져올 수 있습니다.

`Set`의 Value 안의 값들을 member라고 합니다. 그래서 `members`를 통해 해당 Key의 값들을 Set 자료형으로 가져올 수 있습니다.

`ZSet`은 range를 통해 정해진 Rank 범위의 데이터를 가져올 수 있습니다. score를 기준으로 오름차순으로 데이터를 정렬합니다.

`선임이직`이라는 이벤트가 발생시, 경력직 채용의 score를 -1하여 rank를 높이도록 해두었습니다. `zset`의 `incrementScore`를 사용했습니다. 그리고 소망이 변경되었으니 소망을 한번 더 말해도록 task를 추가했습니다.

출력결과는 원하는 작업을 잘 수행해서 나옵니다!

![결과](/images/2016/2016_12_04_Spring_Boot_Redis/result.jpeg)

[GitHub Source](https://github.com/kingbbode/spring-boot-redis)

---

[Redis command 문서](https://redis.io/commands)에 redis가 어떤 command를 가지고 있는지 모두 나오며, Operation에서 제공해주는 메서드명이 모두 짐작 가능한 메서드이므로 문서를 잘 참고한다면 누구라도 사용하는데 무리가 없을 것 같습니다.

인기가 많기도 하며, 개인 프로젝트에서 사용하긴 했는데, Redis가 무엇이며, 장점이 무엇이며, 언제 써야하는 것인지를 잘 모르고 많이 썼던 것 같습니다. 이번 정리를 통해 Redis를 좀 더 알 것 같습니다! 아직 멀었지만..
