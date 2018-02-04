---
title: 리엑트로 프로젝트 갈아입히기
categories: 
  - js
  - react
tags: 
  - react
  - SSR
  - project
  - styled-components
keywords:
  - react
  - SSR
  - project
  - styled-components
thumbnailImage: project-react_bg.jpg
thumbnailImagePosition: bottom
metaAlignment: center
coverImage : project-react_bg.jpg
date: 2018-02-04 15:37:32
---

2018년의 첫 번째 목표는 "서비스 중인 프로젝트를 '리엑트'로 바꿔보자!"입니다. 이 목표를 수행하면서 진행한 여러 작업을 정리해보려 합니다.

<!-- more -->

### 서버사이드 렌더링 프로젝트 구축하기

프로젝트는 서버사이드 렌더링이 필요했습니다. {% hl_text primary %}SSR(Server Side Rendering){% endhl_text %}이란 말 그대로 렌더링 되어야 할 파일들이 미리 서버에서 렌더링이 되어서 내려오는 것을 의미합니다.

<p>리엑트는 Webpack을 통해 JSX로 형태로 구성된 파일들을 컴파일해서 브라우저가 읽을 수 있는 번들 파일을 만들어 화면을 렌더링하는 {% hl_text primary %}CSR(Client-side Rendering) {% endhl_text %}방식으로 렌더링하는 방식이 보통의 방식으로 사용되고 있습니다. 그러면 어떻게 리엑트가 SSR이 되도록 할까요?</p>

리엑트 사용 시 컴포넌트를 렌더링 하기 위해 'react-dom'을 import 하는데, 이때 'react-dom'에는 `renderToString`란 메서드가 존재합니다. 이는 리엑트 컴포넌트를 단순 문자열로 변환시켜서 서버에서 바로 사용할 수 있게 해줍니다.

서버는 Node.js express 구성되어있으며, 아래와 같이 기본 html 형태를 템플릿 리터럴 형태로 내려주었습니다. 이때 `renderToString` 에 리엑트 엘리먼트를 넘긴 형태를 root 엘리먼트의 자식으로 넘겨주는 방식으로 사용할 수 있습니다.

``` html
import { renderToString } from 'react-dom/server'

res.send(`
    <!doctype html>
    <html>
        <head>
            <title>SSR App</title>
        </head>
        <body>
            <div id='app'>${renderToString(<App data={preloadedState}/>)}</div>
            <script src='bundle.js'></script>
        </body>
    </html>
`)
```

이렇게 작업한 'server.js'파일을 `node server.js` 가 아닌 `babel-node server.js`로 Run하여 서버를 띄웠습니다. 또한, webpack을 통해 얻은 'bundle.js'를 추가해 상호 작용이 가능하도록 했습니다. 여기에서 `preloadedState`는 서버에서 직접 받는 데이터로써 해당 데이터를 props로 전달해 component에서 활용합니다.

### 리덕스(Redux) 설정하기

이 프로젝트는 크기가 크지 않아 처음에는 리덕스를 사용하지 않으려 했습니다. 하지만 앱의 기능 중 하나로 페이지 전체의 '언어변경'기능이 있었는데 이는 언어 변경 시 전체 페이지의 언어가 다른 국가의 언어로 변경되어야 했습니다. 그렇다는건 모든 텍스트가 직접 컴포넌트에 작성되어 있는 게 아니라 레퍼런스 값이어야 컨트롤이 가능하다는 것을 의미했습니다. __매 컴포넌트 작업 시 언어정보를 root에서 받아서 사용하는 건 매우 비효율적인 작업이 아닐 수 없었습니다.__

컴포넌트에서 즉시 값을 사용할 수 있게 리덕스를 적용해보았습니다. 'redux'의 `createStore`를 사용해 서버에서 받은 값인 'preloadedState'와 store 를 컨트롤 하기 위한 'reducers'를 넘겨서 store를 생성합니다. 생성된 store를 react와 연결하는 역할을 하는 'react-redux'의 `Provider` 사용해 리엑트에 주입합니다.

