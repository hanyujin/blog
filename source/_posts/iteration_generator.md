---
title: Iteration & Generator
categories: 
  - js
  - ES6
tags:
  - javascript
  - ES6
  - iteration
  - generator
keywords:
  - javascript
  - ES6
  - interface
  - iterator
  - iterable 
date: 2018-05-01 16:57:22
---

ES6의 Iteration & Generator에 대하여 알아봅니다.

<!-- more -->

우선, Iteration & Generator를 알아보기에 앞서 Interface에서 대하여 알아보도록 하겠습니다.

#### Interface

우선, Javascript에서의 Interface는 고유명사입니다.
다른 언어에서 말하는 Interface와는 아무 상관이 없습니다.

**ES6에서의 Interface의 정의**

1. Interface란 사양에 맞는 값과 연결된 속성키의 세트.
2. 어떤 Object라도 Interface의 정의를 충족시킬 수 있다.
3. 하나의 Object는 여러 개의 Interface를 충족시킬 수 있다.

> http://www.ecma-international.org/ecma-262/6.0/#sec-iteration

예를 들어 Interface 'TEST'를 만들어 보겠습니다.

**Interface 'TEST'의 요구 사항**
1. 'test'라는 키를 가진다.
2. 'test'에는 함수가 있으며, 문자열 인자를 한 개 받고, Boolean 값을 반환한다.

```javascript
const obj = { // TEST'Interface 를 만족하는 object의 형태
  test(str) {
    return true;
  }
}

// class를 활용한 정의에서도 Interface를 만족하게 하면,
// 그 인스턴스들은 자동으로 원래의 Interface의 요구사항을 만족한다.
const Test = class {
  test(str) {
    return true;
  }
}

const test = new Test();
```
**덕 타이핑(duck typing)**
class Test가 test Interface 조건만 만족한다면 test Interface의 구상체로 볼 수 있습니다.
덕 타이핑을 쓰는 이유는 런타임에는 컴파일타임의 타입체크가 되지 않기 때문입니다.
즉, Javascript에서 말하는 Interface는 덕 타이핑으로 구현을 약속한 프로토콜 같은 것이라고 볼 수 있습니다.

#### Iterator Interface

Javascript 스펙에는 미리 규정된 Interface들이 있습니다.
그 중 Iterator Interface를 살펴보겠습니다.

**Iterator Interface의 요구 사항**
1. next라는 키를 가진다.
2. next에는 함수가 있으며, 인자를 받지 않고 `iteratorResultObject`를 반환한다.
3. `iteratorResultObject`는 `value`와 `done`이라는 키를 갖고 있다.(interface)
4. 이중 `done`은 계속 반복할 수 있을지 없을지에 따라 Boolean 값을 반환한다.
http://www.ecma-international.org/ecma-262/6.0/#sec-iterator-interface 


```javascript
const iterator = { // iterator Interface를 만족하는 object의 형태
  next() {
    return { // Interface iteratorResultObject를 만족하는 object의 형태
      done: true,
      value: 1
    };
  }
};

// 데이터를 갖는 iterator 활용 예제 
const iterator = {
  data: [1, 2, 3, 4],
  next() {
    return { 
      done: this.data.length === 0,
      value: this.data.pop()
    };
  }
};

// iterator 수행 
let iResult = iterator.next()
console.log(iResult.value + ':' + iResult.done); // 4:false
iResult = iterator.next()
console.log(iResult.value + ':' + iResult.done); // 3:false
iResult = iterator.next()
console.log(iResult.value + ':' + iResult.done); // 2:false
iResult = iterator.next()
console.log(iResult.value + ':' + iResult.done); // 1:false
iResult = iterator.next()
console.log(iResult.value + ':' + iResult.done); // undefined:true
// value가 없으면 undefined를 return하고 done은 true가 된다.
```

이렇게 Iterator Interface을 적용하여 iterator object를 만들어 보았고,
반복수행을 해보았습니다. 다음은 Iterable Interface에 대하여 알아보겠습니다. 

#### Iterable Interface

**Iterable Interface의 요구 사항**
1. Symbol.iterator라는 키를 갖는다.
2. Symbol.iterator에는 함수가 있으며, 인자를 받지 않고 iterator object를 반환한다. 

```javascript
const iterable = { // iterable Interface를 만족하는 object의 형태
  [Symbol.iterator]() {
    return iterator; // iterator object
  }
}
```
iterator는 한번 반복 후에는 수명을 다해서 없어집니다.
그래서 원본을 그대로 둔 상태에서 계속 반복 가능한 객체를 새롭게 얻기 위해 iterable을 사용합니다.

#### Loop

ES6에서는 ES5에서 제공되는 기본 기능들에도 변화가 생겼습니다.

```javascript
const a = {
 '0': 3,
  a: 5,
  [Symbole()]: 7,
  b: 6
}
```
ES5으로 for .. in은 순서보장이 지원되지 않았는데,
ES6에서는 숫자, 알파벳, 심볼로 순서로 정렬되서 결과를 나옵니다.(생성은 순서대로 진행된다.)

#### while 문으로 살펴보는 iterator

``` javascript
let arr = [1,2,3,4];
while(arr.length > 0) { // 계속 반복할지 판단
  console.log(arr.pop()); //반복시 마다 처리할 것
}

const iterator = {
  data: [1, 2, 3, 4],
  next() {
    return { 
      done: this.data.length === 0, // 계속 반복할지 판단
      value: this.data.pop() // 반복시 마다 처리할 것
    };
  }
};
```
while문과 iterator객체가 매우 유사하다는 사실을 알 수 있습니다. 

