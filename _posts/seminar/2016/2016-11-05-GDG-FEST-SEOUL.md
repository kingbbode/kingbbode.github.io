---
layout:     post
title:      GDG DevFest Seoul 2016
author:     kingbbode
tags:       developer
subtitle:   한국 Google Developers Group의 개발 컨퍼런스!
category:  seminar
outlink: 0
---

Session 1. 머신러닝
-------------------

### 기계 학습이란?

명시적으로 알려주지 않고도, 학습을 통해 일을 수행. 이미 많은 곳에서 적용한 사례들이 있다.

*사례) 알파고, Google Photos, Smart reply, ...*

### Supervised Learning

-	답이 있는(Labeled Data)
-	선형대수

#### Linear

-	Hypothesis

#### Logistic

-	답이

#### SoftMax

-	답이 2개 이상

---

-	Supervised Learning

	-	답이 있는(Labeled Data)
	-	선형대수

-	Unsupervised Learning

	-	답이 없는(Unlabeled Data)
	-	확률, 통계

-	Reinforcement Learning

**무슨 말인지 몰라서 정리 실패 ㅜㅜ**

Session 2. ES 6+ (ECMAScript 2015+)
-----------------------------------

ES 6를 관리하는 TC39라는 Task Group이 있다. 이 Task Group은 2달에 한번 정기 모임을 하고 있으며, 회의록은 공개된다.

다음과 같은 과정을 통해 검토되며 진행.

> *Strawman- proposal - draft - candidate - finished*

현재 각 제안별 스테이지 사항은 github에 공개.

Inferface
---------

-	Interface 정의에 맞는 키를 가지고 있어야 함
-	어떤 Object라도 인터페이스의 정의를 충족 가능
-	하나으 Objcet는 여러개의 인터페이스를 충족 가능

### Iterator Interface

-	next라는 Key를 갖음
-	값으로 인자를 받지 않고 iteratorResultObject를 반환하는 함수가 옴
-	iteratorResultObject는 value,done이라는 키를 갖고 있음
-	이중 done은 계속 반복할 수 있을지 없을지에 따라 boolean 값을 반환

```javascript
{
  data:[1,2,3,4],
  next: function(){
    return {
      value : this.data.pop(),
      done : this.data.length === 0
    }
  }
}
```

### Iterable Interface

-	Symbol.Iterator라는 키를 갖음
-	값으로 인자를 받지 않고 Iterator Object를 반환하는 함수가 온다.

```javascript
{
  "[Symbol.iterator]":function(){
    return {
      next : function(){
        return {value:1, done:false};
      };
    }
  }
}
```

반복
----

언어에 따라 반복이 문이 아닌 식인 경우도 있다.<br> javascirpt는 반복**문**을 사용

### while문으로 살펴보는 Iterator

```javascript
let arr = [1,2,3,4];
while(arr.legnth>0){
  console.log(arr.pop())
}
```

#### while

while(`계속 반복할지 판단(1)`){<br> `반복시마다 처리할 것(2)`<br> }

#### Iterator Interface

```javascript
{
  data:[1,2,3,4],
  next: function(){
    return {
      value : this.data.pop(), //(2)
      done : this.data.length === 0 //(1)
    }
  }
}
```

반복행위와 반복을 위한 준비를 분리 - 미리 반복에 대한 준비를 해두고 - 필요할 때 필요한만큼 반복 - 반복을 재현할 수 있음

### 사용자 반복 처리기

```
const loop = (iter, f) => {
  if(typeof iter[Symbol.iterator])
}
```

### 내장 반복 처리

```javascript
const iter = {
  [Symbol.iterator](){return this;},
  arr:[1,2,3,4],
  next(){
    return {
      done:this.arr.length == 0,
      value:this.arr.pop()
    };
  }
}
```

#### Array destructuring (배열해체)

ex)<br> iterr -> [a,b,c]

```javascript
const [a,...b] = iter;
console.log(a, b);
//a,[b,c]
```

#### Spread (펼치기)

#### Rest Pramater (나머지인자)

ex)<br> arg -> 1,2,3,4<br>

```javascript
const test = (...arg)=>console.log(arg);
//[4,3,2,1]
```

#### For of

iterator, interable을 소비하는 for.

```javascript
for(const v of iter){
  console.log(v);
}
```

#### Iterator의 구현을 돕는 Generator

```javascript
const generator = function*(max){
  let cursor = 0;
  while(cursor < max){
    yield cursor * cursor;
    cursor++;
  }
}
```

---

> 감기로 인해 두 세션만 듣고 귀가..ㅜㅜ <br>많은 개발자분들과(이번 세미나는 학생들도 엄청나게 많았음) 부스의 구인,구직 활동들, 그리고 서울대학교를 보며 자극을 받을 수 있었다,, 오늘도! 
