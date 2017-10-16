---
title: form 전송과 ajax 전송
categories: js
tags: 
  - js
  - form
  - ajax
keywords:
  - js
  - form
  - ajax
date: 2017-10-16 09:39:10
---

### form 전송과 ajax 전송의 차이

요즘 흔히 서버와의 통신시 ajax를 사용하고 있습니다.
하지만 `<form>`을 통해서도 서버와의 통신이 가능한데 어떤 차이점이 있을까요?

<!--more-->

## 비동기

form 방식과 ajax 방식의 대표적인 차이점이라고 하면 역시 ajax는 페이지의 전환 없이 `비동기`로 서버와 통신을 할 수 있게 되었다는 점입니다. 이러한 `비동기`를 통해 전체 페이지가 아닌 일부분만을 업데이트 할 수 있게 해줍니다.


## 이벤트

사용자 인터렉션에 의헤 데이터를 서버로 전송하는경우 반드시 이벤트에 의해 발생되게 됩니다.

이때, ajax를 사용해 통신을 할 경우 이벤트를 처리하는 이벤트 리스너를 별도로 생성해야합니다.

하지만 form을 이용한다면 별도의 코드 없이 HTML에서 만으로 submit 이벤트를 발생시켜서 데이터를 서버에 전송할 수 있습니다.

form에 form submit이 가능한 버튼이 있고, form에 포커스가 있는 상태라면 `enter`를 누르면 form submit이 됩니다.

```html
<!-- 폼 전송이 가능한 버튼 -->

<!-- 범용 전송 버튼 -->
<input type="submit" value="Submit Form">

<!-- 커스텀 전송 버튼 -->
<button type="submit">Submit Form</button>

<!-- 이미지 버튼->
<input type="image" src="graphic.gif">
```

form에는 이처럼 편의 기능이 있어서, 두가지 경우를 합쳐서 사용한는 경우가 많습니다. 

form을 이용해 submit 이벤트를 발생시키고, 이후에 form 전송이 막고 ajax를 통해 `비동기` 로 통신을 하는 방식이 많이 사용 되고 있습니다.


## content-type

content-type은 request로 보내는 데이터가 무엇인지 알려줍니다.

기본적으로 form과 ajax의 content-type은 `application/x-www-form-urlencoded`으로 key=value&key=value 형태로 전송됩니다.

form에서는 기본 전송인 `application/x-www-form-urlencoded`방식과 파일전송 방식인 `multipart/form-data`을 사용하고 있습니다.

하지만 ajax는 기본을 제외하고 다른 라이브러리들은 이를 `application/json`과 같은 다른 방식으로 사용하는 경우가 많으니 인지하고 사용해야 합니다.

변경된 이유는 예전과 다르게 `다계층`의 데이터를 주고 받아야 할 일이 많아졌기 때문입니다.

> 참고 :
* https://gist.github.com/jays1204/703297eb0da1facdc454
* http://fetobe.co.kr/http-protocol/
* http://1ambda.github.io/content-type-vs-accept-http-header/