차이점으로는 while은 loop를 '문'으로 돌아서 제어가 불가능 하지만 iterator는 '값'으로 돌기 때문에 제어할 수 있습니다. 또한 next() 함수로 실행하기 때문에 지연실행이 가능합니다.

또한 while은 반복을 엔진이 해주지만 iterator는 외부에 실행기를 두어야 합니다.

즉, **iterator object**는 반복 자체를 하지는 않지만, 외부에서 반복하려고 할 때 반복에 필요한 조건과 실행을 미리 준비해둔 **객체** 입니다.

이를 통해 반복행위와 반복을 위한 준비를 분리했다고 볼 수 있습니다.

#### ES6+ Loop
```javascript
// 사용자 반복처리기 (직접 Iterator 반복 처리기를 구현)
const loop = (iter, f) => {

  // iterable이라면 iterator를 얻음
  if (typeof iter[Symbol.iterator] === 'function') {
    iter = iter[Symbol.iterator]();
  }

  // iteratorObject가 아니라면 건너뜀
  if (typeof iter.next != 'function') return;

  while(true) {
    const = v = iter.next();
    if (v.done) return; // 종료 처리
    f(v.value) // 현재 값을 전달
  }
}

// iterable이자, iterator인 객체
const iter = {
  [Symbole.iterator](){return this}
  data: [1, 2, 3, 4],
  next() {
    return { 
      done: this.data.length === 0,
      value: this.data.pop()
    };
  }
};

loop(iter, console.log);
// 4
// 3
// 2
// 1
```

loop라는 반복기를 통해 iterator object를 순회합니다.

#### destructuring && For of 

##### destructuring 
```javascript
const [a,...b] = iter;
console.log(a, b);
//4, [3, 2, 1]
```
실제로 destructuring은 배열을 해체하는 게 아니라 iterable을 해체하는 것입니다.
배열(문자열)을 넣었을 때도 destructuring이 정상작동하는 원리는 배열(문자열)에도 [Symbole.iterator]가 존재하다는 것을 알 수 있습니다.

이처럼 기본에 알고 있는 데이터의 구조도 ES5와는 차이가 있습니다. 

``` javascript
console.log(typeof ""[Symbol.iterator]); // true
console.log(typeof [][Symbol.iterator]); // true
```

##### For of 
새롭게 추가된 iterator를 소비하는 Loop로 기존 제어문이 소비하는 형태의 Loop 입니다.


#### Iterator Example 
```javascript
// 제곱을 요소로 갖는 가상 컬렉션 
const N2 = class {
  constructor(max) {
    this.max = max;
  }
  [Symbol.iterator]() {
    let cursor = 0, max = this.max;
    return {
      done: false,
      next() {
        if (cursor > max) {
          this.done = true;
        } else {
          this.value = cursor * cursor;
          cursor++;
        }
        return this;
      }
    };
  }
};

console.log([...new N2(5)]); // [0, 1, 4, 9, 16]
for(const v of new N2(5)) {
  console.log(v);
}
// 0
// 1
// 4
// 9
// 16
```
데이터 자체가 외부의 반복기에 의존하지 않고 원하는 조건에 의하여 데이터를 생성할 수 있게 되었습니다.

#### Generator

Generator를 가장 쉽게 사용하는 예로는 iterator generator로써 사용하는 것입니다.
Iterator generator라는걸 풀어서 생각해보면 iterator 생성기라는 말인데,
위에서 같이 보신 것처럼 지금까지 iterator 생성기의 역할은 iterable이 하고 있었습니다. 

{% alert info %}
iterator를 만든다는 입장에서 generator와 iterable은 같다고 볼 수 있다.
{% endalert %}

- Iterable이 iterator를 만드는 방식
`Symbol.iterator`호출하는 것에 의해서 그 반환 값으로 호출 당할 때마다 iterator를 만들어 낸다.

- Generator는 Generator를 호출할 때 마다 그 결과로 iterator를 만들어 낸다.

그럼 실제로 우리는 iterable, iterator 인터페이스를 모두 알아야만 사용 가능할까요?
그건 너무 불친절한 일입니다. 그래서 그걸 몰라도 사용할 수 있는 문법적 장치가 바로 `generator`입니다.

```javascript
const generator = function*(max) {
  let cursor = 0;
  while(cursor < max) {
    yield cursor * cursor; // value
    cursor ++;
  }
};
// 1. return이 없지만 return하면 iterator 객체가 나온다.
// 2. 제어문을 멈출 수 있다.(지연 실행이 가능하다.)

console.log([...generator(5)]) // [0, 1, 4, 9, 16]
for(const v of generator(5)) {
  console.log(v);
}
// 0
// 1
// 4
// 9
// 16
```

#### 마치며
iterable, iterator은 ES6에서 완전히 새로 나온 개념이라서 이해하기가 쉽지 않았습니다. 하지만 반면에 새로운 기능이라서 기존의 기능을 확장해서 알아야 할 필요는 없다는 생각으로 이해하려고 노력했습니다.

평소 프로그래밍을 하다 보면 당연하게 데이터를 반복하려면 당연히 반복기가 필요하고, 그 반복기로 순회를 할 때는 멈출 수 없다는 게 당연한 생각이었습니다.

그래서 ES6에서의 generator가 이를 개선해줄 거라고 나왔을 때도 반가워하며 찾아봤지만 이해하기가 쉽지 않았습니다. 하지만 이 기회에 interface를 기반으로 이해하려니까 더 잘 이해가 되는 것 같습니다.

ES6의 데이터가 ES5의 데이터와 다르게 iterator의 interface를 갖고 있다는 것만으로도 많은 이해가 됐습니다.

이제는 generator를 활용하며 익숙해져야겠습니다. :)

> 출처 : https://www.youtube.com/watch?v=GhAkc00TvZs
