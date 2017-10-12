2017년 9월 21일 Java 9 이 출시되었습니다. 크게 부각되고 있는 기능은 `Jigsaw`, `Reactive Streams`, `REPL/JShell` 가 있습니다.

출시 전부터 기대를 많이 받던 기능들이지만, 학습비용이 어마어마할 것 같습니다 ..!

그래서 저는 일단 제가 바로 쓸 수 있을만한, 코드 짜는 것을 더 편하게 만들어줄 수 있는 Java 9 의 새로운 기능들을 소개해보려고 합니다.

예제는 이미 추석 전에 모두 작성했었는데, 어마어마한 게으름에 추석이 지나고서야 글을 작성하게 되었습니다. 배그가 너무 재밌네요.

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
