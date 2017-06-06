---
title: React에서 컴포넌트 스타일링 하기
categories: React
tags: 
  - React
keywords:
  - React
thumbnailImage: react.png  
date: 2017-06-06 13:30:00
---

### 컴포넌트 스타일 적용하기

리엑트는 컴포넌트들을 작성할때 어떤식으로 스타일을 입힐 수 있을까요?

<!-- more -->

#### 1. Basic CSS

React도 다른 웹구현 방식과 동일하게 Head에서 link 태그를 통해 css를 불러와 적용할 수 있습니다.
물론 style 태그를 통해 직접 스타일을 작성하는 것도 가능하죠.

편리하긴 하지만 이 방식은 Class name의 중복성을 피할수가 없습니다.

게다가 이렇게 사용하면 컴포넌트 중심으로 구현되는 React와는 조금 동떨어진 느낌입니다.

#### 2. CSS Module

글보다는 우선 예제를 보시죠

```javascript
import React from 'react';
import styles from './table.css';

export default class Table extends React.Component {
    render () {
        return <div className={styles.table}>
            <div className={styles.row}>
                <div className={styles.cell}>A0</div>
                <div className={styles.cell}>B0</div>
            </div>
        </div>;
    }
}
```

위의 코드처럼 .css를 import로 불러와서 컴포넌트에서 .으로 접근하여 사용하는 방식입니다.
보다 컴포넌트별로 구분할 수 있고 접근하기도 좀 더 명확해 보입니다.

[CSS Module](https://github.com/gajus/react-css-modules)는 webpack을 이용하여 사용할 수 있습니다.

저도 webpack을 잘하면 떡주무르듯 위의 링크에서 적용하라는 대로 해서 사용할 수 있겠는데,
아직 webpack을 잘 모르는 상태라서 생각보다 적용이 쉽지 않네요.

그래서 create-react-app에서 eject를 해서 설정을 변경해서 적용해보았는데 저도 1번 방식이으로 하다가 컴포넌트 단위로 작업해보니 조금 새로웠습니다.

여기에 더불어 여러 classname을 적용할때 es6 template literals을 사용하지 않고도 편리하게 적용할 수 있는 [classnames](https://github.com/JedWatson/classnames)을 사용하거나 sass나 less 같은 preprocessor를 적용하여 사용할 수 있습니다.
  
#### 3. styled-components 

[styled-components](https://www.styled-components.com/)는 컴포넌트 자체가 스타일을 입고 태어난다고 말할 수 있을것 같습니다.

아래와 같이 사용됩니다.

```jsx 
const Button = styled.button`
  border-radius: 3px;
  padding: 0.25em 1em;
  margin: 0 1em;
  background: transparent;
  color: palevioletred;
  border: 2px solid palevioletred;

  ${props => props.primary && css`
    background: palevioletred;
    color: white;
  `}
`
```

해당 스타일 방식은 컴포넌트 자체에 스타일을 입히기 때문에 props를 받아서 바로 스타일에 적용할 수도 있다는게 장점입니다.






