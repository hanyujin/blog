---
title: 반복문에서 break, continue 사용하기
categories: Javascript
tags: Javascript
keywords:
  - Javascript
date: 2017-01-08 14:35:33
---

### JS에서 break와 continue사용하기

자바스크립트에서 제공하는 반복문으로 break와 continue를 사용해보자.

<!-- more -->

보통 프로그래밍 언어에서 break와 continue을 사용할때 아래의 경우처럼 문을 사용합니다.

```Javascript
for (var i=0; i<5; i++) {
  if (i == 2) continue;
  console.log(i);
}

var i = 0, j = 5;
while (i < 10) {
  if (i == j) break;
  console.log(i);   
}
```

forEach에서는 어떻게 사용할까요?

```Javascript
[0,1,2,3,4].forEach(function(v) {
  if (v == 2) continue; // or break;
  console.log(v)
});
```
{% hl_text danger %}forEach에서는 continue와 break를 사용할 수 없습니다.{% endhl_text %}
대신 .some()을 사용 할 수 있습니다.

.some()은 컬렉션의 요소 중 최소 1개라도 조건을 만족시키는지 검사하는 메서드 입니다.

```Javascript
[0,1,2,3,4].some(function(v) {
  if (v == 2) return false;
  console.log(v)
});
```

return true; 가 break 라면,
return false; 는 continue 입니다.

> 출처 - Outsider님 Blog : [forEach에 break문 대신 some 사용하기](https://blog.outsider.ne.kr/847)

jQuery의 $.each에서는 어떨까요?

```Javascript
$.each([0,1,2,3,4], function(v) {
  if (v == 2) return false;
  console.log(v)
});
```

return true; 가 continue 이며,
return false; 는 break 입니다.

some과 다른이유는 some이 break와 continue를 위해 return true와 false를 하는 용도로 만들어진게 아니기 때문입니다.

간단한 경우지만 막상 쓰려면 기억이 잘 나지 않는경우가 있는데 알고 있으면 좋은것 같습니다.
