# bundler

## webpack
- 규모가 있거나 세세한 디테일이 필요한 bundler
1. setting
  ```cli 
    mkdir webpack-template-basic
    cd webpack-template-basic

    npm init -y
    npm i -D webpack webpack-cli webpack-dev-server@next 
  ```
  - 현재 설치된 모듈 버전
    * "webpack": "^5.88.0",
    * "webpack-cli": "^5.1.4",
    * "webpack-dev-server": "^4.0.0-rc.1"
  - package.json setting

  ```package.json
    "dev": "webpack-dev-server --mode development",
    "build": "webpack --mode production"
  ```
    * "dev" : 개발용 
    * "build" : 배포용 

  - ./index.html 생성 및 reset.css 세팅
  - ./js/main.js 세팅
  - ./webpack.config.js 파일 생성
  - ./static 폴더 생성
    * ./static/images/logo.png
    * ./static/favicon.io 

2. entry, output
  - import & export를 담당
  - webpack.config.js
  ```webpack.config.js
    // 1 import
    const path = require('path')

    // 2 export
    module.exports = {
      // 파일을 읽어들이기 시작하는 진입점 설정
      entry: './js/main.js',

      // 결과물(번들)을 반환하는 설정
      output: {
        //path: path.resolve(__dirname, 'public'),
        //filename: 'main.js',
        clean: true
      }
    }
  ```
    * 1 import
      - nodejs의 path라는 모듈을 변수 저장
      - 항상 절대 경로 유지
    * 2 export
      - entry : 파일을 읽어들이는 시작점
        * 경로를 설정 
        * 설정된 경로의 파일을 output의 filename에 자동 반영된다.
        * 진입점을 여러가지로 지정 가능
      - output : 반환하는 설정
        * path 입력 부분은 주석처리되도 자동으로 설정된다. 만약 원하는 위치가 다르다면 주석 해제 후 입력
        * 항상 절대 경로를 사용 
        * filename의 입력부분도 entry의 부분을 참조해 해당파일을 export 한다
        * clean : true를 설정한다면 항상 초기화하여 지정된 파일만 남게 한다
3. plugins
  - index.html 을 /dist에 포함시키기 위한 플러그인이 필요
  ```terminal
    npm i -D html-webpack-plugin
  ```
    * html-webpack-plugin 개발용 설치

  ```webpack.config.js
    const HtmlPlugin = require('html-webpack-plugin')

    plugins: [
      new HtmlPlugin({
        template: './index.html'
      })
    ] 
    devServer: {
      host: 'localhost'
    }
  ```
    * 번들링 후 결과물의 처리방식을 플러그인에서 설정
    * localhost 설정

4. 정적 파일 연결
  - ./static 설정
  ```console
    npm i -D copy-webpack-plugin
  ```
  ```webpack.config.js
    const CopyPlugin = require('copy-webpack-plugin')

    plugins: [
      new HtmlPlugin({
        template: './index.html'
      }),
      new CopyPlugin({
        patterns: [
          { from: 'static}
        ]
      })
    ] 
  ```
    * 새롭게 가져온 CopyPlugin을 plugin에 주입
    * pattern이 [] 배열인 이유는 여러 디렉토리를 지정해 주기 위함
  - ./static 폴더의 디렉토리 구조를 그대로 /dist로 옮긴다
    * 즉 ./index.html의 기준에서 img src의 주소를 ./images/logo.png로 설정해야 되는것

5. module
  - main.js의 내부에 css파일을 import 시키는 방법
    ```js
      import '../css/main.css'
    ```

    ```console
      npm i -D css-loader style-loader
    ```

    ```webpack.config.js
      module: {
        rules: [
          {
            test: /\.css$/,
            use: [
              'style-loader',
              'css-loader'
            ]
          }
        ]
      }
    ```
    * css-loader가 읽히고 style-loader를 읽힌다
    * 정규식 표현으로 .css로 끝나는 파일을 찾는것
6. SCSS
  - npm i -D sass-loader sass 
    * 터미널 설치
    * sass를 설치해서 컴파일 sass-loader를 설치해서 config에 명시하기 위함
  - ./js/main.js 수정
    * import '../scss/main.scss'
  - webpack.config.js
    * test: /\.s?css$/ 정규식 수정
    * module에 'sass-loader' 명시

7. autoprefixer(PostCSS)
  - ./scss/main.scss
    * h1 { display: flex; } 추가
  - terminal
    * npm i -D postscss autoprefixer postcss-loader
  - webpack.config.js > module > rules
    * 'postcss-loader' 추가 -> sass-loader 보다 위 명시
  - package.json
    * "browserslist": [ "> 1%", "last 2 versions" ]
    * 전세계 브라우저 1퍼센트 사용 이상
    * 마지막 버전 2개 전까지
  - ./.postcssrc.js 파일 추가
    ```js
      module.exports = {
        plugins: [
          require('autoprefixer')
        ]
      }
    ```
    * 해당 파일에 추가
  - index.html에 필요없어진 link 태그의 css를 제거하면 console error 메세지가 사라진다 얏호

8. babel
  - npm i -D @babel/core @babel/preset-env @babel/plugin-transform-runtime
  - .babelrc.js 생성
    ```js
      module.exports = {
        presets: ['@babel/preset-env'],
        plugins: [
          ['@babel/plugin-transform-runtime']
        ]
      }
    ```
    * presets -> javascript의 기능을 한번에 지원
    * plugins -> 비동기 처리를 위함, 2차원 배열
  - webpack.config.js
    ```js
      module: {
        rules: [
          {
            test: /\.s?css$/,
            use: [
              'style-loader',
              'css-loader',
              'postcss-loader',
              'sass-loader'
            ]
          },
          {
            test: /\.js$/,
            use: [
              'babel-loader'
            ]
          }
        ]
      },
    ```
    * rules의 두번째 배열로 추가
    * .js로 끝나는 정규식 표현
    * 'babel-loader' 명시
  - npm i -D babel-loader 설치

9. netlify 배포
  - webpack으로 배포가 이뤄지는 구조는 netlify 배포의 방식이 다름
  - 로그인 후 add new site
  - github 선택 후 작성된 repository 까지 접근
  - Basic build setting가 중요함
    * build command > npm run build
    * publish directory > dist/ 
    * 위와 같이 세팅
  - 기존에 학습하던 parcel-template-basic 도 명령어와 디렉토리 세팅이 달라지지 않았다면 동일한 방법으로 사이트 추가

10. NPX, Degit
  - npx degit [hifiv12/webpack-template-basic] [webpack-template-test]
    * 터미널에 위치한 경로에서 github에 있는 레포지토리를 다운로드하는 기능
    * npx는 node.js 설치했기에 사용 가능
    * 원격의 장소로 부터 현재 위치한 로컬 위치에 다운로드
    * [] [] 첫번째 구문은 다운로드할 계정의 레포지토리 위치, 두번째 구문은 다운로드 하면서 지정할 이름
  - git clone vs npx degit 
    * git clone은 버전을 계승하여 버전 업을 하는것
    * npx degit은 템플릿을 받는 것 > 그래서 처음버전부터 시작한다는 의미