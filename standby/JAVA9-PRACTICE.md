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

```

```java

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
