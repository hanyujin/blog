---
title: Under-the-hood-ReactJS(1) - 마운트
categories: React
tags:
  - React
  - javascript
keywords:
  - React
date: 2017-08-11 09:18:00
---

Under-the-hood-ReactJS를 읽어보면서 이해한 부분을 정리해보았습니다. 
Intro ~ 파트7까지의 마운트 부분에 대한 설명이 있습니다.

<!-- more -->

처음 뭔가를 배울때는 아무것도 모르는 상태에서 책이나 다른 문서의 예제를 약간씩 따라 치면서 익히곤 합니다.

그러다 스스로 뭔가 대충 "아, 정확히는 모르겠지만 이건 요런 방식이구나..." 라고 생각되는 시점이 오면 "그런데 이게 어떻게 되는거지?"라고 생각하게 되는 것 같은데 이때 [Under-the-hood-ReactJS](https://bogdan-lyashenko.github.io/Under-the-hood-ReactJS/)를 알게되었습니다.

영어실력이 좋지 않아서 번역기의 도움을 받으면서 몇 번씩 읽어보았고 제가 이해한 방식으로 해석하는 작업을 몇 차례 반복했습니다. 한달정도의 시간동안 틈틈히 공부 했지만, 그럼에도 이해 안된 부분들이 조금씩은 있어서, 번역된 내용중에 '한글인데 뭔말인지 모르겠다...' 란 부분은 바로 그러한 부분이니 참고 부탁드립니다. :)

현재 저는 올초부터 리엑트를 공부하기 시작했지만, 실제 총 시간으로 따지만 약 두달 정도만 리엑트에 집중했던것 같습니다. 이것을 말하는 이유는 저와 비슷한 초보자들도 읽어볼 수 있을 만한 글이라는것을 말씀드리고 싶어서 입니다.

다음은 저자가 14개의 파트로 나눠서 작업한 부분들을 번역하면서 간단히 요약해 놓은 부분입니다. 

---

### Intro

- 전체적으로 두가지 프로세스를 다루고 있습니다 : **'마운트'와 '업데이트'**
	+ 마운트 : 파트 1~7 / 시작 : ReactDOM.render
	+ 업데이트 : 파트 8~14 / 시작 : component.setState 
	
- 코드에 상응하는 전체 스키마가 있는 것은 아닙니다.

- 마지막에 보이는 샘플 코드는 앞으로 나오는 파트들에 꾸준히 예제로 쓰이고 있습니다.

{% alert info %}
각 파트별로 해당되는 스키마를 잘라서 보여주기는 합니다만 **전체 스키마**와 **샘플코드**를 항상 띄워 놓고 다음 파트들을 살펴보면 보면 이해가 좀 더 수월한 것 같습니다.
{% endalert %}
___

### 파트 0

#### ReactDom.render

- 앱의 시작은 ReactDom.render에서 부터 시작됩니다.
   + JSX가 리엑트의 엘리먼트되는 시작이며, 이건 단순히 plain objects에 불과합니다.
   + 실제로, ReactDOM에는 로직이 없고, `ReactMount`를 사용하기 위한 인터페이스일 뿐입니다. 따라서 `ReactDOM.render`를 호출하면 기술적으로 `ReactMount.render`를 호출합니다.

> **마운팅이란?** 
  DOM 엘리먼트를 생성하고 제공된 container에 삽입하여 리엑트 컴포넌트를 초기화하는 작업입니다.

#### React 컴포넌트 인스턴스화

- 가장 먼저 TopLevelWrapper의 인스턴스화가 먼저 일어나면서 트리를 렌더링 합니다.

{% alert info %}
JSX를 작성해보셨다면, JSX 사용시 하나의 root 엘리먼트로 감싸주어야 한다는 사실을 알고있으실 텐데요. 그게 바로 TopLevelWrapper라고 생각하시면 될 것 같습니다.
{% endalert %}

- JSX는 내부 컴포넌트로 변환됨

- 내부 컴포넌트의 종류
   + ReactCompositeComponent : 자체 컴포넌트 클래스
   + ReactDOMComponent : HTML 태그 클래스
   + ReactDOMTextComponent : text nodes 클래스

- Virtual DOM
   + Virtual DOM은 리엑트가 diff 중에 DOM을 직접 건드리지 않는데 사용되는 DOM 표현의 일종.(개념)
   + Virtual DOM은 위 3개의 내부 컴포넌트를 참조.

{% alert info %}
마운트라는 개념을 잘 생각해보고, 내부 컴포넌트의 종류 잘 기억하는게 좋은것 같습니다. 
{% endalert %}
___


### 파트 1

#### Transaction(1)

- `ReactUpdates`모듈을 통해 컴포넌트 인스턴스가 리엑트 생태계에 연결 되도록 합니다.

- 리엑트는 chunks 형태로 업데이트를 수행하기 때문에 사전/사후 처리를 한번만 적용할 수 있도록 트렌젝션을 활용합니다.

- 리엑트에서 트렌잭션이 어떻게 사용되고 있는지 말하고 있습니다.
 	+ `Transaction` 모듈에서 확장하여 사용되며, **어떤 트렌잭션 래퍼로 감싸는지에 따라 목적이 달라집니다.**
 	+ 래퍼는 기본적으로는 초기화 및 클로즈 메소드를 포함한 객체입니다.

