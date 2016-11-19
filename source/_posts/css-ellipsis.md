---
title: CSS 말줄임표 사용시 기억할 점
categories: CSS
tags: CSS
keywords:
  - css
date: 2016-11-15 18:49:54
---

### CSS 말 줄임표를 사용하려면 ?

CSS를 사용시 단락에 '말 줄임표'를 사용할 경우가 있는데,
이때 'text-overflow: ellipsis' 속성을 사용하여 적용 할 수 있습니다.

<!-- more -->

사용하려면 다른 속성들이 같이 사용되어야 하는데 거의 공식과 같으니 알아두면 좋습니다.

``` CSS
.ellipsis {
    width: 150px;
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
}
```

### CSS 말줄임표와 사용 가능한 마크업

위의 방식처럼 잘 사용하다가 갑자기 적용이 안되면서 삽질을 한차례 했습니다.
해당 속성을 쓰는데 있어 기초가 되는 사실을 잊었기 때문입니다.

위의 코드를 보면 'ellipsis'를 쓰기 위해서는 'width'를 지정 해주었습니다.
'width'를 지정해야 한다는 의미는 'width'가 {% hl_text danger %} 지정 가능{% endhl_text %}해야 한다는 부분입니다.

평소 저는 'ellipsis'를 ``<p>``에 주로 사용했습니다.
그러나 이번에는 최근에 ``<b>``와 ``<strong>``의 차이를 알게되면서 "마크업 사용시 목적에 맞게 사용하자" 라는 생각을 갖고, ``<b>``에 해당 css를 적용했습니다.
 
하지만, ``<b>``는 기본적으로 ``<span>``과 같은 'display: inline'속성을 가지고 있기 때문에 적용이 불가능했고,
``<b>``의 'display: inline-block'으로 바꾸고 나서야 정상 작동하는걸 확인할 수 있었습니다.

간단한 CSS라도 충분히 이해하고 사용해야 할것 같습니다. 