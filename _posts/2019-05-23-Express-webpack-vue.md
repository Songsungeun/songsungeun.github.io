---
layout: post
title:  "Express-webpack App 개발 및 운영 빌드"
date:   2019-05-23 10:19:42 +0900
categories: express webpack vue
background: '/img/posts/vue-express.png'
---
Express App을 Webpack이용하여 Dev와 Prod 빌드를 해보자.
  
현재 프로젝트 특성상 webpack-dev-server를 이용해서 hot-reloading을 할 수가 없는 상황이어서
(proxytable 사용해서 proxy로 Express쪽에 전달을 하면 안되는 상황이다.ㅜㅜ)
방법을 찾던 중 유용한 블로그가 있어서 참고해서 D-Log를 작성한다.
이 글은 [Binyamin Midium][binyamin-medium]를 참고해서 작성했음을 알린다.

## 1.Express
정리할겸 처음부터 시작해보자.  
폴더를 만들고 `package.json`을 만들자

```
mkdir express-webpack  
cd express-webpack  
npm init -y
```

`express`를 설치하고
```
npm i express
```

`express`구동을 위해 `package.json`의 `start` 스크립트를 변경한다.

```
"scripts": {
    "start": "node ./server.js"
}
```

자 이제 server.js로 구동하라고 했으니,  
root에 server.js를 하나 작성해주자  

``` javascript
const path = require('path')
const express = require('express')
const app = express(),
            DIST_DIR = __dirname,
            HTML_FILE = path.join(DIST_DIR, 'index.html')
app.use(express.static(DIST_DIR))
app.get('*', (req, res) => {
    res.sendFile(HTML_FILE)
})
const PORT = process.env.PORT || 8080
app.listen(PORT, () => {
    console.log(`App listening to ${PORT}....`)
    console.log('Press Ctrl+C to quit.')
})
```
그리고 간단한 html 파일을 만들어보자
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Express and Webpack App</title>
    <link rel="shortcut icon" href="#">
</head>
<body>
    <h1>Expack</h1>
    <p class="description">Express and Webpack Boilerplate App</p>
</body>
</html>
```

이제 `npm start`를 이용하여 실행해보자  
<http://localhost:8080>에 진입하면 페이지가 보일것이다.

###### NOTE: 브라우저 콘솔창에 아무 오류도 없어야 한다.  
  
    
## 2.Webpack
이제 `Webpack`을 설치해보자.
cmd에서 webpack을 사용하기 위해 `cli`도 설치하고, 배포시 `node_modules`를 무시하기위해 `webpack-node-externals`도 설치하자
```
npm i -D webpack webpack-cli webpack-node-externals
```
그리고 transpile을 위해 Babel을 설치하자
```
npm i -D @babel/core @babel/preset-env babel-loader
```

추가로 index.html파일을 빌드하기 위해 **html-loader** 와 **html-webpack-plugin** 을 설치하고, import시켜야 한다.
```
npm i -D html-loader html-webpack-plugin
```
이제 webpack 빌드 설정을 위해 웹팩 설정 파일이 필요하다.  
**webpack.config.js** 파일을 만들고 code를 작성해보자.

``` javascript
const path = require('path')
const webpack = require('webpack')
const nodeExternals = require('webpack-node-externals')
const HtmlWebPackPlugin = require("html-webpack-plugin")
module.exports = {
  entry: {
    server: './server.js',
  },
  output: {
    path: path.join(__dirname, 'dist'),
    publicPath: '/',
    filename: '[name].js'
  },
  target: 'node',
  node: {
    // Need this when working with express, otherwise the build fails
    __dirname: false,   // if you don't put this is, __dirname
    __filename: false,  // and __filename return blank or /
  },
  externals: [nodeExternals()], // Need this to avoid error when working with Express
  module: {
    rules: [
      {
        // Transpiles ES6-8 into ES5
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        // Loads the javacript into html template provided.
        // Entry point is set below in HtmlWebPackPlugin in Plugins 
        test: /\.html$/,
        use: [{loader: "html-loader"}]
      }
    ]
  },
  plugins: [
    new HtmlWebPackPlugin({
      template: "./index.html",
      filename: "./index.html",
      excludeChunks: [ 'server' ]
    })
  ]
}
```

html에 서버 파일이 포함될 필요가 없기 때문에  
**excludeChunks**를 이용해서 **server** 파일을 제외한다.  

<!-- ```javascript
function func() {
    var a = 'AAA';
    return a;
}
``` -->

기존 server.js에서는 **require** 모듈을 이용하여 **path**와 **express**를 불러왔으나,
이제 Babel을 설치하였으니 **import**구문으로 변경해보자.  
###### Note: **node**에서 **import**와 **export**구문을 아직 Default로 지원하지 않고 있어 Babel을 사용하여야 한다.  

``` javascript
const path = require('path')
const express = require('express')
```
**Change to**
``` javascript
import path from 'path'
import express from 'express'
```
  
바벨 설정을 위해 **.babelrc** 파일을 root에 만들자.
``` javascript
{
  "presets": ["@babel/preset-env"]
}
```
이제 **package.json**에 build용 스크립트를 만들어주자.
``` javascript
"scripts": {
  "build": "rm -rf dist && webpack --mode development",
  "start": "node ./dist/server.js"
},
```
빌드시 마다 기존 dist폴더를 지우고 새로 dist폴더를 만들기 위함이지만,
rm -rf가 리눅스 명령어이기때문에 윈도우에서는 해당되지 않는다.
윈도우의 경우에는
``` 
"build": "del /f /s /q dist && webpack --mode development",
```
위와 같이 실행하면 된다.  
  
  start 명령어는 이제 기존의 **./server.js** 를 실행하는 것이 아닌, Build된 **./dist/server.js**로 변경한다.    
  
**npm run build** 후 **npm start**하여 이상없이 서버 실행되는 것을 확인한다.
  

## 3.CSS와 js 함수 추가  

이제 CSS 스타일과 js, image등을 추가해보자  
이에 앞서, 우리는 Webpack 설정 파일을 2개로 분할해보자.  
나중에는 3개파일이 될거다.  
우선 **webpack.server.config.js**와 **webpack.config.js** 파일로 분리해보자  
- **webpack.server.config.js** = 서버코드 번들링  
- **webpack.config.js** = 이외 코드 번들링  
  
이후에는 이것들도 각각 Dev용 Prod용으로 다시 분할 것이다.  
  
이제 필요한 의존모듈을 설치하자  
```
npm i -D css-loader file-loader style-loader
```
  
후에 Directory구조는 아래와 같이 만들어질 것이다.  
```
.babelrc
.git
.gitignore
README.md
dist
node_modules
package-lock.json
package.json
webpack.config.js
webpack.server.config.js
src
    index.js
    html
        index.html
    css
        style.css
    js
        index.js
    img
        awful-selfie.jpg
    server
        server.js
