---
title: Bootstrap의 tap plugins 분석
categories:
  - js
  - lib
  - bootstrap
tags:
  - javascript
  - bootstrap.js
keywords:
  - javascript
  - library
thumbnailImage: https://raw.githubusercontent.com/Jae-kwang/blog/master/source/img/bootstrap.png
date: 2017-10-11 09:10:00
---

### Bootstrap의 tabs 코드 분석 

Bootstrap은 plugin을 생성할때 코드를 어떻게 구조화하는지 tab plugin을 통해 분석해보며 이해해 봅니다.

버전은 v3.3.2을 사용합니다.

<!-- more -->

### 코드 분석의 목적

현재 export, import 혹은 require와 같은 방법을 통해 module이나 component를 효율적으로 나눠서 사용하고 있습니다.

그렇다면 위의 기능들이 없었을때는 어떻게 효율적으로 코드를 분할할 수 있었을까요?

만약 코드 구조가 현재 지원하고 있는 방식이 아니라 다른방식일 경우 어떻게 사용할 수 있을지 알기 위함입니다.

### Bootstrap의 함수 

기본적으로 Bootstrap은 plugin을 구현시 `생성자 패턴`과 `프로토타입 패턴`을 적용해서 구현하고 있습니다.
 
패턴 적용시 자신만의 인스턴스를 가지면서도 참조 방식을 통해 메서드를 공유해 메모리를 절약할 수 있습니다.

Bootstrap은 다음과 같이 익명함수로 모든 기능을 정의해서 스코프가 섞이지 않도록 합니다.

```javascript
+function ($) {
  ...
}(jQuery);

```
여기서 `+` 다음에 나오는 부분을 파서는 표현식으로 처리하며, 위와 같이 즉시 실행함수에서 사용됩니다.

만약, `+`가 없는 상태라면 파서가 `function`을 함수 표현식이 아닌 선언식으로 인식하여 뒤에 나오는 `()`가 오류로 처리됩니다.
 
> 참고 : https://stackoverflow.com/questions/13341698/javascript-plus-sign-in-front-of-function-name/13341710

### 이벤트 바인딩 

jQuery의 'on'을 API 통해 이벤트를 등록하고 있습니다.

여기서 '이벤트 위임(Event Delegation)' 방식으로 이벤트를 등록하고 있습니다. 

{% alert info %}
**이벤트 위임(Event Delegation)**
이벤트 위임은 이벤트 버블링의 장점을 활용하여, 이벤트 헨들러를 하나만 할당해서 해당 타입의 이벤트를 모두 처리하는 테크닉입니다. DOM요소 하나에만 접근해 거기에만 이벤트 핸들러를 할당하므로 비용이 훨씬 적으며, 메모리를 훨씬 조금 사용합니다.
{% endalert %}

이벤트 위임의 또 하나의 장점은 동적으로 생성되는 요소에 일일이 이벤트를 할당할 필요가 없다는 점입니다.

```javascript
$(document)
  .on('click.bs.tab.data-api', '[data-toggle="tab"]', clickHandler)
  .on('click.bs.tab.data-api', '[data-toggle="pill"]', clickHandler)
```

이벤트 대상은 '[data-toggle="tab"]'와 '[data-toggle="pill"]'이며, 이벤트 처리는 'clickHandler'에서 진행됩니다.

이때 걸려 있는 이벤트는 'click' 이벤트로, 이 이벤트는 'bs, tab, data-api'라는 namespace를 갖습니다.

### 이벤트 핸들러

이벤트에 대한 이벤트 처리가 이루어집니다. 

```javascript
var clickHandler = function (e) {
  e.preventDefault()
  Plugin.call($(this), 'show')
}
```

엘리먼트의 e.preventDefault()를 통해 기본기능을 막고, 선언된 Tab Plugin을 .call()로 'show'를 호출하며 현재 'this'를 교체합니다.

