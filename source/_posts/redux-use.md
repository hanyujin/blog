---
title: Redux 코드로 이해하기
categories: 
  - js
  - lib
  - react
tags: 
  - react
  - redux
keywords:
  - react
thumbnailImage: https://raw.githubusercontent.com/Jae-kwang/blog/master/source/img/redux.png
date: 2017-06-21 08:17:00
---

### 리덕스를 코드로 이해해보자

리엑트를 공부하다 보면 자연스럽게 리덕스를 접하게 됩니다.
리덕스를 이해하기 위해 많은 문서를 접하지만, 저 같은 경우에는 코드랑 연결이 되어 있을 때 가장 잘 이해하는 것 같습니다. 그래서 코드로 리덕스를 접근해보려고 합니다.

<!-- more -->


### 리덕스의 단어들

"리덕스는 뭐다."라는 부분은 많은 문서에서 잘 정리되어 있고, 저 또한 그런 곳을 많이 참조하여 읽습니다.
그래서 저는 제가 이해한 만큼에서 직접 사용되는 리덕스의 단어들을 코드로써 이야기해 보려고 합니다.

#### 1. 스토어(Store)
가장 먼저 스토어 입니다. 스토어는 리덕스를 처음 접하자마자 들을 수 있는 단어로 __state(상태 값)__총괄하는 부분입니다. state는 각 컴포넌트가 갖는 특정한 값일 수도 있으며, UI의 상태를 나타낼 수 있습니다.

{% codeblock 스토어 형태 lang:javascript %}
const initState = {
    color: 'black',
    number: 0
}
{% endcodeblock %}

컴포넌트들은 이 스토어에 액션을 dispatch하고, subscribe 하며 상태를 반영할 수 있습니다.
액션, dispatch는 천천히 알아보도록 하죠. 

#### 2. 액션(action)
스토어에 대한 값을 변경하기 위해서 어떤 신호를 보내주어야 하는데 바로 그게 바로 액션입니다.

예를 들어 컴포넌트에서 어떤 이벤트가 발생한 경우 컴포넌트가 "이벤트가 발생했어! 값을 변화시켜줘!"라고 말하는 상태에 대한 변경 값이 들어 있습니다.

액션은 무엇이냐? 그냥 객체입니다.
다만, 한가지 알아두어야 하는 건 모든 액션객체는 __'type'__이라는 값을 지닙니다.
{% code 액션 형태 lang:javascript %}
{
    type: "INCREMENT"
}
{% endcode %}

추가로 이벤트 발생 후 변경할 값을 같이 전달해 주려면 아래와 같이 단순히 값을 추가하면 됩니다.

{% code 액션 + 값 추가 lang:javascript %}
{
    type: "SET_COLOR",
    color: "black"
}
{% endcode %}

#### 3. 리듀서(reducers)
리듀서는 액션을 받아서 스토어의 값을 처리하는 작업을 해줍니다.
사실, 스토어는 그냥 저장소일 뿐 아무것도 하지 않습니다. 이 리듀서가 바리바리 움직여서 스토어의 값을 변화시켜줍니다.

그럼 코드로서는 어떨까요? 리듀서는 스토어의 값을 변형시켜준다고 했습니다.
위에서 언급된 것처럼 스토어는 값을 담은 객체일 뿐이고 리듀서는 그 값을 변경한다고 했으니 스토어라는 객체의 값을 변경시켜주는 함수일 뿐이죠.

대신 액션을 받아서 어떤 값인지 판단해서 어떻게 변경해줘야 할지 분기해야 햐죠.
그럼 그 부분을 코드로 작성해보면 아래와 같이 됩니다.

{% code 스토어 + 리듀서 lang:javascript %}
// 스토어
const initState = {
    color: 'black',
    number: 0
};

// 리듀서
// 1. state에 store로 설정한 값을 초기값으로 넣습니다.(처음 스토어가 생성됩니다.)
// 2. action에는 지정한 action 객체가 들어옵니다.
function exampleReducers(state = initState, action) {
    switch (action.type) {
        case types.INCREMENT: 
            return {
                ...state,
                number: state.number + 1
            };
        case types.SET_COLOR:
            return {
                ...state,
                color: action.color
            };
        default:
            return state;
    }
};

export default exampleReducers;
{% endcode %}

### 4. 액션 생성자(Action Creator)
사실 액션 설명할 때 액션 생성자를 같이 설명하는데, 리듀서 다음에 나온 이유는 제가 이해한 흐름대로 나열하기 때문이죠.

처음 예제들을 참고하며 공부했을 때 혼동이 있었던 부분이었습니다.
액션을 리듀서에 전달할 때 "액션 타입을 지정하는 파일(ActionTypes.js)을 만들고, 액션 생성자를 통해서, 리듀서로 전달한다."라는 말이 직관적으로 머리에 들어오지 않았습니다.

