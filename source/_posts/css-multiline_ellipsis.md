---
title: 여러줄에서 말줄임표 사용하기
categories: css
tags: css
keywords:
  - css
date: 2017-01-30 14:32:00
---

### 멀티라인에서의 말줄임은 단일줄과 다릅니다.

기존 말줄임은 간단히 'text-overflow: ellipsis' 속성을 사용하여 적용 할 수 있습니다.
말줄임 기능은 여러줄에 있을 경우 더욱 필요한 경우가 더 많습니다.
하지만 우리가 아는 ellipsis로는 구현할 수 없습니다.
 
<!-- more -->

웹킷엔진을 사용하는 브라우저에서는 -webkit-line-clamp 프로퍼티를 활용하여 여러줄의 문단도 CSS를 통해 말줄임 처리를 할 수 있습니다.

[지원하는 브라우저 확인하기](http://caniuse.com/#search=line-clamp)

``` CSS
display: -webkit-box; /* 블록을 임의의 배치 순서로 변경할 수 있게 해줌 */
-webkit-line-clamp: 3; /* 라인수 */
-webkit-box-orient: vertical; /* 박스의 흐름의 방향을 지정 */
```

### -webkit-line-clamp 미지원 브라우저 지원
 
-webkit-line-clamp를 지원하지 않는 브라우저에서는 아래와 같이 사용할 수 있습니다.
 
height = line-height * -webkit-line-clamp 으로 설정해주시면 됩니다.

```css
overflow: hidden;
text-overflow: ellipsis;
display: block; /* Fallback for non-webkit */
display: -webkit-box;
-webkit-line-clamp: 3; 
-webkit-box-orient: vertical;
line-height: 20px;
height: 60px; /* Fallback for non-webkit */
```

하지만, 위의 방식으로 하면 텍스트를 원하는 만큼 줄이는건 가능하지만 여전히 '...' 를 사용하는 말줄임이 노출되지는 않습니다

### firefox에서 '...' 표현하기

firefox는 line-clamp을 지원하지 않는 브라우저중 하나입니다.
이때는 css hack을 활용해 '...'를 나타낼 수 있습니다.

```css
@-moz-document url-prefix() {
  .DescriptionExcerpt {
    overflow: hidden;
    position: relative;
  }
  .DescriptionExcerpt:before {
    background: #FFFFFF;
    bottom: 0;
    position: absolute;
    right: 0;
    float: right;
    content: '\2026';
    margin-left: -3rem;
    width: 3rem;
  }
  .DescriptionExcerpt:after {
    content: '';
    background: #FFFFFF;
    position: absolute;
    height: 50px;
    width: 100%;
    z-index: 1;
  }
}
```
@-moz-document url-prefix는 Gecko (Mozilla Firefox) 만 타겟팅하는 데 사용되는 CSS 해킹입니다.
다른 모든 브라우저는 스타일을 무시합니다.

즉, 다른 미지원 브라우저들도 hack을 활용하여 '...'을 나타낼 수 있습니다.
{% hl_text danger %}하지만, 이렇게 되면 각 브라우저마다 다른 코드들을 생성으로 코드가 길어질 수 있습니다.{% endhl_text %}  

<br/><br/>

그렇기때문에 모든 브라우저를 지원하는게 단순히 일정 브라우저에서 지원하는것보다 중요하다면,

<br/>

{% hl_text primary %} line-clamp를 사용하는것 대신에 공통으로 쓸 수 있는 코드를 사용하는것도 좋은방법일거라 생각됩니다.{% endhl_text %}

참고자료 :
1. [What does @-moz-document url-prefix() do?](http://stackoverflow.com/questions/3123063/what-does-moz-document-url-prefix-do)
2. [Multi-line Ellipsis using pure CSS as a work around.](http://revelry.co/multi-line-ellipsis-using-pure-css/)