```
  
**package.json** 파일도 수정한다.  
```
"scripts": {
  "build": "rm -rf dist && webpack --mode development --config webpack.server.config.js && webpack --mode development",
  "start": "node ./dist/server.js"
},
```
위에서 언급했듯이 윈도우는 del /f /s /q이다.  
  
그리고 **./src/html/index.html** 을 작성하자  
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Express and Webpack App</title>
    <link rel="shortcut icon" href="#">
</head>
<body>
    <h1>Expack</h1>
    <p class="description">Express and Webpack Boilerplate App</p>
    <div class="awful-selfie"></div>
</body>
</html>
```
  
**./src/css/style.css**도 작성한다.  
``` css
h1, h2, h3, h4, h5, p {
  font-family: helvetica;
  color: #3e3e3e;
}
.description {
  font-size: 14px;
  color: #9e9e9e;
}
.awful-selfie{
  background: url(../img/bg.jpg);
  width: 300px;
  height: 300px;
  background-size: 100% auto;
  background-repeat: no-repeat;
}
```
img 폴더에 그림파일을 아무거나 넣어주고 이름을 맞춰주자.  
  

이제 **./src/index.js** 를 작성하여 import및 스타일이 잘 동작하는지 기본적인 확인을 해보자.  
``` javascript
import logMessage from './js/logger'
import './css/style.css'
// Log message to console
logMessage('Welcome to Expack!')
```
물론 **./src/js/logger.js**도 생성해야 한다.  
``` javascript
const logMessage = msg => console.log(msg)
export default logMessage
```
단순히 js 함수 확인하기 위한 로그 함수를 하나 만들었다.  
  
