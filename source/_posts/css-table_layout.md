---
title: 테이블 패턴을 활용한 레이아웃
categories: css
tags: css
keywords:
  - css
date: 2016-12-08 08:57:02
---

### 어떤 방법으로 레이아웃을 만드시나요? 

레이아웃을 고려할때 여러 방법을 생각을 하게 됩니다.
많은 방법중에 테이블패턴을 활용한 레이아웃은 생각보다 편합니다.

<!-- more -->

이런저런 레이아웃을 만들면서 새로운 방법들을 점점 배워가는것 같습니다.

처음 웹을 배울때는 ``<div>``로 모든 컨텐츠를 만들때도 있었고, 
[Bootstrap](http://getbootstrap.com/)을 알고부터는 class="col-xs-*"로 도배를 한 적도 있었습니다.
(물론 반응형을 구현할때는 요긴하고 사용합니다.)

당시에 저는 테이블 패턴을 이용한 레이아웃은 옛날 옛적의 방식이라고 생각했었습니다.
그러나 ``<table>`` 태그를 사용하는것과 테이블 패턴을 사용하는것은 차이가 있다는 것을 알게되었습니다.

처음 테이블 패턴을 접한건 {% hl_text primary %} "수직 중앙정렬" {% endhl_text %}이 필요해서 였습니다.
단순 중앙정렬은 text-align:"center"나 "margin: 0 auto"을 활용해서 쉽게 구현이 가능합니다.
 
하지만 "수직 중앙정렬"은 뭔가 입맞에 딱 맞는게 없었습니다.
늘 margin이나 padding 그것도 안되면 absolute를 사용해 어떻게 사용해서 하는 방식이 너무 구렸(?)습니다.

아래는 간단히 그 방법을 설명하는 좋은 링크가 있어서 참조 합니다.

> [최고의 CSS 중앙정렬 기법](https://webdesign.tutsplus.com/ko/tutorials/the-holy-grail-of-css-centering--cms-22114)

이러한 방식을 레이아웃에도 적용할수 있을 뿐만 아니라, 상위정렬이나 하위정렬을 만들어 놓으면 편리하게 사용할 수 있습니다.

위의 링크처럼 .inner를 지정해서 사용해도 좋지만 마크업으로 설계를 잘한다면 굳이 하위 클래스를 만들지 않고 아래와 같이 사용할 수도 있습니다.

``` CSS
.table-layout {
  display: table;
  width: 100%;
}

.table-layout-top > * {
  display: table-cell;
  vertical-align: top;
}

.table-layout-middle > * {
   display: table-cell;
   vertical-align: middle;
}

.table-layout-bottom > * {
  display: table-cell;
  vertical-align: bottom;
}
```
물론 이 방법도 단점이 있습니다.
레이아웃사이에 gutter가 당연히 존재할텐데 이때 별도의 마크업을 추가해야 하는 점이 그렇습니다.

다만 이런 방식으로 레이아웃을 구현할 수 있다는 점을 알아두면 좋을것 같습니다.