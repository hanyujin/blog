---
title: Underscore의 _.each 
categories: Javascript
tags: Javascript
keywords:
  - Javascript,
  - library
date: 2017-04-24 09:18:00
---

### Underscore 의 _.each

v1부터 현재까지 코드가 어떻게 변해왔는지 확인해봅니다.

<!-- more -->

[~v0.1.0]
```javascript
window._ = {
  each : function(obj, iterator, context) {
    var index = 0;
    try {
      if (obj.forEach) {
        obj.forEach(iterator, context);
      } else if (obj.length) {
        for (var i=0; i<obj.length; i++) iterator.call(context, obj[i], i);
      } else if (obj.each) {
        obj.each(function(value) { iterator.call(context, value, index++); });
      } else {
        var i = 0;
        for (var key in obj) {
          var value = obj[key], pair = [key, value];
          pair.key = key;
          pair.value = value;
          iterator.call(context, pair, i++);
        }
      }
    } catch(e) {
      if (e != '__break__') throw e;
    }
    return obj;
  }
}
```
1. window객체에 _.each function을 직접 할당해서 사용하고 있습니다.
2. 데이터가 배열, 기본 배열이 아닌 리스트 (ex. 문자열, jQuery), 객체인 경우 loop가 적용되고 있습니다.
   (obj.each는 뭐지?)
-----------------------------------------------------------------------------------
[~v0.3.1]
```javascript
_.each = function(obj, iterator, context) {
  var index = 0;
  try {
    if (obj.forEach) {
      obj.forEach(iterator, context);
    } else if (obj.length) {
      for (var i=0, l = obj.length; i<l; i++) iterator.call(context, obj[i], i, obj);
    } else if (obj.each) {
      obj.each(function(value) { iterator.call(context, value, index++, obj); });
    } else {
      for (var key in obj) if (Object.prototype.hasOwnProperty.call(obj, key)) {
        iterator.call(context, obj[key], key, obj);
      }
    }
  } catch(e) {
    if (e != '__break__') throw e;
  }
  return obj;
};
```
1. v2 버전에서 window객체가 없을 때도 사용되도록 전체 구조가 변경되었습니다.
2. 들어오는 데이터가 객체일 경우 프로퍼티 값이 있는지 확인하는 코드가 추가 되었습니다.
3. iterator.call의 argument값이 달라졌습니다. 하나의 obj가 아니라 각각의 값을 넣습니다.

-----------------------------------------------------------------------------------
[~v0.4]
```javascript
var breaker = typeof StopIteration !== 'undefined' ? StopIteration : '__break__';

_.each = function(obj, iterator, context) {
  var index = 0;
  try {
    if (obj.forEach) {
      obj.forEach(iterator, context);
    } else if (obj.length) {
      for (var i=0, l=obj.length; i<l; i++) iterator.call(context, obj[i], i, obj);
    } else {
      var keys = _.keys(obj)
        , l = keys.length;
      for (var i=0; i<l; i++) iterator.call(context, obj[keys[i]], keys[i], obj);
    }
  } catch(e) {
    if (e != breaker) throw e;
  }
  return obj;
};
```

v0.4.1
1. obj.each를 검사하는 코드가 제거되었습니다.

v0.4.3
1. 에러 체크 방식이 변경되었습니다.
   StopIteration를 체크하는 코드가 있었는데 현재 StopIteration는 Deprecated 되었습니다.

v0.4.7
1. obj Loop시 _.keys function이 추가되었습니다.
2. for in -> for 문으로 변경되었습니다. 
-----------------------------------------------------------------------------------
[~v0.5]
```javascript
_.each = function(obj, iterator, context) {
  var index = 0;
  try {
    if (obj.forEach) {
      obj.forEach(iterator, context);
    } else if (_.isNumber(obj.length)) {
      for (var i=0, l=obj.length; i<l; i++) iterator.call(context, obj[i], i, obj);
    } else {
      var keys = _.keys(obj), l = keys.length;
      for (var i=0; i<l; i++) iterator.call(context, obj[keys[i]], keys[i], obj);
    }
  } catch(e) {
    if (e != breaker) throw e;
  }
  return obj;
};
```
v0.5.1
1. obj.length -> _.isArray(obj) || _.isArguments(obj)로 변경되었습니다.

v0.5.8
1. 다시 _.isNUmber(obj.length)로 변경되었습니다.

-----------------------------------------------------------------------------------
[~v0.6.0]
```javascript
var each = _.forEach = function(obj, iterator, context) {
  var index = 0;
  try {
    if (nativeForEach && obj.forEach === nativeForEach) {
      obj.forEach(iterator, context);
    } else if (_.isNumber(obj.length)) {
      for (var i = 0, l = obj.length; i < l; i++) iterator.call(context, obj[i], i, obj);
    } else {
      for (var key in obj) {
        if (hasOwnProperty.call(obj, key)) iterator.call(context, obj[key], key, obj);
      }
    }
  } catch(e) {
    if (e != breaker) throw e;
  }
  return obj;
};
```
1. each가 foreach라는 이름도 갖게 되었습니다.
2. 기본 배열인지 확인하는 비교문이 수정되었습니다.(nativeForEach = ArrayProto.forEach)
3. 들어오는 obj가 객체인 경우 loop 가 다시 for에서 for in으로 변경되었습니다.
   내부 loop에서 hasOwnProperty를 사용해서 property확인을 합니다.