이제 기존 root에 있던 **server.js**를 **./src/server.**로 옮겨보자  
마지막으로 Webpack 설정 파일을 백엔드용과 프론트용으로 분할한다.  
**./webpack.server.config.js**
``` javascript
const path = require('path')
const webpack = require('webpack')
const nodeExternals = require('webpack-node-externals')
module.exports = {
  entry: {
    server: './src/server/server.js',
  },
  output: {
    path: path.join(__dirname, 'dist'),
    publicPath: '/',
    filename: '[name].js'
  },
  target: 'node',
  node: {
    // Need this when working with express, otherwise the build fails
    __dirname: false,   // if you don't put this is, __dirname
    __filename: false,  // and __filename return blank or /
  },
  externals: [nodeExternals()], // Need this to avoid error when working with Express
  module: {
    rules: [
      {
        // Transpiles ES6-8 into ES5
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  }
}
```
그리고 **webpack.config.js**이다.  
``` javascript
const path = require("path")
const webpack = require('webpack')
const HtmlWebPackPlugin = require("html-webpack-plugin")
module.exports = {
  entry: {
    main: './src/index.js'
  },
  output: {
    path: path.join(__dirname, 'dist'),
    publicPath: '/',
    filename: '[name].js'
  },
  target: 'web',
  devtool: 'source-map',
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader",
      },
      {
        // Loads the javacript into html template provided.
        // Entry point is set below in HtmlWebPackPlugin in Plugins 
        test: /\.html$/,
        use: [
          {
            loader: "html-loader",
            //options: { minimize: true }
          }
        ]
      },
      {
        test: /\.css$/,
        use: [ 'style-loader', 'css-loader' ]
      },
      {
       test: /\.(png|svg|jpg|gif)$/,
       use: ['file-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebPackPlugin({
      template: "./src/html/index.html",
      filename: "./index.html",
      excludeChunks: [ 'server' ]
    })
  ]
}
```
여기서 중요한것은 **target: 'web'**이다.  
server쪽 설정파일은 **target: 'node'**이고, 프론트쪽은 **target: 'web'**인 것을 잘 확인하여야 한다.  
  
기존 server.js 파일을 ./src/server/server.js로 옮긴 후 **npm run build**와 **npm start**를 해보자  
###### NOTE: 브라우저 콘솔창에 로그 찍힌 것도 확인하여야 한다.    
  
  
## 4. Dev와 Prod용 빌드 분할  
여기까지 기초적인 App의 설정이 완료되었다. 이제는 개발용 빌드를 작성하여 개발을 좀 더 편하게 해보자  
개발용 빌드에만 **HMR**을 적용할 것이다.  
  
자 우선, **webpack.config.js** 파일을 **webpack.dev.config.js** 와 **webpack.pord.config.js**로 분할하자.  
또한 **./src/server/server.js**도 **server-dev.js**와 **server-prod.js**로 분할하자.  
  
그러면 Directory 구조는 다음과 같이 된다.  
```
src
    server
        server-dev.js
        server-prod.js
webpack.dev.config.js
webpack.prod.config.js
webpack.server.config.js
```
  
이제 설정파일들을 각각 분할하였으니, **package.json** scripts 또한 변경한다.  
```
  "scripts": {
    "buildDev": "del /f /s /q dist && webpack --mode development --config webpack.server.config.js && webpack --mode development --config webpack.dev.config.js",
    "buildProd": "del /f /s /q dist && webpack --mode production --config webpack.server.config.js && webpack --mode production --config webpack.prod.config.js",
    "start": "node ./dist/server.js"
  },
```
  
**webpack.server.config.js** 파일을 변경해보자.  
``` javascript
const path = require('path')
const webpack = require('webpack')
const nodeExternals = require('webpack-node-externals')
module.exports = (env, argv) => {
  const SERVER_PATH = (argv.mode === 'production') ?
    './src/server/server-prod.js' :
    './src/server/server-dev.js'
return ({
    entry: {
      server: SERVER_PATH,
    },
    output: {
      path: path.join(__dirname, 'dist'),
      publicPath: '/',
      filename: '[name].js'
    },
    target: 'node',
    node: {
      // Need this when working with express, otherwise the build fails
      __dirname: false,   // if you don't put this is, __dirname
      __filename: false,  // and __filename return blank or /
    },
    externals: [nodeExternals()], // Need this to avoid error when working with Express
    module: {
      rules: [
        {
          // Transpiles ES6-8 into ES5
          test: /\.js$/,
          exclude: /node_modules/,
          use: {
            loader: "babel-loader"
          }
        }
      ]
    }
  })
}
```
mode에 따라 entry에 지정할 server 파일을 변경해주었다.  

css minifier와 이미지 base64 인코딩을 위해 모듈을 설치해주자
```
npm i -D mini-css-extract-plugin uglifyjs-webpack-plugin optimize-css-assets-webpack-plugin url-loader
```  
  
