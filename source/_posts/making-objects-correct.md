---
title: 객체 바르게 만들기
categories: 
  - js
  - core
tags: 
  - js
  - object
  - pattern
  - prototype
keywords:
  - js
  - object
  - pattern
  - prototype
thumbnailImage: https://raw.githubusercontent.com/Jae-kwang/blog/master/source/img/javascript.png
date: 2018-03-01 14:27:00
---

프로젝트를 구조화하면서 객체의 생성방식은 매우 중요해집니다. 여러가지 객체생성 패턴을 살펴보고 제가 겪었던 문제를 간단히 정리해보았습니다.

<!-- more -->

객체 생성 패턴에 대한 간략한 정리이며, 하단부분에는 저의 생각을 작성했습니다. 읽는데 참고하시면 좋겠습니다. "자바스크립트 패턴과 테스트" 책을 참고하였습니다.

### 1. 모듈 패턴
모듈 패턴은 `데이터 감춤이 주목적인 함수`가 `모듈 API`를 이루는 `객체`를 반환하게 하는 구조이며, 두가지 기반의 모듈 생성 방식이 있습니다.

#### 1.1 임의 모듈 생성

{% tabbed_codeblock 임의 모듈 패턴 예시 %}
 <!-- tab javascript -->
var MyApp = MyApp || {};

MyApp.wildlifePreserveSimulator = function(animalMaker) {
  var animals = []; // 프라이빗 변수
  return { // API 반환
    addAnimals: function(species, sex) {
      animals.push(animalMaker.make(species, sex));
    },
    getAnimalCount: function() {
      return animals.legnth;
    }
  }
}

// 사용
var perserve = MyApp.wildlifePreserveSimulator(realAnimalMaker);
preserve.addAnimal(gorilla, female);
<!-- endtab -->
{% endtabbed_codeblock %}

#### 1.2 즉시 실행 모듈 생성

API 반환하는건 임의 모듈과 같지만, 외부 함수를 선언하자마자 실행하는 방법입니다.
반환된 API는 이름공간을 가진 전역 변수에 할당된 후 해당 모듈의 싱글톤 인스턴스가 됩니다.

{% tabbed_codeblock 싱글톤 모듈 %}
 <!-- tab javascript -->
var MyApp = MyApp || {};

MyApp.wildlifePreserveSimulator = (function() {
  var animals = [];
  return {
    addAnimals: function(animalMaker, species, sex) {
      animals.push(animalMaker.make(species, sex));
    },
    getAnimalCount: function() {
      return animals.legnth;
    }
  }
}());

// 사용
MyApp.wildlifePreserveSimulator.addAnimal(realAnimalMaker, gorilla, female);
<!-- endtab -->
{% endtabbed_codeblock %}

외부 함수는 애플리케이션 기동 코드의 실행과 상관없이 코드가 작성된 지점에서 즉시 실행되기 때문에 함수 (즉시) 실행시 의존성을 가져오지 못하며 외부 함수에 주입할 수 없습니다.

### 2. new 객체 생성 패턴

{% tabbed_codeblock Marsupial함수와 생성자 함수 사용법 %}
 <!-- tab javascript -->
function Marsupial(name, nocturanl) {
  this.name = name;
  this.isNocturnal = nocturanl;
}

var maverick = new Marsupial('매버릭', true);
var slider = new Marsupial('슬라이더', false);

console.log(maverick.isNocturnal); // true
console.log(maverick.name);        // '매버릭'
console.log(slider.isNocturnal);   // false
console.log(slider.name);          // '슬라이더'
<!-- endtab -->
{% endtabbed_codeblock %}

Marsupial함수는 주어진 인자를 내부적으로 생성할 인스턴스의 프로퍼티에 할당한다. 다음 줄에서 maverick, slider라는 이름으로 생성한 두 Marsupial 인스턴스는 각자 고유한 프로퍼티 값을 가진다.

