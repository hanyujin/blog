---
title: CSS 말줄임표
categories: CSS
---

CSS를 사용할때 어떠한 단락에 말 줄임표를 사용애야할 경우가 있다.

이때 'text-overflow: ellipsis' 속성을 사용하면 가능하다.

하지만 저것만으로는 안된다.

사용하려면 다른 값들이 같이 사용되어야 하는데 거의 공식과 같으니 알아두자.

.ellipsis {
  width: 150px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}



