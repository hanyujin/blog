---
title: 클로저(closure)란?
categories: 
  - js
  - core
tags:
  - javascript
  - closure
keywords:
  - javascript
  - closure
thumbnailImage: https://raw.githubusercontent.com/Jae-kwang/blog/master/source/img/javascript.png
date: 2017-11-13 10:02:33
---

### 클로저(closure)

자바스크립트에서 빠지지 않은 개념 중 하나인 클로저에 대해 알아봅시다.

<!-- more -->


### 아직도 클로저

자바스크립트를 사용한지 꽤 지났음에도 불구하고 다시 한번 클로저에 대한 포스팅을 올린다는것은 아직도 내가 클로저를 완벽하게 이해하지 못한다는 것을 의미하고 있는 것 같습니다.

누군가 나에게 클로저에 대하여 묻는다면, 똑부러지게 대답할 수 있을까요? '접근하려고 하는 함수의 생명주기가 종료됬지만, 내부함수가 참조 하고 있어서 그 함수에 접근할 수 있는 함수'입니다.

더 설명하라고 하면 아직 뭐라해야 할지 모르겠습니다. 다른 사람에게 설명할 수 있을 정도가 되야 아는 거라도 했던가요? 여러번 봤지만 오늘도 또 한번 공부해보려고 합니다. 같은 개념이지만 공부할때마다 이해하는 범위가 조금씩 확장되는것 같습니다. 이러한 이유로 또 한번 클로저를 공부해보려고 합니다.

### 클로저를 알기 위한 키워드

예전에는 클로저는 무엇이다라는 것에 대해서만 집중했지만, 클로저를 잘 이해하기 위해서는 몇가지 개념들을 사전에 이해해야 한다고 생각합니다.

제가 생각하는 클로저를 알기 전에 이해해야할 몇 가지 키워드들입니다.

1. 유효 범위(Scope)
2. 어휘적 유효 범위(Lexical scope)
3. 실행 컨텍스트(Execution Context)
4. 유효 범위 체인(Scope Chain)

#### 1. 유효범위(Scope)

프로그래밍 언어에서 Scope는 변수와 매겨변수의 접근성 및 생존 기간을 제어하고, 이름 충돌 문제 및 메모리 관리를 해주는 중요한 개념 중 하나입니다.  

대표적으로 C언어와 같은 언어는 `블록 유효범위(Block Scope)`가 있습니다.
1. 블록(중괄호로 묶인 문장들의 집합) 내에서 정의된 모든 변수는 블록의 바깥쪽에서는 접근할 수 있다.
2. 블록 내에서 정의된 변수는 블록의 실행이 끝나면 해제된다.

그러나 자바스크립트는 `블록 유효범위`가 아닌 `함수 유효범위(Function Scope)`를 가지고 있습니다.
(ES6 이후로는 자바스크립트도 `블록 유효범위`도 사용가능합니다.)

{% alert info %}
함수 내에서 정의된 매개변수와 변수는 함수 외부에서는 유효하지 않지만, 내부에서 정의된 변수는 함수 내부 어느 곳에서도 접근할 수 있습니다.
{% endalert %}

#### 2. 어휘적 유효 범위(Lexical scope)

자바스크립트는 `Lexical scope` 특성을 지닙니다.
Lexical scope란 Scope가 함수 실행시점이 아닌 함수 정의 시점에 정해진다는 의미입니다.

아래의 예를 보도록 하겠습니다. 
같은 함수 형태를 보이지만, 좌측은 'nero'를 우측은 'zero'를 출력합니다.

{% image lexical-scope.png %}

좌측은 함수를 처음 선언하는 순간부터 log()의 name 변수는 자신의 상위 스코프의 변수 name을 참조합니다.
그래서 Wrapper()에서 log()를 실행하기전에 참조변수인 name을 변경했기에 'nero'를 출력하고 있습니다.

그러나 우측에서는 Wrapper()안에서 새로운 name 변수를 정의 하였습니다.
하지만 log()는 이미 상위 스코의 변수인 name을 참조하고 있기때문에 'zero'를 출력하게 되는 것입니다.

{% alert info %}
다시 한번 말하자면 함수가 실행될 때가 아니라 정의될 때에 변수를 참조하게 됩니다.
{% endalert %}

#### 3. 실행 컨텍스트(Execution Context)

`Execution Context`는 실행 가능한 코드를 형상화하고 구분하는 추상적인 개념으로, 코드가 실행되는 `환경`입니다. 변수나 함수의 실행 컨텍스트는 다른 데이터에 접근할 수 있는지, 어떻게 행동하는지를 규정합니다.

자바스크립트 인터프리터가 어떤 함수를 실행시킬때마다 그 함수에 대한 새로운 실행 컨텍스트가 생성됩니다.

실행 컨텍스트가 생성되면 자바스크립트 엔진은 실행에 필요한 여러 정보들을 담을 객체를 생성합니다.
이를 `변수 객체(Variable Object)`라고 합니다. 해당 컨텍스트에서 정의된 모든 변수와 함수는 이 객체에 존재합니다.

