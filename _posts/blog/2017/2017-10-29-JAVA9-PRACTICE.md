---
layout:     post
title:      Java 9 Collections, Stream Improvements
author:     kingbbode
tags:       java
subtitle:   핵심 기능 외! 바로 쓸 수 있을만한, 코드 짜는 것을 더 편하게 만들어줄 수 있는 Java 9 의 새로운 기능들을 소개
category:  posts
outlink: 0
---

![late](/images/2017/2017-10-29-JAVA9-PRACTICE/java.png)

2017년 9월 21일 Java 9 이 출시되었습니다. 크게 부각되고 있는 기능은 `Jigsaw`, `Reactive Streams`, `REPL/JShell` 가 있습니다.

출시 전부터 기대를 많이 받던 기능들이지만, 학습비용이 어마어마할 것 같습니다 ..!

그래서 저는 일단 제가 바로 쓸 수 있을만한, 코드 짜는 것을 더 편하게 만들어줄 수 있는 Java 9 의 새로운 기능들을 소개해보려고 합니다. 성능적인 부분과 좀 더 언어에 대한 깊은 이해가 있어야 하겠지만, 지극히 부족한 제 관점에서 편리한 기능을 소개해봅니다!

소개할 내용은

-	Collections Improvements
-	Stream Improvements

입니다.

예제는 이미 추석 전에 모두 작성했었는데...

![late](/images/2017/2017-10-29-JAVA9-PRACTICE/late.png)

어마어마한 게으름 추석이 지나고서야 글을 작성하게 되었습니다. 배그가 너무 재밌네요.ㅜㅜ

![bg](/images/2017/2017-10-29-JAVA9-PRACTICE/bg.jpeg)

**아래 사용한 예제들은 지극히 사용법을 소개하기 위한 예제이며, 응용에 대한 것은 고려하지 않았습니다.**

Collections Improvements
------------------------

첫 번째로 살펴 볼 것은 정말 많이 사용하고 있는 `Collection Framework` 에 제공되는 immutable collection을 만드는 팩토리 메서드입니다.

Java의 버전이 올라가면서 정말 많은 기능이 추가되었지만, 2%의 아쉬움을 채워줄 google의 `guava` library를 사용할 수 밖에 없었습니다.

하지만! 이번 업데이트는 `guava`를 이제 그만 보내줄 수 있을 것 같습니다!

### List.of

**Java 8**

```java
Collections.unmodifiableList(
  Arrays.asList(1,2,3,4,5,6,7,8,9,10,11)
);

//or

Stream.of(1,2,3,4,5,6,7,8,9,10,11)
  .collect(collectingAndThen(toList(),Collections::unmodifiableList));
```

```java
IntStream.range(1, 100)
  .boxed()
  .collect(collectingAndThen(toList(),Collections::unmodifiableList))
);
```

**Java 9**

```java
List.of(1,2,3,4,5,6,7,8,9,10,11);
```

```java
List.of(
  IntStream.range(1, 100)
          .boxed()
          .toArray(Integer[]::new)
);
```

### Set.of

**Java 8**

```java
Collections.unmodifiableSet(
        new HashSet<>(
                Arrays.asList(1,2,3,4,5,6,7,8,9,10,11)
        )
);

//or

Stream.of(1,2,3,4,5,6,7,8,9,10,11)
  .collect(collectingAndThen(toSet(),Collections::unmodifiableSet));
```

```java

IntStream.range(1, 100)
  .boxed()
  .collect(collectingAndThen(toSet(),Collections::unmodifiableSet))
);
```

**Java 9**

```java
Set.of(1,2,3,4,5,6,7,8,9,10,11);
```

```java
set = Set.of(
        IntStream.range(1, 100)
                .boxed()
                .toArray(Integer[]::new)
);
```

### Map

**Java 8**

```java
Map<String, String> map = new HashMap<>();
map.put("key1", "value1");
map.put("key2", "value2");
map.put("key3", "value3");
map.put("key4", "value4");
map.put("key5", "value5");
map.put("key6", "value6");
map.put("key7", "value7");
map.put("key8", "value8");
map.put("key9", "value9");
map.put("key10", "value10");
map.put("key11", "value11");
map.put("key12", "value12");
map.put("key13", "value13");
map.put("key14", "value14");
Collections.unmodifiableMap(map);
```