```html
import { renderToString } from 'react-dom/server'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import reducers from './reducers'

const store = createStore(reducers, preloadedStateㅇ)

res.send(`
    <!doctype html>
    <html>
        <head>
            <title>SSR Redux App</title>
        </head>
        <body>
            <div id='app'>${renderToString(<Provider store={store}><App /></Provider>)}</div>
            <script>window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState)}</script>
            <script src='bundle.js'></script>
        </body>
    </html>
`)
```
이제 컴포넌트들은 store를 통해 값을 받아 사용할 수 있습니다. 랜더링 이후에도 'bundle.js'에도 동일하게 store를 사용해야하기 때문에 브라우저의 전역 변수인 window에 임의값(`__PRELOADED_STATE__`)에 preloadedState를 넣어 주도록 합니다.

### HMR(Hot Module Replacement) 사용하기

{% hl_text primary %}HMR{% endhl_text %}이란 코드의 수정사항 발생시 브라우저의 새고로침 없이 모듈을 업데이트 해주는 기능을 말합니다. 저는 이를 위해 `webpack-hot-middleware`를 사용하기로 했습니다. `webpack-hot-middleware`는 `webpack-dev-middleware`에 의존적이기 때문에 두가지가 설치되어야 합니다.

webpack.config.js에 entry와 plugins에 다음과 같이 추가합니다.
``` javascript
entry: [
    'webpack-hot-middleware/client',
    path.resolve(__dirname, 'src')
],
plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin()
]
```

server.js에 `webpack-dev-middleware`와 `webpack-hot-middleware`를 사용하도록 설정합니다.
``` javascript
var webpack = require('webpack')
var webpackConfig = require('./webpack.config')
var compiler = webpack(webpackConfig)

app.use(require("webpack-dev-middleware")(compiler, {
    noInfo: true,
    publicPath: webpackConfig.output.publicPath
}))

app.use(require("webpack-hot-middleware")(compiler))
```

store값 변경에도 hmr이 감지하도록 reducers를 바라보도록 합니다.
```javascript
import reducers from './reducers'

const store = createStore(reducers)

if (process.env.NODE_ENV == 'development' && module.hot) {
    module.hot.accept('./reducers', () => {
        store.replaceReducer(require('./reducers').default);
    })
}
```

위에 보시면 `process.env.NODE_ENV == 'development'`가 있습니다. 이는 개발 환경에서만 이를 사용한다는 의미입니다. 이렇게 분기를 준 이유는 HRM은 일종의 가상 웹서버를 띄워서 그 웹서버를 제어해 컴포넌트 변화 시 화면을 리렌더링하는 방식입니다. 하지만 서버사이드 프로젝트로 다른 서버를 통해 렌더링 되게 프로젝트를 구성한 현재의 방식에는 적합하지 않습니다. 그렇기 때문에 랜더링 되는 부분을 개발 모드와 프로덕션 모드로 나눠줍니다.

```html
if (process.env.NODE_ENV === 'development') {
    res.send(`
        <!doctype html>
        <html>
            <head>
                <title>SSR Redux App</title>
            </head>
            <body>
                <div id='app'></div>
                <script>window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState)}</script>
                <script src='bundle.js'></script>
            </body>
        </html>
    `)
} else if (process.env.NODE_ENV === 'production')
    res.send(`
        <!doctype html>
        <html>
            <head>
                <title>SSR Redux App</title>
            </head>
            <body>
                <div id='app'>${renderToString(<Provider store={store}><App /></Provider>)}</div>
                <script>window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState)}</script>
                <script src='bundle.js'></script>
            </body>
        </html>
    `)
}
```

개발 모드에서는 `renderToString`을 사용하는 SSR이 아닌 CSR을 이용하도록 해서 HMR이 동작하도록 합니다.  