-----------------------------------------------------------------------------------
[~v1.1.4]
```javascript
var each = _.each = _.forEach = function(obj, iterator, context) {
  if (obj == null) return;
  if (nativeForEach && obj.forEach === nativeForEach) {
    obj.forEach(iterator, context);
  } else if (_.isNumber(obj.length)) {
    for (var i = 0, l = obj.length; i < l; i++) {
      if (iterator.call(context, obj[i], i, obj) === breaker) return;
    }
  } else {
    for (var key in obj) {
      if (hasOwnProperty.call(obj, key)) {
        if (iterator.call(context, obj[key], key, obj) === breaker) return;
      }
    }
  }
};
```
1. try catch문을 제거되었습니다.
2. null check를 추가되습니다.

-----------------------------------------------------------------------------------
[~v1.1.7]
```javascript
var each = _.each = _.forEach = function(obj, iterator, context) {
  if (obj == null) return;
  if (nativeForEach && obj.forEach === nativeForEach) {
    obj.forEach(iterator, context);
  } else if (obj.length === +obj.length) {
    for (var i = 0, l = obj.length; i < l; i++) {
      if (i in obj && iterator.call(context, obj[i], i, obj) === breaker) return;
    }
  } else {
    for (var key in obj) {
      if (hasOwnProperty.call(obj, key)) {
        if (iterator.call(context, obj[key], key, obj) === breaker) return;
      }
    }
  }
};
```
1. _.isNumber(obj.length)가 obj.length === +obj.length으로 변경되었습니다.

-----------------------------------------------------------------------------------
[~v1.3.1]
```javascript
var each = _.each = _.forEach = function(obj, iterator, context) {
  if (obj == null) return;
  if (nativeForEach && obj.forEach === nativeForEach) {
    obj.forEach(iterator, context);
  } else if (obj.length === +obj.length) {
    for (var i = 0, l = obj.length; i < l; i++) {
      if (i in obj && iterator.call(context, obj[i], i, obj) === breaker) return;
    }
  } else {
    for (var key in obj) {
      if (_.has(obj, key)) {
        if (iterator.call(context, obj[key], key, obj) === breaker) return;
      }
    }
  }
};
```
1. hasOwnProperty.call(obj, key)이 _.has()로 수정되었습니다.

-----------------------------------------------------------------------------------

[~v1.4.2]
```javascript
var each = _.each = _.forEach = function(obj, iterator, context) {
  if (obj == null) return;
  if (nativeForEach && obj.forEach === nativeForEach) {
    obj.forEach(iterator, context);
  } else if (obj.length === +obj.length) {
    for (var i = 0, l = obj.length; i < l; i++) {
      if (iterator.call(context, obj[i], i, obj) === breaker) return;
    }
  } else {
    for (var key in obj) {
      if (_.has(obj, key)) {
        if (iterator.call(context, obj[key], key, obj) === breaker) return;
      }
    }
  }
};
```
1. obj.length === +obj.length에서 "i in obj"가 제거되습니다.

-----------------------------------------------------------------------------------
[~v1.5.2]
```javascript
var each = _.each = _.forEach = function(obj, iterator, context) {
  if (obj == null) return;
  if (nativeForEach && obj.forEach === nativeForEach) {
    obj.forEach(iterator, context);
  } else if (obj.length === +obj.length) {
    for (var i = 0, length = obj.length; i < length; i++) {
      if (iterator.call(context, obj[i], i, obj) === breaker) return;
    }
  } else {
    var keys = _.keys(obj);
    for (var i = 0, length = keys.length; i < length; i++) {
      if (iterator.call(context, obj[keys[i]], keys[i], obj) === breaker) return;
    }
  }
};
```
1. obj loop방식도 또 변경되었고, l의 네이밍이 length로 변경되었습니다.

-----------------------------------------------------------------------------------
[~v1.7.0]
```javasccript
_.each = _.forEach = function(obj, iteratee, context) {
  if (obj == null) return obj;
  iteratee = createCallback(iteratee, context);
  var i, length = obj.length;
  if (length === +length) {
    for (i = 0; i < length; i++) {
      iteratee(obj[i], i, obj);
    }
  } else {
    var keys = _.keys(obj);
    for (i = 0, length = keys.length; i < length; i++) {
      iteratee(obj[keys[i]], keys[i], obj);
    }
  }
  return obj;
};
```
1. interator.call 함수가 전역으로 나누어졌습니다.

-----------------------------------------------------------------------------------
[~v1.8.0]
```javascript
_.each = _.forEach = function(obj, iteratee, context) {
  iteratee = optimizeCb(iteratee, context);
  var i, length;
  if (isArrayLike(obj)) {
    for (i = 0, length = obj.length; i < length; i++) {
      iteratee(obj[i], i, obj);
    }
  } else {
    var keys = _.keys(obj);
    for (i = 0, length = keys.length; i < length; i++) {
      iteratee(obj[keys[i]], keys[i], obj);
    }
  }
  return obj;
};
```
1. createCallback -> optimizeCb 으로 function이름이 변경되었습니다. 
2. null 체크가 삭제되었습니다.

-----------------------------------------------------------------------------------
> 1. v0.3.1에서 "(Object.prototype.hasOwnProperty.call(obj, key)"를 for문과 같이 쓰이는 구조가 처음보는 익혀둬야 할것 같다.
2. 점진적으로 여러 예외경우를 추가하면서 try catch 을 제거하는 방식도 괜찮은 방식이인것 같다.
3. for문과 for in 문에 반복적으로 교체되고 있다. 장단점을 확실히 아는게 좋을것 같다.
4. 확실히 최종보전부터 보면 이해가 안되지만 처음버전부터 보면 히스토리를 알게되서 이해가 더 알기 좋다.