자바스크립트 언어는 Marsupial 함수를 생성자 함수(new 키워드와 함께 사용하려고 작성한 함수)로 사용하라고 강요하지 않는다. 즉, new 키워드 없이 생성자 함수를 사용해도 이를 못하게 막을 보호 체계가 없다. 그래서 더러 개발자들은 파스칼 표기법(PascalCase)으로 생성자 함수를 따로 표기하여 구분하기도 한다.

### 3. 생성자 함수 + 프로토 타입 함수

{% tabbed_codeblock Marsupial함수와 생성자 함수 사용법 %}
 <!-- tab javascript -->
function Marsupial(name, nocturanl) {
  this.name = name;
  this.isNocturnal = nocturanl;
}

Marsupial.prototype.isAwake = function(isNight) {
  return isNight === this.isNocturnal; 
}

var maverick = new Marsupial('매버릭', true);
var slider = new Marsupial('슬라이더', false);

var isNightTime = true;

console.log(maverick.isAwake(isNightTime)); // true
console.log(slider.isAwake(isNightTime));   // false
<!-- endtab -->
{% endtabbed_codeblock %}

생성자 함수의 프로토타입에 함수를 정의하면 객체 인스턴스를 대량 생성할 때 함수 사본 개수를 한개로 제한하여 메모리 점유율을 낮추고 성능까지 높이는 추가 이점이 있습니다.

### 4. 올바른 객체 생성에 대한 나의 생각

구조화를 위해 데이터의 객체화에 대한 필요성을 느끼기 시작했습니다. 그래서 흔히들 말하는 모듈패턴과 생성자 함수 프로토 타입을 익히는데 노력했습니다.

그리고는 특별히 이상한 점을 느끼지 못하고는 이리저리 패턴들을 적용하기에 이르렀습니다. 하지만 자바스크립트가 아시다시피 모든걸 묵인하고 인용하며 에러는 주지않고 "너 하고싶은대로 하거라" 라는  부처님 같은 녀석이라서 이것에 대한 오용과 남용(?)에 대해 잘 인지하지 못하고 있었습니다.</br>
{% image center Buddha.jpeg 마음대로 만들어도 괜찮아~ %}

결국 전 이런 코드까지 쓰게 된거죠. 끔찍한 혼종을 만들어 버리고 맙니다.

``` javascript
function Car() {
  this.oil = 0;
  return {
    getOil: function() {
      return oil;
    }
  }
}

Car.prototype.setOil = function(newOil) {
  this.oil = newOil;
}

// 모듈패턴 처럼 사용
var myCar = Car();

// 생성자 함수 처럼 사용
var myNewCar = new Car();
```
당연하지만 생각처럼 동작지 않습니다. 두 경우 모두 `setOil`을 찾아볼 수 없습니다. 모듈 패턴에서는 당연히 myCar가 Car의 인스턴스가 아니기 때문이며, 생성자 패턴에서는 return의 대상이 변경되어서 더 이상 생성자의 역할을 하지 못하기 때문입니다.

이를 통해 알게 된 점은 목적에 맞는 방법으로 객체를 생성해야 한다는 점입니다. 그러니 모듈 패턴은 유틸과 같은 한 개의 모듈로 동작하게 되는 경우가 적당하며, 여러 개의 인스턴스가 필요한 경우에는 생성자 패턴을 적용해야 합니다.

이름에서 그 패턴의 목적이 잘 녹아있음에도 불구하고, 자바스크립트의 특성에 의해 전혀 생각지도 못하게 쓰고 있었던 것 같습니다.

비록 실제 서비스가 `Car`와 같이 쉽게 객체를 명확히 나눌 수 있는 것은 아닙니다. 객체 자체를 어떻게 구조조화 할건지는 참 어려운 일 중 하나인 것 같습니다. 하지만 이제 스스로도 객체를 생성할 때 좀 더 명확한 판단의 기준을 가지고 효율 적인 객체를 생성하는데 큰 도움이 될 것 같습니다. 😃
