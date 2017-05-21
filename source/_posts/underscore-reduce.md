---
title: Underscore의 _.reduce 
categories: Javascript
tags: 
  - Javascript
  - underscore.js
keywords:
  - Javascript
  - library
date: 2017-05-10 09:52:00
---

### Underscore 의 _.reduce

v1부터 현재까지 코드가 어떻게 변해왔는지 확인해봅니다.

<!-- more -->

[~v0.1.0]
```javascript
inject : function(obj, memo, iterator, context) {
  _.each(obj, function(value, index) {
    memo = iterator.call(context, memo, value, index);
  });
  return memo;
}
```
1. 처음에는 reduce란 이름대신 inject란 이름의 함수로 존재합니다.
2. 실제 결과를 받는 return value 이외에 memo라는 inject내부에서 누적값을 쌓는데 사용되는 변수를 넣어주어야 하는 구조로 되어있습니다.
3. map과 다르게 내부 memo에 새로운 값을 할당해줍니다.

-----------------------------------------
[~v0.2.0]
```javascript
_.reduce = function(obj, memo, iterator, context) {
  _.each(obj, function(value, index) {
    memo = iterator.call(context, memo, value, index);
  });
  return memo;
};
```
1. reduce로 이름이 변경되었습니다.

-----------------------------------------
[~v0.3.3]
```javascript
_.reduce = function(obj, memo, iterator, context) {
  if (obj && obj.reduce) return obj.reduce(_.bind(iterator, context), memo);
  _.each(obj, function(value, index, list) {
    memo = iterator.call(context, memo, value, index, list);
  });
  return memo;
};
```
1. javascript에서 기본적으로 제공되는 reduce가 사용되도록 하는 코드가 추가되었습니다.
2. iterator의 argument에 list를 추가할 수 있게 되었습니다.

-------------------------------

[~v0.5.1]
```javascript
_.reduce = function(obj, memo, iterator, context) {
  if (obj && _.isFunction(obj.reduce)) return obj.reduce(_.bind(iterator, context), memo);
  _.each(obj, function(value, index, list) {
    memo = iterator.call(context, memo, value, index, list);
  });
  return memo;
};
```
1. obj에 기본 reduce가 있는지 체크하는 방식이 obj.reduce에서 _.isFunction(obj.reduce)로 변경되었습니다.

-------------------------------

[~v0.6.0]
```javascript
_.reduce = function(obj, memo, iterator, context) {
  if (nativeReduce && obj.reduce === nativeReduce) return obj.reduce(_.bind(iterator, context), memo);
  each(obj, function(value, index, list) {
    memo = iterator.call(context, memo, value, index, list);
  });
  return memo;
};
```
1. native reduce체크하는 코드가 추가 및 변경되었습니다.

-------------------------------

[~v1.1.3]
```javascript
_.reduce = _.foldl = _.inject = function(obj, iterator, memo, context) {
  var initial = memo !== void 0;
  if (nativeReduce && obj.reduce === nativeReduce) {
    if (context) iterator = _.bind(iterator, context);
    return initial ? obj.reduce(iterator, memo) : obj.reduce(iterator);
  }
  each(obj, function(value, index, list) {
    if (!initial && index === 0) {
      memo = value;
    } else {
      memo = iterator.call(context, memo, value, index, list);
    }
  });
  return memo;
};
```
1. argument의 순서가 변경되었습니다.(내부적으로 누적값에 사용되는 memo argument와 interator argument)
2. foldl와 inject란 이름이 생겼습니다.
3. 처음 초기값(memo)가 있는지 확인한 후 초기값이 있으면 해당 초기값 이후부터 값을 누적시킵니다.

-------------------------------

[~v1.2.3]
```javascript
_.reduce = _.foldl = _.inject = function(obj, iterator, memo, context) {
  var initial = arguments.length > 2;
  if (obj == null) obj = [];
  if (nativeReduce && obj.reduce === nativeReduce) {
    if (context) iterator = _.bind(iterator, context);
    return initial ? obj.reduce(iterator, memo) : obj.reduce(iterator);
  }
  each(obj, function(value, index, list) {
    if (!initial) {
      memo = value;
      initial = true;
    } else {
      memo = iterator.call(context, memo, value, index, list);
    }
  });
  if (!initial) throw new TypeError('Reduce of empty array with no initial value');
  return memo;
};
```
1. reduce을 수행할 데이터와 초기값이 둘 다 없을경우 에러메세지를 보여줍니다. (기본 reduce function이 리턴하는 에러메시지와 동일)
2. initial의 체크 방식이 변경되었습니다. memo가 입력됬는지의 유무가 아닌 parameter를 몇 개나 넣어주었는지를 판정합니다.
3. null 체크가 추가되었습니다.

-------------------------------

[~v1.7.0]
```javascript
var reduceError = 'Reduce of empty array with no initial value';

_.reduce = _.foldl = _.inject = function(obj, iteratee, memo, context) {
  if (obj == null) obj = [];
  iteratee = createCallback(iteratee, context, 4);
  var keys = obj.length !== +obj.length && _.keys(obj),
      length = (keys || obj).length,
      index = 0, currentKey;
  if (arguments.length < 3) {
    if (!length) throw new TypeError(reduceError);
    memo = obj[keys ? keys[index++] : index++];
  }
  for (; index < length; index++) {
    currentKey = keys ? keys[index] : index;
    memo = iteratee(memo, obj[currentKey], currentKey, obj);
  }
  return memo;
};
```

1. each나 map에서도 사용된 함수인 createCallback가 적용되었습니다.
2. keys에는 reduce를 수행할 data가 object형태일 경우에는 object의 key들을 갖고, 배열일 경우 false값을 같습니다.
3. 초기값이 들어오지 않았을 경우, 배열은 첫번째 인덱스의 값을 초기값으로 잡고, 오브젝트의 경우에도 첫번째 값을 초기값으로 설정합니다.
4. 기존에 underscore에 제공한 each문이나, js 기본 reduce function을 사용하는 부분이 사라졌습니다.

------------------------------

[~v1.8.3]
```javascript
var createReduce = function(dir) {
  // Wrap code that reassigns argument variables in a separate function than
  // the one that accesses `arguments.length` to avoid a perf hit. (#1991)
  var reducer = function(obj, iteratee, memo, initial) {
    var keys = !isArrayLike(obj) && _.keys(obj),
        length = (keys || obj).length,
        index = dir > 0 ? 0 : length - 1;
    if (!initial) {
      memo = obj[keys ? keys[index] : index];
      index += dir;
    }
    for (; index >= 0 && index < length; index += dir) {
      var currentKey = keys ? keys[index] : index;
      memo = iteratee(memo, obj[currentKey], currentKey, obj);
    }
    return memo;
  };

  return function(obj, iteratee, memo, context) { 
    var initial = arguments.length >= 3;
    return reducer(obj, optimizeCb(iteratee, context, 4), memo, initial);
  };
};

_.reduce = _.foldl = _.inject = createReduce(1);
```
1. createReduce()를 통해 reduce가 수행되도록 변경되었습니다.
2. return 되는 값이 function 지정되게 하여 실행된 parameter를 받는 구조입니다. (ex: function(dir)(argument))
3. 주석으로 달려있거나 추가적으로 코드 변경에 따른 장점은 더 공부해야 할 필요한 부분인것 같습니다.