```java
Stream.of(
  new Pair<>("key1", "value1"),
  new Pair<>("key2", "value2"),
  new Pair<>("key3", "value3"),
  new Pair<>("key4", "value4"),
  new Pair<>("key5", "value5"),
  new Pair<>("key6", "value6"),
  new Pair<>("key7", "value7"),
  new Pair<>("key8", "value8"),
  new Pair<>("key9", "value9"),
  new Pair<>("key10", "value10"),
  new Pair<>("key11", "value11"),
  new Pair<>("key12", "value12"),
  new Pair<>("key13", "value13"),
  new Pair<>("key14", "value14")
).collect(collectingAndThen(toMap(Pair::getKey, Pair::getValue),Collections::unmodifiableMap));
```

**Java 9**

```java
Map.of(
  "key1","value1",
  "key2","value2",
  "key3","value3",
  "key4","value4",
  "key5","value5",
  "key6","value6",
  "key7","value7",
  "key8","value8",
  "key9","value9",
  "key10","value10"
);
```

```java
Map.ofEntries(
  Map.entry("key1", "value1"),
  Map.entry("key2", "value2"),
  Map.entry("key3", "value3"),
  Map.entry("key4", "value4"),
  Map.entry("key5", "value5"),
  Map.entry("key6", "value6"),
  Map.entry("key7", "value7"),
  Map.entry("key8", "value8"),
  Map.entry("key9", "value9"),
  Map.entry("key10", "value10"),
  Map.entry("key11", "value11"),
  Map.entry("key12", "value12"),
  Map.entry("key13", "value13"),
  Map.entry("key14", "value14")
);
```

```java
Map.ofEntries(
  IntStream.range(1, 100)
    .mapToObj(index ->
            Map.entry("key" + index, "value" + index)
    )
    .toArray((IntFunction<Map.Entry<String, String>[]>) Map.Entry[]::new)
);
```

`Collections Improvements` 에는 편리한 Factory Method 가 정말 많이 생겼습니다. 제가 생각하는 몇 가지 포인트를 집어보았습니다.

### 1. 가독성

위의 예제들을 보면 알겠지만, 코드 라인 수가 그렇게 많이 절약되는 것이 아닌 경우도 분명히 있습니다.

위 예제 중 한가지를 예로 들자면,

**Java 8**

```java

 IntStream.range(1, 100)
   .boxed()
   .collect(collectingAndThen(toSet(),Collections::unmodifiableSet))
 );
```

**Java 9**

```java
 Set.of(
         IntStream.range(1, 100)
                 .boxed()
                 .toArray(Integer[]::new)
 );
```

위와 같은 상황입니다.

**Java 8** 로 작성된 코드를 보았을 때 저 코드가 Set을 반환할 것이라는 것이 한번에 보이지는 않습니다. 반면 **Java 9** 의 코드는 명시적으로 무슨 행위를 하는지가 먼저 나올 수 있는 형태입니다.

주체에 대한 역전도 보입니다. **Java 8** 에서 `IntStream`이 `Set`을 만든다라면 **Java 9** 은 `Set`이 `IntStream`을 인자로 사용한다로 읽혀질 수 있습니다.

저는 이것이 코드의 흐름을 읽을 수 있는 명시적 작성 효과로 가독이 더 좋아질 수 있다고 생각합니다.

### 2. 중복에 대한 원천 차단.

```java
@Test(expected = IllegalArgumentException.class)
public void 중복을_허용하지_않는다() {
    set = Set.of(1, 2, 3, 1);
}
```

```java
@Test(expected = IllegalArgumentException.class)
public void 키_중복을_허용하지_않는다() throws Exception {
    Map.of(
            "key1", "value1",
            "key2", "value1",
            "key3", "value1",
            "key1", "value1"
    );
}
```

중복 값을 넣는 경우 예외를 발생시켜, 버그를 원천 차단합니다.

---

Stream Improvements
-------------------

`Stream` 을 사용하면서 항상 `Stream` 에서 반복에 대한 제어를 못함에 어려운 부분이 많았는데, 이번 새로운 기능으로 어느 정도는 해소가 가능할 것으로 보입니다.

