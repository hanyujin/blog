---
title: 더 멋져진 리엑트 프로젝트 
categories: 
  - js
  - react
tags: 
  - react
  - contextAPI
  - react-router
  - project
  - webpack
  - javascript
  - typescript
keywords:
  - react
  - contextAPI
  - react-router
  - project
  - webpack
  - javascript
  - typescript
thumbnailImage: thumbnailImage.jpg
thumbnailImagePosition: bottom
metaAlignment: center
coverImage : coverImage.jpg 
date: 2018-07-19 09:07:12
---

올 초에 작업했던 리엑트 프로젝트를 조금 더 개선하기 위해 수행했던 여러 작업에 대해 간단히 정리해보았습니다.

<!-- more -->

# 리엑트 프로젝트를 조금 더 멋지게 만들자!

기존 프로젝트에서 아쉬웠던 부분과 새롭게 추가해보고 싶은 것들을 정리했습니다.</br></br>

* 빌드 프로세스 개선
* Router 적용
* react v15 -> v16 (ContextAPI 활용)
* typescript 적용

너무 많은 목표를 두는 게 아닌가 생각은 했지만, 목표들이 어느 정도 포함관계가 있어서 지금 해두는 게 이후에 적용하는 것보다 덜 복잡할 것 같다는 생각을 해서 이번 기회에 추진하기로 했습니다.

## 빌드 프로세스 개선 

올 초에 프로젝트를 진행하면서 리엑트로 프로젝트를 변경하는 것을 중점으로 작업하다 보니 그 이외에 부분을 신경 쓰지 못했습니다. 그중에 한 부분은 바로 빌드 프로세스입니다.

#### 리비전 추가

기존에는 번들 파일이 `bundle.js`와 같은 형태로 생성되었습니다. 이러한 형태는 파일이 수정되어도 브라우저는 알지 못할 수 있습니다. 브라우저는 기본적으로 리소스를 브라우저캐시에 두고 파일 이름의 차이로 인해 변경사항을 파악합니다. 그래서 아래처럼 작성하여 브라우저 캐시를 갱신합니다.

```javascript
{
  output: [name].[chunkhash].js,
  plugins: [
    new ExtractTextPlugin('[name].[contenthash].css')
  ]
}
```