- `ReactDefaultBatchingStrategyTransaction`레퍼에는 'FLUSH_BATCHED_UPDATES', 'RESET_BATCHED_UPDATES'래퍼가 존재합니다.
	+ 내부에서 `initialize` 메소드가 비어있지만, `close`에는 `ReactUpdates.flushBatchedUpdates`를 호출합니다.
	+ `ReactUpdates.flushBatchedUpdates`는 dirty 컴포넌트에 대한 검증을 시작합니다. (파트 8 이후)

{% alert info %}
해당 컴포넌트가 마운팅 되기전에 일련 작업들을 한번에 처리하기위해 **트렌젝션**을 사용해서 진행되고 있습니다.
{% endalert %}
___


### 파트 2

#### Transaction(2)

- 파트 1의 트렌젝션에서의 메소드 실행부분에서 발생되는 또 다른 트렌잭션.

- 여기에서는 트랜젝션이 다음을 제어하는데 사용됩니다.
	+ 현재 선택한 영역이 다른 영역들로부터 침범되지 않도록 합니다.
	+ 리플로우가 발생되는 정도의 DOM 조작 발생 가능성이 있는 이벤트를 제한합니다.
	+ 이런 과정을 `initialize` 할 때 제한하고, `close` 할때 사용하도록 설정합니다.

- 파트 2의 메소드 실행시 `ReactReconciler.mountComponent`로 DOM에 넣을 준비가 된 마크업을 리턴합니다.

{% alert info %}
랜더링을 위한 마크업을 만들기 전에 다른 영역에 영향을 미칠 수 있는 부분을 제한하는데 이를 **트랜젝션**을 활용하고 있다는 것을 알 수 있습니다.   
{% endalert %}
___

### 파트 3

#### Mount

- 컴포넌트의 트리에 처음 삽입되는 컴포넌트는 TopLevelWrapper 입니다.
  + 이후에 자식의 자식들도 같은 방식으로 마운트 됩니다. 

- ExampleApplication 컴포넌트에 대한 ReactCompositeComponent 인스턴스 생성(new ExampleApplication())
	+ 생성자를 통해 인스턴스에 props, context, refs, updater가 할당됩니다.
	  (updater는 setState에 의해 사용됩니다.)

#### 초기 마운트 수행

- 초기 마운트 수행
  + componentWillMount가 호출됩니다.
  + updater가 할당되었기 때문에 componentWillMount안에서 setState를 사용할 수는 있습니다.
 	  하지만 render()전에 호출되기 때문에 컴포넌트가 아직 마운트 되지 않았기 때문에, state를 알 수 가 없어 state를 설정해도 리렌더링이 발생되지 않습니다.
  + componentDidMount 트랜젝션 큐에 넣어둡니다.(이후에 실행되기 때문에) 	
  + render()가 호출됩니다.

- render 메소드에서 얻은 엘리먼트를 기반으로 그 자식에 대한 가상 컴포넌트(ReactDOMComponent 인스턴스)를 생성합니다.
  + ReactReconciler.mountComponent() 호출합니다.
  
{% alert info %}
- 코드랑 매칭 시켜서 이해해 볼 수 있는 첫번째 부분입니다. `constructor`, 라이프 사이클 메소드인 `componentWillMount`와 `componentDidMount`를 연결시켜 볼 수 있습니다.
- 컴포넌트의 인스턴스를 만들면서 ReactCompositeComponent(내부 컴포넌트)를 만들고, 그 자식의 인스턴스를 만들어 ReactDOMComponent(HTML 컴포넌트)까지 생성합니다.
{% endalert %}
___

#### 파트 4

- HTML 태그에서 video, form, textarea의 경우에는 추가 래핑을 진행합니다.

- 내부 props가 올바르게 설정되었는지 확인하기 위해 유효성 검사 메서드를 호출합니다.

- 실제 HTML 엘리먼트는 document.createElement에 의해 생성됩니다.
___

#### 파트 5

- 이전 props와 새 props의 차이를 계산합니다.
  + 순서 : 스타일 업데이트 -> 트랜잭션 패턴을 이용해 이벤트 처리 -> attribute, property 업데이트
___

#### 파트 6

- 자식 요소들을 관리하는 `ReactMultiChild` 모듈의 `mountChildren`메소드를 통해 자식요소들을 인스턴스화하고 마운팅합니다.

- 전반적인 마운팅 프로세스에 대한 전체적인 확인
	+ 커스텀 컴포넌트 인스턴스 생성(`constructor` 호출)
	-> render 메소드 호출로 랜더링 진행 하고 `React.createElement`로 엘리먼트 생성
	-> 커스텀 컴포넌트가 리턴된 경우 'ReactCompositeComponent'로 인스턴스 화하고 HTML인 경우 `ReactDOMComponent`로 인스턴스화 함
	-> 반복하면서 DOM 엘리먼트가 생성되면 이벤트 리스너를 할당하면서 DOM 컴포넌트를 마운트합니다.
	-> 다음 자식들을 이어서 위의 동작들을 반복합니다.
___

### 파트 7

- 마운팅 메소드 실행의 결과로 document에 세팅할 준비가 된 HTML 엘리먼트를 가질 수 있었지만, 실제로는 HTML 마크업이 아닌, children, node(실제 DOM 노드)등의 필드를 가진 데이터 구조입니다.

- parentNode.insertBefore로 인해 드디어 document에 엘리먼트가 추가됩니다.

- 아직 트렌젝션의 중간 단계이기 때문에 상위 두 단계의 트렌젝션을 모두 닫아줍니다(파트2).

---

여기까지가 마운팅 프로세스에 대한 일련의 과정입니다.