### 탭 플러그인 정의 

```javascript
function Plugin(option) {
  return this.each(function () {
    var $this = $(this)
    var data  = $this.data('bs.tab')

    if (!data) $this.data('bs.tab', (data = new Tab(this)))
    if (typeof option == 'string') data[option]()
  })
}
```

우선, 대상되는 요소의 data정보에 'bs.tab'이 있는지 확인하는 작업을 하는데 이는 이전에 이벤트가 바인딩 되어 있는지 체크하는 작업입니다.

만약 이벤트가 바인딩이 되어 있지 않다면 선언된 Tab을 new 통해 인스턴스를 생성하여 해당 요소의 data에 'bs.tab'의 값으로서 할당합니다.

**이 작업은 이벤트가 바인딩 되어있는지를 확인하는 flag로써 활용할 수 있으며, 실제로 기능을 하는 함수로 사용됩니다.**

마지막으로 option으로 넘어온 'show'를 실행시킵니다. (data.show())

### jQuery Plugin 설정

``` javascript
var old = $.fn.tab

$.fn.tab             = Plugin
$.fn.tab.Constructor = Tab

$.fn.tab.noConflict = function () {
  $.fn.tab = old
  return this
}
```

jQuery의 Plugin을 설정시 $.fn에 연결합니다. jQuery에서는 이부분에 연결된 기능을 prototype으로 연결시킵니다.

다음으로는 Constructor에 연결합니다. 그 이유는 일부 동일한 다른 함수의 경우 (popover의 경우에는 tooptip을 상속받아서 사용)에는 다른 기능을 상속받아 사용하기 때문에 어떤 유형인지 확인이 어렵습니다. 그래서 Constructor를 사용해 어떤 유형인지 확인하는데 사용될 수 있습니다.

> 참고 : https://stackoverflow.com/questions/19680895/bootstrap-constructor

$.fn.tab.noConflict를 사용해 이전에 $.fn.tab에 사용되고 있었던 플러그인을 사용할 수 있습니다. (예전에 동일한 이름으로 플러그인을 사용하고 있다는 가정하에)

### 탭 클래스 정의 

```javascript
var Tab = function (element) {
  this.element = $(element)
}

Tab.VERSION = '3.3.2'

Tab.TRANSITION_DURATION = 150

Tab.prototype.show = function () {...}

Tab.prototype.activate = function {...}

```
부트스랩은 프로토타입 상속에 의해 기능들이 정의 되어 있습니다.

인스턴스를 생성하기위한 Tab 클래스를 생성하고, 필요한 상수를 선언하며, 기능들을 prototype에 추가합니다.
 
앞서 설명된 플러그인에서 탭 플러그인 설정에서 사용되는 클래스 입니다.

Tab.prototype.show에서는 요소를 노출하기 위한 action을 trigger하는 기능들을 모아놓았으며,

Tab.prototype.activate에서는 실제 요소를 활성화 시키는 .active 클래스를 컨트롤 합니다.

### 마치며 

**"높은 응집도 낮은 결합도"**에 대한 이해가 조금이나마 와닿은것 같습니다.
예전에는 머리로 이했었던 느낌이지만 요즘은 이런게 필요하구나 라는 것을 직접 느끼고 있는것 같습니다.

사실 디자인 패턴을 공부하면서 직접 적용해서 작업하는건 쉽지 않는다고 판단이 들어서,
유명한 라이브러리들을 조금씩 열어보면서 이들은 어떤식으로 구조화를 했었고, 어떻게 진화하고 있는지 알아보려고 했습니다.

우선 제가 가장 많이 쓰고 있는 것중에 하나인 bootstrap중에서 가장 짧은 tab을 살펴 보았습니다.
더 공부해야겠지만, 이러한 방식을 더 잘 이해하면 현재 사용되는 모듈 방식을 좀 더 잘 사용할 수 있지 않을까 생각됩니다.
