---
title: Under-the-hood-ReactJS(2) - 업데이트
categories: React
tags:
  - React
  - javascript
keywords:
  - React
thumbnailImage: https://raw.githubusercontent.com/Jae-kwang/blog/master/source/img/react.png
date: 2017-09-28 09:18:00
---

Under-the-hood-ReactJS의 파트 8~10인 업데이트 관련 부분입니다.

<!-- more -->


### 파트 8

#### this.setState

{% alert info %}
인트로에서도 언급되었듯 업데이트의 시작은 **this.setState**에서 부터 다뤄집니다.
{% endalert %}

- setState는 `ReactComponent`에서 컴포넌트를 상속받았고, `updater` 속성을 받습니다. (파트3)
- 각 컴포넌트는 자신의 pandding state의 목록을 가지고 있고, 하나의 트랜잭션에서 setState를 호출 할 때마다 그 객체를 대기열에 넣은 다음 나중에 하나씩 컴포넌트 state로 머지합니다.
- setState를 호출하여, 컴포넌트를 dirtyComponents 리스트에 추가 할 수 있습니다.
  + dirtyComponents는 변경된 항목들을 비교하기위해 넣어두는 리스트 입니다. (파트 10)
___

### 파트 9

#### setState 메소드가 호출되는 두가지 경우

- 마우스 이벤트에 의한 호출
  + 마우스 이벤트가 발생하면 앞서 살펴본 것처럼 여러 레이어를 통해 일괄(batched)업데이트 되는데, ReactReconcileTransaction 래퍼가 비활성화되고 이후 ReactEventListener enabled때 다시 발생될때 안전하게 마운팅됩니다. (파트 2)
- setTimeout에 의한 호출
  + setTimeout의 경우에도 위의 경우와 다르지 않게 트랜잭션이 아직 오픈되지 않았을 경우 일괄(batching) 트렌젝션 열고 dirtyComponents리스트에 영향받은 컴포넌트 추가하고 트렌젝션을 close(ReactUpdates.flushBatchedUpdates)하고 dirtyComponents를 처리합니다.

___

### 파트 10

#### Dirty components

- dirtyComponents리스트를 처리하기 위해 루프를 수행합니다.
- 이때 ReactUpdates.runBatchedUpdates 호출합니다.
	+ 새로운 또 하나의 트렌젝션 **ReactUpdatesFlushTransaction 래퍼** 사용
	+ dirtyComponentsLength가 변경된 경우 flushBatchedUpdates를 통해 한번 더 수행
- ReactUpdatesFlushTransaction 래퍼(dirtyComponents 배열을 지우고, 마운트 준비 핸들러(예: componentDidUpdate)가 대기열에 추가 한 모든 업데이트를 수행)
	+ dirtyComponets배열 정렬(mount order)
	+ dirtyComponentsLength을 기준으로 반복해서 수행.
	+ 각 컴포넌트를 ReactReconciler.performUpdateIfNecessary로 전달.
		* 실제로 performUpdateIfNecessary메소드는 ReactCompositeComponent.updateComponent에서 호출합니다.
___

### 파트 11

#### Update component

- setState가 호출되거나 props가 변경된 경우 updateComponent 메소드가 호출될 수 있습니다.
- pending state queue기반으로 nextState를 비교합니다. (예제 에서의 {message : "click state message"})
- shouldUpdate를 디폴트값 true로 설정
	+ 실제로 `shouldComponentUpdate`가 지정되지 않은 경우에도 컴포넌트가 기본적으로 업데이트되는 이유
- 강제 업데이트인지 확인
	+ 강제 업데이트가 아닌경우 `shouldComponentUpdate` 메소드가 호출되어 `shouldUpdate`의 결과 값으로 재할당됩니다.
___

### 파트 12

#### 실제로 컴포넌트를 업데이트하는 경우

- `componentWillUpdate`가 명시되어 있다면 그것을 호출합니다.

- 컴포넌트를 리랜더링합니다.
	+ 컴포넌트의 render 메소드를 호출하고, 그에 따라 DOM을 업데이트 합니다.
	+ 이전에 렌더링된 엘리먼트와 비교하여 DOM을 실제로 업데이트해 하는지 확인합니다.(update or replace completely)
	  * shouldUpdateReactComponent(중복된 DOM 업데이트를 피하고 리엑트 성능을 향상)

- `componentDidUpdate` 호출을 대기열에 추가합니다.

___

### 파트 13

#### Receive component

- shouldUpdateReactComponent -> ReactReconciler.receiveComponent -> ReactDOMComponent.receiveComponent
  + 위와 같은 형태로 다음 엘리먼트 전달합니다.

- DOM 컴포넌트 인스턴스에 전달받은 엘리먼트를 할당하고 update 메소드를 호출합니다.

- updateComponent 메소드는 실제로 prev와 next props를 기반으로 2가지 동작을 수행합니다.
	+ DOM properties 업데이트 : _updateDOMProperties(파트 5)
	+ DOM children 업데이트 : _updateDOMChildren(파트 14)
	
___

### 파트 14

#### _updateDOMChildren

- 'ExampleApplication'는 button, ChildCmp, text string를 자식으로 가지고 있습니다.

- 해당 컴포넌트의 자식이 'content'형태인지 'complex'형태 인지 판단합니다.
	+ complex : 리엑트 컴포넌트
	+ content : 문자열이나 숫자와 같은 단순타입

- complex인 경우
	+ 부모 컴포넌트는 이전에 수행했던 시나리오와 동일한 시나리오를 통해 자식들을 전달합니다.
	+ 'content' 레벨에 도달할때까지 여러번 작업을 반복합니다.

- content인 경우
	+ 이전과 현재 텍스트가 같은지 비교하고 수정되었다면 업데이트 합니다.(replace)
	+ 텍스트 업데이트인 경우 setTextContent를 호출해 HTML노드의 내용을 수정합니다.
	+ 콘텐츠가 업데이트되고 페이지에서 사용자를 위해 리렌더링됩니다.

- 이전에 대기열에 넣어두었던 `componentDidUpdate`를 호출합니다. (파트 12)
	+ `componentDidUpdate`를 호출할 수 있는 이유
	  * dirty 컴포넌트 업데이트는 ReactUpdatesFlushTransaction로 래핑되었고, 래퍼 중 하나는 실제로 this.callbackQueue.notifyAll() 로직을 포함하므로 `componentDidUpdate`를 호출되는 것입니다. (파트 10)

---

저는 이 글들을 번역하고 공부하면서도 실제로 100% 이해했다고는 말할 수 없었습니다. 다만 제가 리엑트 이해도를 100%로 정했을때 스스로의 이해도가 10%정도의 수준이라면 15%정도 되지 않았을까라는 생각정도만 가지고 있습니다.

그 이유로는, 저는 무언가를 읽고 학습할때 글보다는 코드로 이해하는게 훨씬 이해도가 높습니다. 그래서 제가 처음보는 말들보다는 익숙한 예약어들이 많이 등장할때 훨씬 이해가 잘되는 느낌이었습니다.

그 예약어로는 라이프사이클 메소드를 들 수가 있었고, 제가 가지고 있는 라이프사이클 순서 사이사이에 위에서 알아본 일련의 과정들을 집어넣어보면서 이해했던것 같습니다.

제법 오랜시간동안 이거 한다고 다른걸 안하고 있었으니, 이제는 실제로 써먹을 수 있을지 간단히 리엑트 프로젝트를 한번 해봐야겠습니다!