### 1. takeWhile

while문과 같은 사용성으로 조건이 false가 나올 **때까지** Stream 을 생성합니다.

```java
@Test
public void 조건까지의_데이터를_스트림으로_만드는_takeWhile() throws Exception {
   list = Stream.of(
           IntStream.range(1, 5)
                   .boxed()
                   .toArray(Integer[]::new)
   ).takeWhile(integer -> integer < 3)
           .collect(Collectors.toList());
   assertThat(list.size(), is(2));

   //list : [1,2]
}
```

기존에는 한번 시작된 `Stream` 을 멈출 수 없었지만, `takeWhile` 을 사용한다면 특정 조건까지만 돌고 멈추도록 할 수 있게 되었습니다.

```java
@Test
public void takeWhile은_전체를_조건으로_filter하는_것이_아님() throws Exception {
   list = Stream.of(
           1,2,3,4,5,1,2,3,4,5
   ).takeWhile(integer -> integer < 5)
           .collect(Collectors.toList());
   assertThat(list.size(), is(4));
}
```

`takeWhile` 은 해당 조건을 만족하지 않는 경우가 처음 만날 때까지, 즉 처음으로 `false` 가 나올 때 까지만 이므로, `Stream` 의 `filter` 와는 전혀 다른 사용성을 갖습니다.

### 2. dropWhile

dropWhile 은 takeWhile의 정반대로 조건이 false가 나올 **때부터** Stream 을 생성합니다.

```java
@Test
public void 조건까지의_데이터를_버리고_스트림으로_만드는_dropWhile() throws Exception {
    list = Stream.of(
            IntStream.range(1, 5)
                    .boxed()
                    .toArray(Integer[]::new)
    ).dropWhile(integer -> integer < 3)
            .collect(Collectors.toList());
    assertThat(list.size(), is(2));
}

//list : [3,4]
```

정확히 이것이 언제 필요할지는 잘 모르겠지만, takeWhile의 반대 케이스로 분명히 유용히 쓰일 수 있을 것 입니다.

```java
@Test
public void dropWhile은_전체를_조건으로_filter하는_것이_아님() throws Exception {
    list = Stream.of(
            1,2,3,4,5,1,2,3,4,5
    ).dropWhile(integer -> integer < 5)
            .collect(Collectors.toList());
    assertThat(list.size(), is(6));
}
```

`dropWhile` 역시 마찬가지로 해당 조건을 만족하지 않는 경우가 처음 만날 때부터이므로, `Stream` 의 `filter` 와는 전혀 다른 사용성을 갖습니다.

### 3. iterate

`for문` 의 `Stream` 버전이라고 생각하면 됩니다. 이전에는 `IntStream` 을 통해 불편하게 사용될 수 있던 부분이 아주 명시적으로 사용할 수 있게 되었습니다.

```java
@Test
public void for문의_Stream_버전_iterate() throws Exception {
    list = Stream.iterate(1, x-> x < 11, x->x+3).collect(Collectors.toList());
    assertThat(list.size(), is(4));

    //list : [1,4,7,10]
}
```

### 4. ofNullable

값이 null 인 경우 빈 Optionals를 반환하는 기능 입니다. 간혹은 Stream으로 전환을 위해 불필요한 `NullPointerExceptions` 확인이 필요했었습니다. null 검사를 회피할 수 있도록 도와주는 기능이라고 생각됩니다.

```java
@Test
public void null을_허용하는_ofNullable() throws Exception {
    list = Stream.<Integer>ofNullable(null).collect(Collectors.toList());
    assertThat(list, notNullValue());
    assertThat(list.size(), is(0));
}
```

---

마무리
------

이상 자바 9의 핵심 기능 외에도 소소하게 유용하게 써먹을 수 있는 기능들을 정리해보았습니다.

이 글을 쓰려고 한참 전부터 미루다가 작성을 하게 되었는데, 그동안 고수님들이 언어와 성능을 생각하며 위 내용에 대한 대화를 나누시는 걸 보고, 앞으로도 더 열심히 자바 공부를 해야겠구나 느꼈습니다.ㅜㅜ