실행 컨텍스트는 포함된 코드가 모두 실행된 후에 파괴되는데, 이때 해당 컨텍스트 내부에서 정의된 변수와 함수도 함께 파괴됩니다.

{% alert info %}
**Scope VS Context**
Scope는 변수의 가시성과 관련이 있으며, Context는 함수가 실행되는 객체를 나타냅니다.
{% endalert %}

#### 4. 유효 범위 체인(Scope Chain)

`Scope Chain`은 일종의 리스트로서 중첩된 함수의 Scope의 참조를 차례로 저장하고 있는 개념입니다. Scope Chain의 목적은 실행 컨텍스트가 접근할 수 있는 모든 변수와 함수에 순서를 정의하는 것입니다.

컨텍스트가 함수인 경우 '활성화 객체(activation object)'를 변수 객체(Variable Object)로 사용합니다.

중첩된 함수에서 변수 객체가 값을 찾지 못햇을 경우 해당 부모 컨텍스트에서 찾고 다시 그 다음 부모의 컨텍스에서 찾으며, 전역 컨텍스트에 도달할 때 까지 계속합니다. 전역 컨텍스트의 변수 객체는 항상 스코프 체인의 마지막에 존재합니다.

내부 컨텍스트는 스코프 체인을 통해 외부 컨텍스트 전체에 접근할 수 있지만 외부 컨텍스트는 내부 컨텍스트에 대해 전혀 알 수 없습니다. 컨텍스트 사이의 연결은 선형이며 순서가 중요합니다.  각 건텍스트는 스코프 체인을 따라 상위 컨텍스트에서 변수나 함수를 검색할 수 있지만 스코프 체인을 따라 내려가며 검색할 수는 없습니다.

### 클로저

클로저는 가장 맨 앞에서 언급했듯 한마디로 {% hl_text primary %}'접근하려고 하는 함수의 생명주기가 종료됬지만, 내부함수가 참조 하고 있어서 그 함수에 접근할 수 있는 함수'{% endhl_text %}라고 말할 수 있습니다.

```javascript
function funA() {
  var x = 'Hello';
  var funB = function () { console.log(x); };
  return funB;
}

var funC = funA();
funC(); // Hello 
```

funA()가 실행종료 되었으니 x또한 접근할수가 없어야 하는데 return 된 funB()를 받은 funcC()를 실행하니 x를 참조 할 수 있게 되었습니다.

### 클로저 사용시 주의해야할 점

클로저는 외부 함수의 변수를 참조 합니다. 하지만 값의 복사값은 아닙니다.

```javascript
var add_the_handlers = function(nodes) {
	for(var i=0; i<nodes.length; i++) {
		nodes[i].onClick = function(e) {
			alert(i);
		};
	}
};
```

이 함수는 노드를 클릭하면 해당 노드가 몇 번째 노드인지를 경고창으로 알려주는것이 함수의 목적이지만, 함수 전체 노드의 수만 보여주게 됩니다. 이유는 함수의 i가 함수가 만들어지는 시점의 i가 아니라 그냥 상위 스코프의 변수 i를 참조하기 때문입니다.

```javascript
var add_the_handlers = function(nodes) {
	for(var i=0; i<nodes.length; i++) {
		nodes[i].onClick = function(i) {
			return function(e) {
				alert(i);
			};
		}(i);
	}
};

```
이제 onclick에 함수를 연결되어 있는 내부 함수에서 새로 함수를 정의하고 여기에 i를 넘기면서 곧바로 실행시켰습니다. 실행된 함수는 add_the_handlers에 정의된 i가 아니라 넘겨받은 i의 값을 이벤트 핸들러 함수에 연결하여 반환합니다.
이 반환되는 이벤트 핸들러 함수는 onclick에 할당합니다. 함수가 원한는 목적대로 구현이 되었습니다.

### 결론

보통 코드를 잘때면 클로저를 생각하지 않고 자연스러운 흐름에 의해 클로저를 사용하는 경우가 많습니다. 앞으로는 그러한 부분에 있어서 좀더 의식적으로 이해하고 코드를 작성하도록 해야 하겠습니다. 

> 참고 :
- 블로그
[자바스크립트의 스코프와 클로저](http://meetup.toast.com/posts/86)
[함수의 범위(scope)](https://www.zerocho.com/category/Javascript/post/5740531574288ebc5f2ba97e)
[JavaScript : Scope 이해](http://www.nextree.co.kr/p7363/)
[유효범위의 효용](http://webclub.tistory.com/113)
[실행 컨텍스트와 자바스크립트의 동작 원리](http://poiemaweb.com/js-execution-context)
[Understanding Scope and Context in JavaScript](http://ryanmorr.com/understanding-scope-and-context-in-javascript/)
[What is the Difference Between Scope and Context in JavaScript?](https://blog.kevinchisholm.com/javascript/difference-between-scope-and-context/)
[자바스크립트 클로저(Closure)에 대해서...](https://blog.outsider.ne.kr/506)
- 책
자바스크립트 완벽 가이드
프론트엔드 개발자를 위한 자바스크립트 프로그래밍
함수형 자바스크립트
