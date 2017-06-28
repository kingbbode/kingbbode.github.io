---
layout:     post
title:      Jquery Plugin 소개 - form 데이터를 Object로 만들기
author:     kingbbode
tags:       web javascript
subtitle:   form 데이터와 실제 서버 데이터 스펙을 어떻게 맞추고 계신가요? 유용한 Jquery Plugin을 정리 및 소개합니다.
category:  posts
outlink: 0
---

이번 포스팅에서는 유용한 Jquery 플러그인을 소개하려고 합니다.

![main](/images/2017/2017-02-08-JQUERY-SERIALIZEOBJECT/main.png)

`form` 데이터를 `Object`로 바꿔주는 것을 도와주는 [Jquery serializeObject 플러그인](https://plugins.jquery.com/serializeObject/) 입니다.

### 왜 `form` 데이터를 `Object` 변환하는 것이 필요한가?

HTML에서 사용자의 데이터를 서버에 전송하는 방법은 `form`을 사용하는 것 입니다. `form`은 form 내부의 type이 submit인 input 태그의 작동으로 인해 HTML의 기본 인터렉션이 실행되어 데이터를 전송하게 됩니다.

SPA가 활성화되면서 대부분의 어플리케이션에서 기존 인터렉션의 방식 대신, 여러가지 방식으로 submit을 intercept하여 데이터를 비동기로 전송하고 있습니다. 이 때 필요한 것이 `form` 데이터의 `Object` 변환입니다.

많은 프론트 프레임워크들이 SPA를 쉽게 지원하기 위하여 등장하였지만, Angularjs를 제외한 대부분의 프레임워크는 데이터를 단방향으로 바인딩하기 때문에, 결국에는 사용자가 작성한 데이터를 각 프레임워크의 특정 구조로 바인딩 시켜야하기 때문에, `form` 데이터의 `Object` 변환은 불가피해 보입니다.

### Dom을 탐색하여 값을 셋팅하고 있지는 않은가?

데이터를 비동기로 전송하기 위하여 데이터를 Dom 탐색하고 있진 않으신가요?

**HTML**

```html
<form id="form">
  <input type="text" id="email" name="email"/>
  <input type="text" id="age" name="age"/>
</form>
```

**javascript**

```javascript
var email = $('#email').val();
var age = $('#age').val();
var data = {
  email : email,
  age : age
};
```

데이터가 적을 때는 간단할 수 있겠지만, 데이터가 많아질 수록 셋팅하기 힘들어지고, `form`을 수정하면 `javasript`도 수정해야하는 정말 귀찮아지는 코드가 될 것 입니다.

![omg](/images/2017/2017-02-08-JQUERY-SERIALIZEOBJECT/omg.png)

**이쯤 되면 `form`을 `Object`로 만들어주는 것은 필요한 일이라는 것을 아실 겁니다!**

### 직접 구현해서 사용하면 되지 않은가?

검색을 통해 찾아보면 `serializeObject`를 직접 구현하면 된다는 수많은 글들을 볼 수 있습니다.

`form` 데이터를 `Object`로 만드는데에 핵심적인 기술은 `Jquery`에서 지원하는 `serializeArray`입니다.

`serializeArray`는 `form` 내부의 input 태그의 특정 type의 name 속성을 key, 값은 value로 갖는 Object들의 배열을 만들어줍니다.

**HTML**

```html
<form id="form">
  <input type="text" name="email"/>
  <input type="text" name="age"/>
</form>
```

**javascript**

```javascript
$("#form").serializeArray();

결과 :
[{"name":"email","value":"kingbbode@gmail.com"},{"name":"age","value":"29"}]
```

`serializeObject`는 `serializeArray`를 사용하여 직접 구현할 수 있습니다.

```javascript
jQuery.fn.serializeObject = function() {
  var obj = null;
  try {
    if(this[0].tagName && this[0].tagName.toUpperCase() == "FORM" ) {
      var arr = this.serializeArray();
      if(arr){
        obj = {};    
        jQuery.each(arr, function() {
        obj[this.name] = this.value;
        });             
      }
    }
  }catch(e) {
    alert(e.message);
  }finally  {}
  return obj;
}

결과:
{
  email : "kingbbode@gmail.com",
  age : "29"
}
```

*출처: http://seongilman.tistory.com/277 [SEONG]*

비교적 간단하게 구현 가능합니다.

**그럼에도 플러그인를 추천하는 이유가 당연히 있습니다**

### 실제 데이터와 같은 형태의 `form` 데이터를 원한다!

**서버로 보내야할 데이터 스펙**

```json
{
  title : "",
  statudents : [
    {
        name : "",
        age  : ""
    },
    {
        name : "",
        age  : ""
    },
    {
        name : "",
        age  : ""
    }
  ]
}
```

서버로 보내야할 데이터 스펙을 위와 같이 정의해놓고 플러그인을 쓰면 무엇이 좋은지를 설명하겠습니다.

#### 직접 구현한 간단한 스크립트의 문제점

**HTML**

```html
<form id="form">
  <input type="text" name="title"/>
  <input type="text" name="name1"/>
  <input type="text" name="age1"/>
  <input type="text" name="name2"/>
  <input type="text" name="age2"/>
  <input type="text" name="name3"/>
  <input type="text" name="age3"/>
</form>
```

**serializeObject 결과**

```json
{
  title : "",
  name1 : "",
  age1  : "",
  name2 : "",
  age2  : "",
  name3 : "",
  age3  : ""
}
```

Object의 key 값이 중복될 수 없기 때문에, name을 다르게 정의를 하여 object로 만들어야 합니다. 딱 보아도 원했던 데이터 스펙과는 많이 다릅니다.

그럼 여기서 나온 값으로 원하는 데이터 스펙으로 다시 변환하는 과정을 거친다면? **결국 dom 탐색을 했을 때 두번 일하는 번거로운 코드가 된다는 것보다 더 귀찬은 코드가 될 것 입니다.**

그럼 key 값이 중복일 때 array로 만들어주거나 object화하는 기능을 추가하여야 할 듯 합니다.

그럼 코드가 간단하지는 않겠네요...

![omg](/images/2017/2017-02-08-JQUERY-SERIALIZEOBJECT/omg.png)

**이쯤되면 그냥 플러그인 쓰는게 좋겠다는 결론이 나옵니다.**

#### 플러그인은 무엇을 해주는가?

플러그인의 기본 기능은 위 직접 작성한 기능과 같습니다. 거기에 더하여 이 플러그인의 가장 큰 장점이라고 생각되는 것은 name을 통한 객체화 방법입니다.

**push**

동일한 이름을 갖는 name은 array로 묶어줍니다.

```javascript
<input name="foo" value="a">
<input name="foo" value="b">

결과:
{foo: ["a", "b"]}
```

**fixed**

숫자인자를 갖는 []로 선언시 value는 array의 특정 index에 추가됩니다.

```javascript
<input name="foo[2]" value="a">
<input name="foo[4]" value="b">

결과:
{foo: [,"a", ,"b"]}
```

**named**

문자인자를 갖는 []로 선언시 인자를 key 값, value를 value로 한 object를 생성합니다.

```javascript
<input name="foo[bar]" value="a">

결과:
{foo:
  {
    bar : "a"
  }
}
```

기능들을 조합하면 다양한 형태로 Object 구성이 가능합니다.

```javascript
<input name="foo[0][bar]" value="a">
<input name="foo[0][baf]" value="b">
<input name="foo[1][bar]" value="c"
<input name="foo[1][baf]" value="d">
<input name="foo[2][bar]" value="e"
<input name="foo[2][baf]" value="f">

결과:
{foo:
  [
    {
      bar : "a",
      baf : "b"
    },
    {
      bar : "c",
      baf : "d"
    },
    {
      bar : "e",
      baf : "f"
    },
  ]
}
```

자세한 지원 기능은 [GitHub에 명세된 API](https://github.com/macek/jquery-serialize-object)를 확인해보시길 바랍니다.

#### 플러그인을 사용한 객체화

**HTML**

```html
<form id="form">
  <input type="text" name="title"/>
  <input type="text" name="statudents[0][name]"/>
  <input type="text" name="statudents[0][age]"/>
  <input type="text" name="statudents[1][name]"/>
  <input type="text" name="statudents[1][age]"/>
  <input type="text" name="statudents[2][name]"/>
  <input type="text" name="statudents[2][age]"/>
</form>
```

**serializeObject 결과**

```json
{
  title : "",
  statudents : [
    {
        name : "",
        age  : ""
    },
    {
        name : "",
        age  : ""
    },
    {
        name : "",
        age  : ""
    }
  ]
}
```

원했던 정확한 데이터가 나옵니다. 이제 불필요한 2차 작업은 필요 없을 것 입니다! 플러그인을 통해 더 깔끔하고, 유지보수하기 쉬운 코드가 작성됐다고 생각됩니다.

#### 플러그인 단점

name 값이 변경되었기 때문에, form의 기본 인터렉션은 사용이 어려울 것으로 보입니다. 비동기만을 위한 form 작성에 적합할 것 입니다.

마치며
------

![th](/images/2017/2017-02-08-JQUERY-SERIALIZEOBJECT/th.png)

이 플러그인을 소개하게 된 개기는 후배 개발자가 플러그인의 `serializeObject`를 사용하며 겪은 문제점을 함께 해결해가면서 얻은 정보를 공유하기 위함입니다.

`form` 데이터와 원하는 데이터 스펙을 쉽게 통일 시켜 사용할 수 있게 하여 단방향 바인딩의 일부 단점을 보완해줄 수 있는 좋은 플러그인이라고 생각했습니다.

웹상 관련된 한글 문서를 보지 못하기도 하였으며, 정작 이 플러그인을 많은 곳에서 쓰고 있지만 이것이 플러그인인지도 모를 수도 있겠단 생각이 들었습니다.

기본 인터렉션을 사용할 수 없는 문제가 있지만, SPA를 구성할 때는 적절하고 좋은 플러그인일 듯 합니다.
