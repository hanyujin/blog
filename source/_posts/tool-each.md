---
title: Grunt, Gulp, Webpack
categories: Tool
tags: Tool
keywords:
  - tool
date: 2017-03-29 08:20:33
---

### 2017년에 Grunt와 Gulp를 언급하는 이유 

현재 대세는 단연코 Webpack이라고 할 수 있습니다.
하지만 모든 새로운 것들에는 그들이 나오게 된 이유가 있다고 생각됩니다
Webpack을 더 잘 사용하기위해서는 이전 Grunt와 Gulp에 대한 이해가 필요하다는 생각이 듭니다.
 
이 글은 사용법에 대한 글이 아닙니다.
 
<!-- more -->

### Grunt and Gulp 

Grunt와 Gulp는 사람의 실수나 반복적인 작업을 줄이기 위한 자동화 툴입니다.

보통 CSS와 Javascript 파일을 concat, minify, compress, uglify를 하는데 많이 사용됩니다.
이와 같이 사전에 필요한 반복적인 작업들을 간단한 작업만으로 진행할 수 있습니다.

또한, 이들은 Node.js에서 사용할 수 있으며 여러 Plugin을 활용 할 수 있습니다


### Grunt vs Gulp

위에서 처럼 두가지 툴 모두 같은 작업을 수행하지만, 수행하는 방법은 차이가 있습니다

1. 설정방법

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


```javascript
// Gulp는 javascript를 사용합니다.
gulp.task('jpgs', function() {
    return gulp.src('src/images/*.jpg')
    .pipe(imagemin({ progressive: true }))
    .pipe(gulp.dest('optimized_images'));
});
```

이처럼 각각 설정한는 형태가 다르기때문에 무엇이 더 사용하기 편하다는 부분은 의견이 갈리는 부분입니다.
하지만 저는 gulp가 grunt보다 더 익숙해서 좋은것 같습니다.

2. 속도

Gulp와 Grunt의 속도를 이해하기위해서는 둘의 처리 방식이 다르다는걸 알아야 합니다.

Gulp는 steam을 이용해 데이터를 읽고 출력하며 작업들을 메모리에서 처리합니다.

> 스트림(Stream)이란?
  데이터의 입, 출력시 비동기적으로 처리될 수 있는 데이터의 연속된 흐름으로써, 
  Node.js에서는 이 스트림을 읽고 쓸 수 있다.

또한 Gulp는 동시에 여러 작업을 처리 할 수 ​​있지만 Grunt는 일반적으로 한 번에 하나의 작업 만 처리합니다.

3. 결론

많은 자료를 조사하면서 알게 된 부분이지만 어느 누구하나 뭐가 더 좋다라고 하는 경우를 보지 못했습니다.
대체적으로 많은 부분에서 gulp가 grunt보다 속다가 빠르다는건 맞지만, 확연한 차이가 나는것은 아니기에 무엇이 자신에게 맞는지를 잘 파악해서 사용하라는게 대부분의 결론입니다.

그래서 저는 Gulp가 더 좋은것 같아 보입니다 :)

> 출처
  1. https://www.keycdn.com/blog/gulp-vs-grunt/
  2. https://jaysoo.ca/2014/01/27/gruntjs-vs-gulpjs/
  3. http://juhoi.github.io/post/%EB%B2%88%EC%97%AD%5D-introduction-to-node-dot-j-s-streams/
  4. http://tech.tmw.co.uk/2014/01/speedtesting-gulp-and-grunt/
  5. etc...
  
사실 Grunt와 Gulp에 대해 생각보다 많이 알아보았는데 대체적으로 강조하가나 언급하는 부분들이 대부분 유사하다보니 더 짧고 간단하게 정리된것 같다.
물론 더 많은 자료들이 있습니다.

### Webpack

Gulp와 Grunt와 같은 자동화 도구덕분에 많은 작업의 양이 줄었는데도 불구하고 Webpack이 나왔습니다.
이런 자동화도구에서 어떤 부분이 불편했던 것이었을까요?

우선 Webpack은 모듈 번들러이고 Gulp와 Grunt는 task runners입니다.

Webpack이 이들을 1:1로 대체된다기 보다 이 기능이 포함되어 있고 더 많은 작업을 할 수 있기 때문에 이 도구를 쓰는것입니다.

실제로 webpack은 Browserify의 대체라고 말할 수 있을것 같습니다.

> 출처
1. https://www.toptal.com/front-end/webpack-browserify-gulp-which-is-better
2. etc
