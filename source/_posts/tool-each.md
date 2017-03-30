---
title: Grunt, Gulp, Webpack
categories: Tool
tags: Tool
keywords:
  - Tool
  - Gulp
  - Grunt
  - Browserify
  - Webpack
date: 2017-03-29 08:20:33
---

### 2017년에 Grunt와 Gulp를 언급하는 이유

현재 대세는 단연코 Webpack이라고 할 수 있습니다.
하지만 모든 새로운 것들에는 그들이 나오게 된 이유가 있다고 생각됩니다.

Webpack을 더 잘 사용하기위해서는 이전 Grunt와 Gulp에 대한 이해가 필요하다는 생각이 들어서 간단히 정리해보려고 합니다.

<!-- more -->

### Grunt와 Gulp

Grunt와 Gulp는 사람의 실수나 반복적인 작업을 줄이기 위한 자동화 툴(task runner)입니다.

보통 CSS와 Javascript 파일을 concat, minify, compress, uglify를 하는데 많이 사용되죠,
즉 사전에 필요한 반복적인 작업들을 간단한 작업만으로 진행할 수 있습니다.

또한, 이들은 Node.js에서 사용할 수 있으며 여러 Plugin을 활용 할 수 있습니다

### Grunt vs Gulp

위에서 처럼 두가지 툴 모두 같은 작업을 수행하지만, 수행하는 방법은 차이가 있습니다

#### 설정방법

Grunt는 설정에 기반하여 동작하는 반면, Gulp는 Javascript를 기반으로 동작합니다.

아래는 Grunt의 설정 예 입니다.

```javascript
// Grunt는 JSON 형태의 confing를 구성하는 방식으로 구현됩니다
imagemin: {
    jpgs: {
        options: {
        progressive: true
        },
        files: [{
            expand: true,
            cwd: 'src/img',
            src: ['*.jpg'],
            dest: 'images/'
        }]
    }
}
```

설정파일 기반의 Grunt와 달리 Gulp는 Javascript (Nodejs) 코딩을 할 줄 아는 사람이면 쉽게 접근할 수 있는 것이 장점인것 같습니다.

```javascript
// Gulp는 javascript를 사용합니다.
gulp.task('jpgs', function() {
    return gulp.src('src/images/*.jpg')
    .pipe(imagemin({ progressive: true }))
    .pipe(gulp.dest('optimized_images'));
});
```

#### 속도

Gulp와 Grunt의 속도의 차이를 이해하기 위해서는 둘의 처리 방식이 다르다는걸 알아야 합니다.

Gulp는 스트림(Stream)을 기반으로 하는 빌드 시스템 입니다. 스트림을 이용해 데이터를 읽고 출력하며 작업들을 메모리에서 처리합니다. 즉, 요청 후 한번에 결과를 받는 것이 아니라 이벤트 중간중간 전달받아 작업을 하기 때문에 비교적 작업속도가 빠릅니다.

> 스트림(Stream)이란?
  데이터의 입, 출력시 비동기적으로 처리될 수 있는 데이터의 연속된 흐름으로써, Node.js에서는 이 스트림을 읽고 쓸 수 있습니다.

또한 Gulp는 동시에 여러 작업을 처리 할 수 있지만 Grunt는 일반적으로 한 번에 하나의 작업 만 처리합니다.

> 출처 :
* [Speedtesting gulp.js and Grunt](http://tech.tmw.co.uk/2014/01/speedtesting-gulp-and-grunt/)
* [Introduction to Node.js Streams](http://juhoi.github.io/post/%EB%B2%88%EC%97%AD%5D-introduction-to-node-dot-j-s-streams/)

#### 결론

Gulp가 Grunt보다 속다가 빠르다는건 맞지만, 사용의 편의성 또한 무시할 수 있는 부분이 아니기에 무엇이 자신에게 맞는지를 잘 파악해서 사용하는게 좋은것 같습니다.

> 출처 :
  * [Gulp vs Grunt – Comparing Both Automation Tools](https://www.keycdn.com/blog/gulp-vs-grunt/)
  * [Grunt vs Gulp - Beyond the Numbers](https://jaysoo.ca/2014/01/27/gruntjs-vs-gulpjs/)
  * etc...

### Webpack

Gulp와 Grunt와 같은 자동화 도구덕분에 많은 작업의 양이 줄었는데도 불구하고 Webpack이 나왔습니다. 이런 자동화도구에서 어떤 부분이 불편했던 것이었을까요? 우선 Webpack은 모듈 번들러이고 Gulp와 Grunt는 task runners입니다.

결론부터 말하자면 Webpack이 이들을 1:1로 대체된다기 보다 이 기능이 포함되어 있고 더 많은 작업을 할 수 있기 때문에 이 도구를 쓰는것입니다. webpack은 Browserify와 같은 의존성 관리 기능까지 포함하고 있으니 "webpack = (Grunt || Gulp) + Browserify" 이게 될텐데 안 쓸 이유가 없을 것입니다. 게다가 속도까지 더 빠르니까요.

#### 우선 Browserify?

Browserify 는 Node.js 기반 javascript code 를 브라우저 환경에서도 실행 가능하도록 해줍니다.

```javascript
var path = require('path');
var foo = require('./helpers/foo')

module.exports = function bar () {
  return 'bar'
};
```

즉, 브라우저와 Node.js의 코드를 동일하게 사용할 수 있습니다.

하지만 브라우저에서 작업결과를 봐야하기 때문에 매번 코드를 컴파일이 해야하는데, 코드의 양이 많아지면 하나 수정하는데도 엄청 오래걸릴 수 있다는 얘기가 됩니다.

참고한 블로그를 보면 webpack을 사용해 watch한 경우 빌드시간이 20초 0.1 ~ 0.3으로 단축되었다는 글을 볼 수 있었습니다.

실제로 정확히 이해하고 적용하는데는 시간이 걸리겠지만, 좋은 부분이 있다는 건 확실히 이해해야 할것 같습니다.


> 출처 :
* [Webpack – 모듈화와 속도를 모두 잡는 방법.](https://ironhee.com/2015/06/06/webpack-%EB%AA%A8%EB%93%88%ED%99%94%EC%99%80-%EC%86%8D%EB%8F%84%EB%A5%BC-%EB%AA%A8%EB%91%90-%EC%9E%A1%EB%8A%94-%EB%B0%A9%EB%B2%95/)
* [Webpack or Browserify & Gulp: Which Is Better?](https://www.toptal.com/front-end/webpack-browserify-gulp-which-is-better)
* etc...