그래서 저는 저 나름대로 이해를 했습니다.

{% quote %}
Q. 액션 생성자를 만드는 이유.
A. 매번 객체(액션 객체)를 만들기 귀찮아서.
{% endquote %}

{% quote %}
Q. ActionTypes.js란 파일을 만드는 이유.
A. 매번 문자열로 전달하면 귀찮아서.
{% endquote %}

### 5. 디스패치(dispatch)
액션 생성자와 디스패치의 설명 순서는 좀 애매했습니다.
액션 생성자를 설명할 때 "귀찮다"가 언제 인지를 알아야 사용의 편리함을 아니까요.

바로 디스패치를 통해 액션을 전달할 때 귀찮습니다.

이 단락에 액션 생성자 코드를 넣는 건 이상하지만 흐름으로는 이게 맞아 보여서 여기에 쓰겠습니다.
정말 이름 그대로 액션이란 객체를 만들어주는 객체(액션) 생성자일 뿐입니다.

{% code 액션생성자 lang:javascript %}
// () => ({}) 은, function() { return { } } 와 동일한 의미
export const increment = () => ({
    type: types.INCREMENT
});
{% endcode %}

디스패치는 컴포넌트에서 특정한 이벤트 발동 시 실행이 됩니다.
디스패치는 액션(객체)을 스토어로 넘겨주는 이벤트 트리거라고 생각하면 될 것 같습니다.

{% code 컴포넌트 lang:javascript %}
import * as actions from '../actions';

const mapDispatchToProps = (dispatch) => ({
    onIncrement: () => dispatch(actions.increment()),
});
{% endcode %}

onIncrement()가 실행되면 actions.increment()란 액션 생성자를 통해 액션객체를 생성하고 그 액션객체를 dispatch로 리듀서에 전달되는 겁니다.

### 6. react-redux

그렇다면 이 모든 과정에서 어떻게 액션객체를 컴포넌트에서 스토어로 스토어에서 컴포넌트로 공유가 될 수 있을까요? 바로 'react-redux' 가 해줍니다.

#### 6-1. Provider

redux 에서 store는 단 하나입니다. 그렇다면 이게 어디에 위치하는 게 좋을까요?
바로 최상단에서 생성되고 하위로 흐르는 방식입니다.

이는 Provider로 전달할 수 있습니다.
{% code Provider로 store 전달 lang:javascript %}
// Redux 관련 불러오기
import { createStore } from 'redux'
import reducers from './reducers';
import { Provider } from 'react-redux';

// 스토어 생성
const store = createStore(reducers);

// Provider를 사용해서 스토어를 전달합니다.
ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('root')
);
{% endcode %}

이렇게 하면 스토어가 컴포넌트로 흐르죠.
그럼 컴포넌트에서는 스토어의 state를 어떻게 받으며, action은 어떻게 스토어로 전달이 될까요?

#### 6-2. Connect

connect()를 통해 state를 가지며 dispatch를 사용할 수 있습니다.

{% code connect로 action 전달 lang:javascript %}
import exampleComponent from '../components/exampleComponent';
import { connect } from 'react-redux';

// store의 값을 컴포넌트에게 props 형태로 전달합니다.
const mapStateToProps = (state) => ({
    color: state.color,
    number: state.number
});

// dispatch를 통해 리듀서로 액션생성자를 전달합니다. props형태로 전달됩니다.
const mapDispatchToProps = (dispatch) => ({
    onDecrement: () => dispatch(actions.decrement())
});

// connect를 사용하여 스토어와 컴포넌트와 연결을 해줍니다.
export default connect(mapStateToProps, mapDispatchToProps)(exampleComponent);
{% endcode %}

### 7. 미들웨어(middleware)
미들웨어는 이름답게 중간에서 뭔가를 처리해줍니다. 그 중간은 바로 액션이 리듀서로 전달되는 사이입니다.

액션 생성자를 통해 액션이 생성되고 dispatch를 통해 리듀서로 전달되기 전에 어떠한 동작을 추가할 수 있습니다.
예를 들어 콘솔을 기록할 수도 있고 액션을 취소시킬 수도 있죠. 다른 액션을 디스패치 할 수도 있습니다.

미들웨어는 redux 의 applyMiddleware()를 사용해 적용할 수 있습니다.

{% code redux-logger 미들웨어 사용하기  lang:javascript %}
import { createStore, applyMiddleware } from 'redux';
import modules from './modules';

import { createLogger } from 'redux-logger';

const logger = createLogger(); 

const store = createStore(modules, applyMiddleware(logger))

export default store;
{% endcode %}

### Redux flow
아래의 그림은 이해한 부분들을 그림으로 그려놓은 Flow입니다.

{% image redux-cycle.png %}