> [Live Reload / Hot Module Replacement with Webpack Middleware](https://blog.cloudboost.io/live-reload-hot-module-replacement-with-webpack-middleware-d0a10a86fc80)
[Setting Up Webpack Dev Middleware in Express](http://madole.github.io/blog/2015/08/26/setting-up-webpack-dev-middleware-in-your-express-application/)

### styled-component 사용하기

리엑트는 컴포넌트 방식으로 코드를 모듈화합니다. 그럼 컴포넌트란 무엇을 의미할까요? __저는 html, CSS, JS가 하나로 뭉쳐있어서 그것만 다른 곳에 놓더라도 바로 사용할 수 있는 것이라고 생각합니다.__ 하지만 현재의 상태로는 html과 JS는 합쳐 있지만, CSS는 그렇지 않습니다. 여전히 CSS는 다른 파일에서 작업 된 후 적용이 되어야 하죠. 그렇기 때문에 [styled-component](https://www.styled-components.com/)를 사용하기로 했습니다.

기본적으로 컴포넌트에서 styled-components를 import 해서 사용 할 수 있지만, 서버사이드 렌더링 시에는 추가로 작업해줘야 할 부분이 있습니다.

'styled-components'의 `ServerStyleSheet`를 통해 얻은 sheet 인스턴스의 collectStyles메서드로  렌더링 될 앱에 넘겨주어야 하며 sheet 인스턴스의 `getStyleTags()` 를 통해 렌더링 될 컴포넌트의 CSS를 얻어 head에 넣어줍니다.

```html
import { renderToString } from 'react-dom/server'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import reducers from './reducers'
import { ServerStyleSheet } from 'styled-components'

const store = createStore(reducers, preloadedState)
const sheet = new ServerStyleSheet()
const styles = sheet.getStyleTags()
if (process.env.NODE_ENV === 'development') {
    ...
} else if (process.env.NODE_ENV === 'production')
    res.send(`
        <!doctype html>
        <html>
            <head>
                <title>SSR Redux styled-components App</title>
                ${styles}
            </head>
            <body>
                <div id='app'>${renderToString(sheet.collectStyles(<Provider store={store}><App /></Provider>)}</div>
                <script>window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState)}</script>
                <script src='bundle.js'></script>
            </body>
        </html>
    `)
```

#### 프로젝트 구성하기

어느 정도 환경은 세팅이 되었다는 판단으로 관련 파일들을 어떻게 구성해야 할지 고민이 많았습니다. 대략적인 구조를 오픈소스들을 참고하며 구조를 잡아 보았습니다.

1. <b>containers, components</b>
 - components : 재사용 가능한 컴포넌트들을 관리합니다
 - containers : 프로젝트는 단일 페이지로 router 설정이 필요 없었습니다. 하지만 서버에서 IP를 판단해서 그에 따라 다른 페이지가 나와야 하므로 container는 페이지를 나누는 구분으로 사용하였습니다. (원래라면 components 하위에 존재하는 컴포넌트들을 다루는 여러 핸들러나 로직이 있는 컴포넌트를 모아두는 데 주로 사용합니다.)
2. <b>reducers, actions, constants, store</b>
 - Redux 에 종속적인 디렉터리입니다.
 - reducers: 액션 타입에 따라 store에 직접 접근해 store 값을 수정하는 리듀서 작업을 위치시킵니다.
 - actions: dispatch에 넘겨줄 액션 생성자를 정의하거나, dispatch 후 리듀서로 접근 하기 전 값을 컨트롤할 함수를 작성합니다. (이를 위해 redux-thunk 합니다.)
 - contants: 액션 타입을 정해놓은 상숫값들을 지정해 놓은 파일을 넣어둡니다. (actionTypes.js)
 - store: store 설정 파일이 있습니다. (configureStore.js)
3. <b>API</b>
 - server에 직접 API를 호출하는 역할을 합니다.

#### 데이터 플로우

1. 해당 프로젝트의 containers 단순 IP에 의존한 페이지 구분을 목적으로 합니다.
2. store에 저장될 필요 없는 stateless 한 컴포넌트의 경우 API에 직접 접근하여 기능을 수행할 수 있습니다.
3. store에 저장되어야 하는 값은 reducer에 action을 넘겨주는 동작을 반드시 수행해야 합니다.
 - 이때 서버에 데이터가 있어야 하는 경우 `redux-thunk`를 활용해서 비동기 처리를 합니다.

{% image center cycle.png %}

> 참고 : 
[리덕스(Redux) 애플리케이션 설계에 대한 생각](http://huns.me/development/1953)

#### 배포 전 닥친 새로운 문제

맨 처음 프로젝트를 설정할 때 `babel-node server.js`로 서버를 Run 한다는 생각으로 프로젝트를 구성하였습니다. 하지만 babel-node가 ['프로덕션 모드에 사용돼서는 안 된다'](https://babeljs.io/docs/usage/cli/#babel-node)는 사실을 알게 되었습니다. 오픈소스 프로젝트들이 production 모드에서 babel-node를 설정한 부분을 정확히 확인하지 않고 그대로 적용한 저의 실수였습니다.

{% image center areyoukiddingme.gif %}

오픈소스를 조사하면서 서버사이드 렌더링 시 서버를 Run 하는 방법은 크게 두 가지로 나눌 수 있었습니다.
1. babel-node로 서버를 구동한다.
2. 서버를 채로 번들링 한 후 node로 서버를 구동한다.

1번 항목은 프로덕션 배포를 앞둔 저에게는 답이 될 수 없었습니다. 그리고 2번 항목은 저에게는 맞지 않았습니다. 해당 프로젝트는 API Server가 따로 없이 express 서버에 API가 같이 존재했으며, 해당 백엔드 작업은 제가 작업하지 않았거니와 백엔드를 잘 모르는 저에게 그 코드를 번들링해버리는 건 무책임한 행동이라고 생각이 들었습니다.

__그래서 저는 생각 끝에 다음과 같은 방식으로 변경을 시도했습니다.__
1. 서버사이드 렌더링시 템플릿 리터럴 형태로 내려주는 파일을 별도의 .html로 추출합니다.
   이때 기존에 JS들이 들어가는 위치에 특정 형태의 string으로 추가해 놓습니다 (ex: <% data %>
2. renderToString 및 store 설정, styled-components 등 페이지 설정 부분을 별도의 .js 추출합니다.
3. wepack.config.prod.js에 또 하나의 번들 task를 추가해 위에서 2번에서 작업한 .js파일을 node.js에서 읽을 수 있는 `commonjs`형태로 번들링 되도록 설정합니다.
4. 서버에서 페이지를 내려주는 종단에서 .html을 fs로 읽어들이고, 3번에서 번들링된 모듈을 호출해 각각의 값을 읽습니다.
5. 읽은 .html을 toString().replace() 통해 치환해줍니다.

결과적으로 제가 원하는 대로 server는 건드리지 않고 렌더링 되는 부분만 번들링해서 사용할 수 있게 되었습니다.

> 참고 : 
[React 애플리케이션의 서버 렌더링](http://webframeworks.kr/tutorials/react/server-side-rendering/)

#### 마무리

해당 글에는 언급되지 않았지만 이를 제외하고도 많은 문제가 있었고 해결하지 못한 문제들도 많았습니다. 솔직히 말하면 생각했던 방향대로 작업 되지 않아 굴복하고 동작하게끔만 설정해놓은 부분도 더러 있었습니다. 하지만 목표대로 서비스가 되는 프로젝트를 정상적으로 리엑트로 변경하는 작업은 성공이라고 생각됩니다. 시간이 될 때마다 작업한 프로젝트를 손보면서, 새로운 프로젝트에는 또 다른 방식 혹은 더 나은 방식으로 구성해서 작업 해봐야겠습니다. 글이 생각보다 길어져 작업 후 느낀점 같은 부분은 제 기술 블로그가 아닌 개인 블로그에 따로 정리를 해야 할 것 같습니다. 읽어주셔 감사합니다.