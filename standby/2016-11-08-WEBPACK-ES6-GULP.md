Webpack + ES6 + GULP
--------------------

오래된 모바일 프로젝트의 프론트엔드를 더 나은 구조로 개선하는 작업을 진행하려고 합니다. 그래서 설계된 구조로 스프링 부트에서 Webpack + ES6를 사용해보고 GULP를 이용한 개발, 배포 환경까지 정리해보려합니다.

### 구성 환경

#### 현재 상황

오래된 프로젝트라 그런 것인지, 외부 라이브러리 파일이 패키지 관리툴에 의해 관리되고 있지 않았습니다... 또한! 프로젝트에서 아주 많이 사용하고 있는 라이브러리지만, npm에는 존재하지 않는 오래된 js를 사용하고 있습니다.

빠르게 사라지고 있는 커피스크립트가 사용되고 있습니다. 유일하게 커피스크립트가 사용된 프로젝트이기때문에 담당자는 사장되고 있는 커피스크립트를 학습해야합니다.

#### 개선 구조

**패키지 관리**<br> 1차적으로 프로젝트의 구조를 개선하는 것이 우선인 작업이라, 사용되고 있는 구 라이브러리를 최신 라이브러리로 대체하기에는 일정이 맞지 않았습니다. 그래서 현재 사용하는 라이브러리는 유지하돼 npm의 관리될 수 있는 라이브러리는 package.json을 통해 관리하려 합니다.

**ES6 도입**<br> IE 하위 버전까지 고려 안해도 되는 고마운 모바일 프로젝트이기 때문에 핫하디 핫한! 앞으로 자바스크립트의 새로운 표준이 될 `ES6`를 사용하려 합니다. 아직 완벽히 모든 브라우저를 지원하지 않기 때문에 `Babel Transformer`를 사용할 것이고, Babel을 사용하여 쉽게 Bundling 할 수 있는 `Webpack`을 사용하려 합니다.

***현재 패키지 구조***

```
/src/main
  /java
    ...
  /resources
    /static
      /js
        ..
        page1.coffee
        page2.coffee
        page3.coffee
        common1.coffee
        common2.coffee
        common3.coffee
        /lib
          ..
          (구)lib1.js
          (구)lib2.js
          jquery.min.js
          moment.min.js
          underscore-min.js
```

-	page[1-3].coffee : 각 Page에서 사용하는 스크립트
-	common[1-3].coffee : 각 page 스크립트에서 사용될 공통 스크립트
-	lib[1-2].js : 가지고 있는 js 외에는 출처가 불분명한... 외부 스크립트
-	jquery, moment, undersocre : npm으로도 관리가 가능한 외부 스크립트

### Webpack

> webpack.config.js

```javascript
var path = require('path');
var webpack = require('webpack');
var STATIC = path.join(__dirname, 'src/main/resources/static');

module.exports = {
    entry: {
        page1: path.join(STATIC, 'js/page1.js'),
        page2: path.join(STATIC, 'js/page2.js'),
        page3: path.join(STATIC, 'js/page3.js')
    },
    output: {
        path: path.join(STATIC, 'js/dist'),
        filename: '[name].min.js'
    },
    module: {
        loaders: [
            {
                test: /\.js$/,
                loader: 'babel-loader',
                query:{
                    presets: ['es2015']
                }
            }
        ]
    },
    plugins: [
        new webpack.optimize.UglifyJsPlugin({
            compressor: {
                warnings: false,
            },
        }),
        new webpack.optimize.OccurenceOrderPlugin()
    ]
}
```

### GULP

### ES6