하지만 이렇게 번들링된 파일은 파일 변경 시 매번 hash 값이 달라지니 파일 이름도 같이 달라져서 HTML에 sciprt를 추가할 수가 없습니다. 번들링 된 파일 이름을 알아내기 위해 webpack에 [webpack-manifest-plugin](https://www.npmjs.com/package/webpack-manifest-plugin) 을 적용했습니다.

```javascript
new ManifestPlugin({
  fileName: `${path}/manifest.json`
}),   

```

빌드를 통해 나온 `manifest.json` 파일을 통해 번들링된 파일 이름을 얻을 수 있고 리소스를 HTML에 추가할 수 있었습니다.

```json
{
  "bundle.js": "bundle.ac8a6e5e050f1befb5e4.js",
  "bundle.css": "bundle.b0cd9b716fef2bc73a57dcbd83cfefb0.css",
}
```

#### 공통모듈 분리

웹팩의 설정파일은 `webpack.config.dev.js`, `webpack.config.prod.js` 와 같이 나누어져 있었습니다. 여러 설정이 추가되면서 설정파일이 커지면서 공통된 부분을 별도로 나누는 것이 좋다고 생각했습니다. 

공통 부분은 `webpack.common.js`를 생성해서 공통모듈을 생성하였습니다. 해당 파일로 분리된 설정파일을 각각 하나의 `dev`와 `prod` 병합할 때는 [webpack-merge](https://github.com/survivejs/webpack-merge)을 사용했습니다.

```javascript
const merge = require('webpack-merge')
const common = require('./webpack.common.js')
const ssr = require('SSR_render.jg')
module.exports = merge(common, { ... })
```
---

## Router 적용

기존에는 단일페이지라서 별도로 router를 세팅할 필요가 없지만, 새롭게 추가될 페이지를 위해 [react-route](https://reacttraining.com/react-router/)를 적용하기로 하였습니다. 

해당 프로젝트는 앞서 언급했던 것처럼 SSR이 지원되어야 했습니다. 그러므로 리엑트에서 적용했던 `ReactDOM.render` 과 `ReactDOMServer.renderToString`처럼 SSR과 CSR을 위한 다른 Wrapper 메서드가 필요했는데 역시나 App을 Wrapping 하여 사용하는 `react-router` 도 그러한 메서드가 존재했습니다. CSR에는 `BrowserRouter` 를 사용하고, SSR에는 `StaticRouter`를 사용하였습니다.

App 내부에서는 `<Switch/>`로 감싼 `<Route/>` 들을 설정하였습니다.

```javascript
// SSR
import { StaticRouter } from 'react-router-dom'

// CSR
import { BrowserRouter } from 'react-router-dom'
```

---

## React v16 적용

리엑트가 v16을 배포한 지 약 10개월 정도 지났는데, 이번 기회에 v16 중 필요한 기능들을 적용해 보았습니다.

#### Context API 도입

Context API는 주로 전역 데이터가 필요할 때 사용합니다. 이러한 기능은 제가 redux를 도입한 목적과 같아 충분히 교체 가능했습니다. 

{% image center contextAPI.png 출처 : https://blog.bitsrc.io/react-context-api-a-replacement-for-redux-6e20790492b3 %}

redux를 이해하고 적용하는데 많은 시간이 든다고 생각됩니다. 저도 필요한 부분은 적용하며 사용했지만, 모든 부분을 자세히 알고 있다고는 생각할 수 없습니다. 전역 데이터를 위해 사용했지만, redux의 사용이 좀 과하다는 생각이 남아있긴 했습니다.

Context API를 적용하기 위해 많은 정보를 찾았고, 그중 많은 부분을 `velopert`의 블로그에서 참고하여 작업을 진행하였습니다. 이유는 redux를 사용할 때도 store 내부에 크게 2개로 나누어 구분해서 사용했기 때문입니다. 그래서 Context API를 적용하려는 상황에서도 다중 context가 필요하다고 생각이 들었습니다. 이와 관련한 내용이 친절하게 설명이 되어 있었습니다.

> [리액트 16.3 에 소개된 새로워진 Context API 파헤치기](https://velopert.com/3606)

하지만 모든 부분을 이전과 같은 컴포넌트 구조로 교체하지 못했습니다. 하나의 컴포넌트에서 여러 context가 필요한 경우였습니다. redux에는 store가 하나였기에 이런 부분에는 문제가 없었습니다. Context API를 활용하는 컴포넌트를 Hoc 형태로 만들었기에 하나의 컴포넌트에 2개 이상의 context를 주입받아야 할 때 컴포넌트를 겹겹이 감싸는 형태는 좋아 보이지 않았습니다.

그래서 이러한 경우가 발생한 컴포넌트들은 다시 context 기준으로 컴포넌트를 분리하였습니다. context 기준으로 새롭게 컴포넌트를 생성하다 보니 이전보다 더 명확한 목표를 가진 컴포넌트가 자연스럽게 생성된 결과를 볼 수 있었습니다.

작업하면서 "이 데이터가 이 context에 들어가는 게 맞나?"라는 고민을 참 많이 했던 것 같습니다. 구현 전에 데이터 모델링이 중요하다는 사실을 다시 한 번 느꼈습니다. 하지만 설계단계가 아닌 개발 중간마다 이러한 경우를 맞이하는 것이 아직은 더 많은 공부가 필요한 것을 상기시켜 주는 것 같습니다.

#### redux-from에서 formik로 변경

redux를 제거하게 되면서, redux관련 모듈인 react-redux, redux-thunk, redux-form 들을 제거할 수 있었습니다. 그 중 redux-form은 다른 모듈로 대체되어야 했는데 그 역할은 `formik`였습니다. 사용이 간편하고 필요한 기능들을 충족했기 때문이었습니다.

{% image center form.png %}

사실 submit action이나 validation은 작업의 편의성을 위해 라이브러리를 주로 사용합니다. 하지만 단순히 값을 체크하는걸 넘어서 자동으로 다른 폼의 값을 채우거나, 비동기로 값을 체크하거나, 특정 액션에 다른 view에 영향을 미치는 등 예외의 경우를 조작하는 것은 간단하지만은 않았습니다.

이럴 거면 그냥 라이브러리 사용하지 않고 처음부터 만들 걸 그랬나 생각도 들었지만, 기본기능들을 그냥 새로 만드는 건 비효율적일 것 같지 않아 끈기있게 붙들고 적용하였습니다.
</br>

{% image center movie_image.jpg %}

redux의 의존하지 않는다는 점에서 충분히 작업의미가 있었습니다. 처음엔 `redux-form`과 `formik`가 많이 다르다고 생각했지만, 결과적으로 바뀐 코드를 보면 역시 프론트엔드의 컴포넌트 형태의 개발은 크게 다르지 않다는 점을 알 수 있었습니다. 

> [redux-form](https://redux-form.com/7.4.2/)
>
> [formik](https://github.com/jaredpalmer/formik)

#### ReactDOM.hydrate()

v16을 적용하면 `ReactDOM.render()`를 `ReactDOM.hydrate()` 로 변경하라는 경고를 볼 수 있습니다. 이건 v16에서 나온 새로운 렌더링 메서드이며 `ReactDOM.render()` 은 v17에서는 사용되지 않을 예정이라고 합니다. 

`ReactDOM.hydrate()` SSR을 지원하는 앱에서 서버 렌더링 된 마크업이 있는 노드에서 호출하면 리엑트는 이벤트 핸들러만 연결해서 성능을 시켜준다고 합니다.

> [What's the difference between hydrate() and render() in React 16?](https://stackoverflow.com/questions/46516395/whats-the-difference-between-hydrate-and-render-in-react-16)

---

## 타입스크립트를 적용하는 이유

복잡한 데이터 구조는 버그를 발생시키는데 한몫합니다. 여러 프로젝트를 하면서 가장 많은 버그를 생성한 곳 역시 복잡한 데이터를 주고받는 부분이었습니다. 이러한 부분에 도움을 받을 만한 것이 바로 `타입스크립트(typescript)`라고 생각을 했고 적용해보기로 했습니다. 규모가 작은 프로젝트에는 오버엔지니어링이라고 생각될 수 있지만, 이번 기회에 타입스크립트를 적용해보면서 느낀 점도 있어서 결과적으로 "적용해보길 잘했다."라는 생각을 했습니다.

#### 타입스크립트 로더 설정

공식페이지 [React & Webpack](https://www.typescriptlang.org/docs/handbook/react-&-webpack.html)에 언급된 [awesome-typescript-loader](https://github.com/s-panferov/awesome-typescript-loader)를 사용하기로 했습니다. 해당 loader를 설치한 후에 'webpack.config.js' 설정했습니다. 

```javascript
{
	test: /\.tsx?$/,
	loader: 'awesome-typescript-loader',
	exclude: /node_modules/
},
```
타입스크립트를 설정한 파일의 확장자가 `.tsx` 인 것을 확인했고, 그 파일에만 타입스크립트가 적용되기 때문에 `.jsx` 파일을 하나씩 `.tsx` 로 변경하면서 기능을 확인한 후 넘어가는 식으로 변경 작업을 진행했습니다.

#### 리엑트 with 타입스크립트

타입스크립트에서는 타입을 선언하는 방식이 많은데 저는 주로 가장 간단한 `interface` 키워드를 사용하여 타입을 생성했습니다. 우선 선언한 타입을 기준으로 리엑트 컴포넌트를 생성하는 방법입니다.</br></br>

* 함수기반 컴포넌트(React Stateless Functional Component with TypeScript)

```typescript
import * as React from "react"
interface Props {
	name: string
}
const customComponent: React.SFC<Props> = (props) => {
	return <h1>Hello, {props.name}</h1>;
}
```
* 클래스기반 컴포넌트(React StateFul Class Component with TypeScript)

```typescript
import * as React from "react"
interface Props {
    name: string
}
interface State {
    state: boolean
}
class customCompnent extends React.Component<Props, State> {
    ...
}
```

공통된 구조의 타입은 interface를 extends로 상속하여 사용했습니다. 사실 컴포넌트 간의 구조를 생각하면서 하기도 쉽지 않은데 interface 간에 구조도 생각하려니 머리가 아팠습니다. 그러나 결과만 놓고 보면 기본적은 Props의 타입 검사는 만족했으며, 컴포넌트에서 어떠한 구조의 Props를 사용하고 있다는 것을 더욱 가시적으로 확인할 수 있다는 점이 정말 좋았습니다.

하지만 좋은 점만 있는 게 아니었습니다. 1의 일에 1.5배를 추가로 시간을 투자해야 했습니다. 게다가 강하게 타입체크를 하다 보니 쉬이 넘길 수 있는 부분에 추가로 코드가 들어가야 하는 걸 보면 답답함을 느끼곤 했습니다. 하지만 이런 부분은 기존 작업방식에서 빈틈이었던 부분이니 보완해야 할 부분이 맞아 보였습니다.

---

## 마무리

여러 목표를 갖고 개선 프로젝트를 했는데도, 새로운걸. 적용하고 알게 되면서 또 다른 새로운 게 보이고 더 나은 방법이 보이는 건 어쩔 수 없는 것 같습니다. 하지만 이러한 경험 덕분에 다른 것들을 적용해볼 수 있는 환경 및 정보들을 얻었으니 좋은 시도였다고 생각이 듭니다. 다음 개선에는 또 다른 걸 시도해 보고 싶은데 그중에서 하나는 렌더링 성능 관련인데 충분히 더 알아보고 도전해봐야겠습니다. 읽어 주셔서 감사합니다.
