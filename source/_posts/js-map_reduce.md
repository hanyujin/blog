---
title: filter, map 그리고 reduce 
categories: Javascript
tags: Javascript
keywords:
  - Javascript
  - higher-order function
date: 2017-02-15 09:46:22
---

### 데이터 구조 재가공 하기

작업을 위해 원데이터를 재가공해야할때가 있습니다.  
원하는 구조를 만들기 위해 filter, map, reduce를 활용할 수 있습니다.

<!-- more -->

## 1. filter  
filter는 말 그대로 어떠한 기준으로 데이터를 제한할 수 있습니다.

아래는 JSON에서 무효한 항목 거르기는 예제입니다.  
0이 아닌, 숫자 id인 모든 요소의 걸러진 json을 만들기 위해 filter()를 사용합니다.

> 예제 출처 - [MDN : Array.prototype.filter()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

```javascript
var arr = [ { id: 15 }, { id: -1 }, { id: 0 }, { id: 3 }, { id: 12.2 }, { }, { id: null }, { id: NaN }, { id: 'undefined' } ];

var invalidEntries = 0;

function filterByID(obj) {
  if ('id' in obj && typeof(obj.id) === 'number' && !isNaN(obj.id)) {
    return true;
  } else {
    invalidEntries++;
    return false;
  }
}

var arrByID = arr.filter(filterByID);

// 결과 : [{ id: 15 }, { id: -1 }, { id: 0 }, { id: 3 }, { id: 12.2 }]
```

위의 예제처럼 filter는 원 데이터를 자신이 원하는 기준을 이용해 재가공 하는데 작업을 수행 할 수 있습니다.

## 2. map

map을 활용할 경우 원데이터를 기준으로 데이터를 다른 구조로 변경할 수 있습니다.

> 예제 출처 - [MDN : Array.prototype.map()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

```javascript
var kvArray = [{key:1, value:10}, {key:2, value:20}, {key:3, value: 30}];
var reformattedArray = kvArray.map(function(obj) { 
   var rObj = {};
   rObj[obj.key] = obj.value;
   return rObj;
});

// 결과 : [{1:10}, {2:20}, {3:30}] 
```

## 3. reduce

reduce 메서드는 누산기 역할을 할 수 있으며, 값을 재가공 할 수도 있습니다.

reduce를 제대로 사용하기위해서는 넣어야할 파라미터들을 잘 알고 있는게 중요합니다.

- previousValue : 이전 마지막 콜백 호출에서 반환된 값 또는 공급된 경우 초기값(initialValue)  
- currentValue : 배열 내 현재 처리되고 있는 요소(element).  
- currentIndex : 배열 내 현재 처리되고 있는 요소의 인덱스.  
- array : reduce에 호출되는 배열.

우선 reduce활용으로 만든 누산기 예제 입니다.

> 예제 출처 - [MDN : Array.prototype.reduce()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

```javascript
[0, 1, 2, 3, 4].reduce(function(previousValue, currentValue, currentIndex, array) {
  return previousValue + currentValue;
});

// 결과 : 10
```

- initialValue : 초기값을 지정했을 경우

```javascript
[0, 1, 2, 3, 4].reduce(function(previousValue, currentValue, currentIndex, array) {
  return previousValue + currentValue;
}, 10);

// 결과 : 20
```

reduce의 좀 더 나은 활용방법은 어떤것이 있을까요?  

filter를 사용할 경우 원하는 데이터를 어떠한 기준으로 정제할 수 있습니다.  
다만, 가진 데이터 구조를 변경하지는 못합니다.

그럼 map은 어떨까요?  
map을 사용할 경우 데이터 구조를 원하는대로 변경할 수 있지만 그 데이터의 사이즈를 재조정 하지 못합니다.  
즉, 조건에 맞게 데이터를 넣지 않을 수 있지만 실질적으로 Return 되는 데이터에서 제거되지는 못합니다.

가장 첫번째 filter의 예제를 활용해 봅시다.

```javascript
var arr = [ { id: 15 }, { id: -1 }, { id: 0 }, { id: 3 }, { id: 12.2 }, { }, { id: null }, { id: NaN }, { id: 'undefined' } ];

// filter를 사용했을 때 number type의 값만 뺐을 경우 아래와 같은 값을 얻을 수 있었습니다.
// [{ id: 15 }, { id: -1 }, { id: 0 }, { id: 3 }, { id: 12.2 }]
```

map을 사용해 위의 arr에 아래의 label값을 추가하겠습니다.

```javascript
var label = ["a","b","c","d","e","f","g","h","i"];

arr.map(function(obj, i) {
  obj.label = label[i];
  return x
});

/* 결과 
{ id: 15, label: 'a'}, { id: -1, label: 'b' }, { id: 0, label: 'c' }, { id: 3, label: 'c' }, { id: 12.2, label: 'd' },
{ name: 'f'}, { id: null, label: 'g' }, { id: NaN, label: 'h' }, { id: 'undefined', label: 'i' }
*/
```

그렇다면 map안에 filter에서 사용했던 조건문을 추가해서 작업을 해보죠.

```javascript
arr.map(function(obj, i) {
  if (typeof(obj.id) === 'number' && !isNaN(obj.id)) {
    obj.label = label[i];
    return obj
  }
});

// 결과
// [Object, Object, Object, Object, Object, undefined, undefined, undefined, undefined]
```
map은 조건문은 값을 제한하여도 원래 사이즈와 동일한 사이즈의 데이터를 리턴해줍니다.

reduce를 사용해서 데이터를 정제하도록 하겠습니다.

```javascript
var result = arr.reduce(function(prev, value, idx) {
  value.label = label[idx];
  if (typeof(value.id) === 'number' && !isNaN(value.id)) {
    prev.push(value);
  }
  return prev;
}, []);

// 결과 : label데이터가 추가가 됨과 동시에 조건문에 맞게 정제된 데이터가 내려옵니다. 
// [{id: 15, label: 'a}, {id: -1, label: 'b'}, ... ]
```

filter와 map의 기능을 reduce에서 동시에 작업할 수 있었습니다.

하지만 이렇다고 해서 모든 경우 reduce를 쓰는건 옳지 않습니다. 
적당한 상황에 옳게 쓰기위해 filter, map, reduce를 쓰는 법을 알아야 하겠습니다.
