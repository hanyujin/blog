---
title: Underscore의 _.reduceRight 
categories: Javascript
tags: 
  - Javascript
  - underscore.js
keywords:
  - Javascript
  - library
date: 2017-05-23 09:14:00
---

### Underscore 의 _.reduceRight

v1부터 현재까지 코드가 어떻게 변해왔는지 확인해봅니다.

<!-- more -->

reduceRight는 reduce를 내림차순으로 수행하는 함수입니다.

[~v0.3.3]

```javascript
 _.reduceRight = function(obj, memo, iterator, context) {
    if (obj && obj.reduceRight) return obj.reduceRight(_.bind(iterator, context), memo);
    var reversed = _.clone(_.toArray(obj)).reverse();
    _.each(reversed, function(value, index) {
      memo = iterator.call(context, memo, value, index, obj);
    });
    return memo;
  };
```
1. 기본 reduceRight가 있으면 해당 함수를 사용하도록 합니다.
2. 조작할 데이터를 _.clone을 통해 복사를 한후 .reverse()를 통해 내림차순으로 돌립니다.
3. 이후는 reduce와 동일하게 콜백을 수행합니다.

--------------------------

[~v0.5.1]

```javascript
_.reduceRight = function(obj, memo, iterator, context) {
  if (obj && _.isFunction(obj.reduceRight)) return obj.reduceRight(_.bind(iterator, context), memo);
  var reversed = _.clone(_.toArray(obj)).reverse();
  _.each(reversed, function(value, index) {
    memo = iterator.call(context, memo, value, index, obj);
  });
  return memo;
};
```
1. reduce함수와 동일하게 같은버전에서 obj에 기본 reduceRight가 있는지 체크하는 방식이 obj.reduceRight _.isFunction(obj.reduceRight)로 변경되었습니다.

--------------------------

[~v0.6.0]
```javascript
_.reduceRight = function(obj, memo, iterator, context) {
  if (nativeReduceRight && obj.reduceRight === nativeReduceRight) return obj.reduceRight(_.bind(iterator, context), memo);
  var reversed = _.clone(_.toArray(obj)).reverse();
  return _.reduce(reversed, memo, iterator, context);
};
```

1. native ReduceRight를 체크하는 코드가 추가 및 변경되었습니다.(reduce와 동일합니다.)
2. 반복문 도는 구간이 _.each에서 _.reduce로 대체되었습니다.

--------------------------

[~v1.1.0]
```javascript
_.reduceRight = function(obj, iterator, memo, context) {
  if (nativeReduceRight && obj.reduceRight === nativeReduceRight) {
    if (context) iterator = _.bind(iterator, context);
    return obj.reduceRight(iterator, memo);
  }
  var reversed = _.clone(_.toArray(obj)).reverse();
  return _.reduce(reversed, iterator, memo, context);
};
```
1. reduce와 동일하게 argument의 순서가 변경되었습니다.(내부적으로 누적값에 사용되는 memo argument와 interator argument)

--------------------------

[~v1.1.2]
```javascript
_.reduceRight = _.foldr = function(obj, iterator, memo, context) {
  if (nativeReduceRight && obj.reduceRight === nativeReduceRight) {
    if (context) iterator = _.bind(iterator, context);
    return obj.reduceRight(iterator, memo);
  }
  var reversed = (_.isArray(obj) ? obj.slice() : _.toArray(obj)).reverse();
  return _.reduce(reversed, iterator, memo, context);
};
```

1. 들어오는 데이터가 배열타입인지 확인후 배열인경우 .slice() 통해 shallow copy로 새로운 배열을 만들고, 배열이 아닌경우 _.toArray로 배열로 변환후 .reverse()를 이용해 내림차순으로 변경합니다.
2. foldr란 이름이 생겼습니다.
--------------------------
[~v1.1.3]
```javascript
_.reduceRight = _.foldr = function(obj, iterator, memo, context) {
  if (nativeReduceRight && obj.reduceRight === nativeReduceRight) {
    if (context) iterator = _.bind(iterator, context);
    return memo !== void 0 ? obj.reduceRight(iterator, memo) : obj.reduceRight(iterator);
  }
  var reversed = (_.isArray(obj) ? obj.slice() : _.toArray(obj)).reverse();
  return _.reduce(reversed, iterator, memo, context);
};
```
1. 기본 reduceRight를 사용할때 초기값(memo)가 있는지 체크합니다.

