---
title: Underscore의 _.map 
categories: Javascript
tags:
  - Javascript
  - underscore.js
keywords:
  - Javascript
  - library
date: 2017-04-29 11:15:00
---

### Underscore 의 _.map

v1부터 현재까지 코드가 어떻게 변해왔는지 확인해봅니다.

<!-- more -->

[~v0.1.0]
```javascript
map : function(obj, iterator, context) {
  if (obj && obj.map) return obj.map(iterator, context);
  var results = [];
  _.each(obj, function(value, index) {
    results.push(iterator.call(context, value, index));
  });
  return results;
}
```
1. 기존 map function이 있는 기본 function을 사용한다.
2. map의 특징 중 하나인 결과값 return을 하는데 _.each의 결과를 results에 저장하는 것을 확인할 수 있다.
-------------------------------------------------
[~v0.5.0]
``` javascript
_.map = function(obj, iterator, context) {
  if (obj && _.isFunction(obj.map)) return obj.map(iterator, context);
  var results = [];
  _.each(obj, function(value, index, list) {
    results.push(iterator.call(context, value, index, list));
  });
  return results;
};
```
1. obj.map -> _.isFunction(obj.map) 로 변경됨

``` javascript
_.isFunction = function(obj) {
  return obj && obj.constructor && obj.call && obj.apply;
};
```
function이 가지고 있는 property를 확인하는 방식으로 만들어져 있습니다.

-------------------------------------------------
[~v0.6.0]
``` javascript
_.map = function(obj, iterator, context) {
  if (nativeMap && obj.map === nativeMap) return obj.map(iterator, context);
  var results = [];
  each(obj, function(value, index, list) {
    results.push(iterator.call(context, value, index, list));
  });
  return results;
};
```
1. 기본 값인지 확인하는 nativeMap 추가되었습니다.
   (nativeMap = Array.prototype.map)
2. _.each에서 _.each가 each로 변경되는데 부분이 있는데 map을 통해 그시기가 다른 내부 함수에서 each를 사용하기 위해 내부적으로 사용되는 경우를 고려해서 변경된것으로 보입니다.
-------------------------------------------------

[~v1.1.1]
``` javascript
_.map = function(obj, iterator, context) {
  if (nativeMap && obj.map === nativeMap) return obj.map(iterator, context);
  var results = [];
  each(obj, function(value, index, list) {
    results[results.length] = iterator.call(context, value, index, list);
  });
  return results;
};
```
1. push 대신에 key에 접근해 직접 값을 할당하는 하는 방식으로 변경됨

-------------------------------------------------

[~v1.1.4]
``` javascript
_.map = function(obj, iterator, context) {
  var results = [];
  if (obj == null) return results;
  if (nativeMap && obj.map === nativeMap) return obj.map(iterator, context);
  each(obj, function(value, index, list) {
    results[results.length] = iterator.call(context, value, index, list);
  });
  return results;
};
```
1. null 값 체크 추가

-------------------------------------------------
[~v1.2.4]
``` javascript
_.map = function(obj, iterator, context) {
  var results = [];
  if (obj == null) return results;
  if (nativeMap && obj.map === nativeMap) return obj.map(iterator, context);
  each(obj, function(value, index, list) {
    results[results.length] = iterator.call(context, value, index, list);
  });
  if (obj.length === +obj.length) results.length = obj.length;
  return results;
};
```
1. result의 length를 별도로 넣는 코드가 추가 됬습니다.
   key로 접근해 데이터를 넣을때 length가 정확히 측정되지 않는 부분이 있는것 같습니다
   (추가 확인 필요)
-------------------------------------------------
[~v1.4.2]
``` javascript
_.map = _.collect = function(obj, iterator, context) {
  var results = [];
  if (obj == null) return results;
  if (nativeMap && obj.map === nativeMap) return obj.map(iterator, context);
  each(obj, function(value, index, list) {
    results[results.length] = iterator.call(context, value, index, list);
  });
  return results;
};
```
1. _.collect 라는 다른 네이밍이 추가되었습니다.

-------------------------------------------------
[~v1.8.0]
```javascript
_.map = _.collect = function(obj, iteratee, context) {
  iteratee = cb(iteratee, context);
  var keys = !isArrayLike(obj) && _.keys(obj),
      length = (keys || obj).length,
      results = Array(length);
  for (var index = 0; index < length; index++) {
    var currentKey = keys ? keys[index] : index;
    results[index] = iteratee(obj[currentKey], currentKey, obj);
  }
  return results;
};
```
1. v1.8.0에 공통함수인 function cb에 의해서 _.each와 동일하게 iteratee를 사용하고 있습니다.
2. results로 return되는 결과를 for를 돌면서 생성되는 데이터에의 의존되는 방식이 아닌, 미리 값체크를 완료후에 데이터를 삽입하고 있습니다.