이제 **webpack.prod.config.js** 를 수정해주자.  
``` javascript
const path = require("path")
const HtmlWebPackPlugin = require("html-webpack-plugin")
const MiniCssExtractPlugin = require("mini-css-extract-plugin")
const UglifyJsPlugin = require("uglifyjs-webpack-plugin")
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");
module.exports = {
  entry: {
    main: './src/index.js'
  },
  output: {
    path: path.join(__dirname, 'dist'),
    publicPath: '/',
    filename: '[name].js'
  },
  target: 'web',
  devtool: 'source-map',
  // Webpack 4 does not have a CSS minifier, although
  // Webpack 5 will likely come with one
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        cache: true,
        parallel: true,
        sourceMap: true // set to true if you want JS source maps
      }),
      new OptimizeCSSAssetsPlugin({})
    ]
  },
  module: {
    rules: [
      {
        // Transpiles ES6-8 into ES5
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        // Loads the javacript into html template provided.
        // Entry point is set below in HtmlWebPackPlugin in Plugins 
        test: /\.html$/,
        use: [
          {
            loader: "html-loader",
            options: { minimize: true }
          }
        ]
      },
      {
        // Loads images into CSS and Javascript files
        test: /\.jpg$/,
        use: [{loader: "url-loader"}]
      },
      {
        // Loads CSS into a file when you import it via Javascript
        // Rules are set in MiniCssExtractPlugin
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      },
    ]
  },
  plugins: [
    new HtmlWebPackPlugin({
      template: "./src/html/index.html",
      filename: "./index.html"
    }),
    new MiniCssExtractPlugin({
      filename: "[name].css",
      chunkFilename: "[id].css"
    })
  ]
}
```
**webpack.dev.config.js**는 이전과 같다.  
이제 둘의 차이점을 확인해보자.  
각각 dev용 prod용 빌드 후 브라우저에서 개발자도구를 이용하여 이미지의 css를 확인해보자.  
Prod용인경우 인코딩 되어 있는것을 확인할 수 있다.  
  
## 5. Webpack Dev Middleware  
이제 정말 중요한게 남았다.... 내가 webpack-dev-server를 이용할 수 없는 상황이어서 가장 골치아팠던 것..  
매번 npm run buildDev를 하면서 개발을 할 수는 없으니 Reloading이 필요한 상황이다.  
  
Webpack Dev Middleware (WDM)은 파일 변경을 감시하고 변경시마다 Webpack 빌드를 실행한다.  
모든 변경 사항은 파일이 쓰이지 않고 메모리에서 처리한다.  
아직까지 **HMR**은 아니지만, 우선적으로 필요한 단계이니 진행하도록 한다.  
  
자, 이제 의존모듈을 설치하자.  
  
```
npm i -D webpack-dev-middleware
```
  
**webpack.dev.config.js** 파일에 mode와 plugin을 추가해준다.
  
```
output: {
    path: path.join(__dirname, 'dist'),
    publicPath: '/',
    filename: '[name].js'
  },
  mode: 'development',
  target: 'web',
  devtool: 'source-map',
```

```
plugins: [
    new HtmlWebPackPlugin({
      template: "./src/html/index.html",
      filename: "./index.html",
      excludeChunks: [ 'server' ]
    }),
    new webpack.NoEmitOnErrorsPlugin()
  ]
```
이제, **./src/server/server-dev.js** 파일을 수정해주자.  

``` javascript
import path from 'path'
import express from 'express'
import webpack from 'webpack'
import webpackDevMiddleware from 'webpack-dev-middleware'
import config from '../../webpack.dev.config.js'
const app = express(),
            DIST_DIR = __dirname,
            HTML_FILE = path.join(DIST_DIR, 'index.html'),
            compiler = webpack(config)
app.use(webpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath
}))
app.get('*', (req, res, next) => {
  compiler.outputFileSystem.readFile(HTML_FILE, (err, result) => {
  if (err) {
    return next(err)
  }
  res.set('content-type', 'text/html')
  res.send(result)
  res.end()
  })
})
const PORT = process.env.PORT || 8080
app.listen(PORT, () => {
    console.log(`App listening to ${PORT}....`)
    console.log('Press Ctrl+C to quit.')
})
```
  
이제 **npm run buildDev** 후 **npm start** 해서 서버를 띄워보자.
이제 js파일이나 html파일을 수정 후 리프레쉬하면 변경된 것을 확인할 수 있다.  
(변경 후 저장시 터미널에서도 리빌드 되는 것이 확인된다.)  
  
  
  
원본인 [Binyamin Midium][binyamin-medium]에는 hot-reloading, lint설정과 test부분까지 진행되고 있다.  
해당 정보는 [Binyamin Midium][binyamin-medium]에서 확인하기 바란다.


[binyamin-medium]: https://medium.com/@binyamin/creating-a-node-express-webpack-app-with-dev-and-prod-builds-a4962ce51334