--------------------------
[~v.1.1.4]
```javascript
_.reduceRight = _.foldr = function(obj, iterator, memo, context) {
  if (obj == null) obj = [];
  if (nativeReduceRight && obj.reduceRight === nativeReduceRight) {
    if (context) iterator = _.bind(iterator, context);
    return memo !== void 0 ? obj.reduceRight(iterator, memo) : obj.reduceRight(iterator);
  }
  var reversed = (_.isArray(obj) ? obj.slice() : _.toArray(obj)).reverse();
  return _.reduce(reversed, iterator, memo, context);
};
```
1. null체크가 추가되었습니다.

--------------------------
[~v1.2.3]
```javascript
_.reduceRight = _.foldr = function(obj, iterator, memo, context) {
  var initial = arguments.length > 2;
  if (obj == null) obj = [];
  if (nativeReduceRight && obj.reduceRight === nativeReduceRight) {
    if (context) iterator = _.bind(iterator, context);
    return initial ? obj.reduceRight(iterator, memo) : obj.reduceRight(iterator);
  }
  var reversed = _.toArray(obj).reverse();
  if (context && !initial) iterator = _.bind(iterator, context);
  return initial ? _.reduce(reversed, iterator, memo, context) : _.reduce(reversed, iterator);
};
```
1. 초기값인 memo를 판정하는 방법이 reduce와 동일하게 변경되었습니다.
2. 초기값에 따라 _.reduce를 실행에 필요한 parameter를 다르게 설정합니다.
--------------------------
[~v1.4.0]
```javascript
_.reduceRight = _.foldr = function(obj, iterator, memo, context) {
  var initial = arguments.length > 2;
  if (nativeReduceRight && obj.reduceRight === nativeReduceRight) {
    if (context) iterator = _.bind(iterator, context);
    return arguments.length > 2 ? obj.reduceRight(iterator, memo) : obj.reduceRight(iterator);
  }
  var length = obj.length;
  if (length !== +length) {
    var keys = _.keys(obj);
    length = keys.length;
  }
  each(obj, function(value, index, list) {
    index = keys ? keys[--length] : --length;
    if (!initial) {
      memo = obj[index];
      initial = true;
    } else {
      memo = iterator.call(context, memo, obj[index], index, list);
    }
  });
  if (!initial) throw new TypeError('Reduce of empty array with no initial value');
  return memo;
};
```
1. length가 없는경우 obj의 사이즈를 구하여 length를 대신 확인합니다.
2. 반복문이 적용되는 구간이 기존 _.reduce 대신 each()를 사용해 내림차순으로 순환하고 있습니다.
--------------------------
[~v1.7.0]
```javascript
_.reduceRight = _.foldr = function(obj, iteratee, memo, context) {
  if (obj == null) obj = [];
  iteratee = createCallback(iteratee, context, 4);
  var keys = obj.length !== + obj.length && _.keys(obj),
      index = (keys || obj).length,
      currentKey;
  if (arguments.length < 3) {
    if (!index) throw new TypeError(reduceError);
    memo = obj[keys ? keys[--index] : --index];
  }
  while (index--) {
    currentKey = keys ? keys[index] : index;
    memo = iteratee(memo, obj[currentKey], currentKey, obj);
  }
  return memo;
};
```
1. 다른 반목문들과 동일하게 추후 optimizeCb로 변경되는 createCallback function을 사용하여 callback 처리를 하고 있습니다.
2. 반복구간이 while로 사용되고 있습니다.
--------------------------
[~v1.8.0]
```javascript
_.reduceRight = _.foldr = createReduce(-1);
```

1. reduce와 반대로 createReduce에 1이 아닌 -1을 넣어 내림차순으로 reduce를 수행되도록 하고 있습니다.


