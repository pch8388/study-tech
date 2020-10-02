# 개요
- 의존관계로 엮인 모듈들을 묶어 하나의 번들링 파일로 만들어 주는 도구

# 설치 및 실행
- 프로젝트 initialize
```npm init```
- webpack 설치
```
npm install -D webpack webpack-cli
// cli : webpack 을 터미널 명령어로 사용
// -D 옵션으로 개발환경으로 설정
```
- package.json 에 devDependecies 에 webpack 정보가 추가된 것을 확인할 수 있음(-D 옵션으로 개발환경에서 사용한다고 한 것)

## 실행 옵션
- --mode : 어떤 모드로 실행할 것인지
- --entry : 엔트리 포인트 지정
- --output : 결과 파일 저장 위치

## entry / output
- webpack.config.js : 기본설정 파일이름
```javascript
const path = require('path');

module.experts = {
  mode: 'development',
  entry: { // 엔트리 포인트를 여러개 둘 수도 있음 (SPA 가 아닌 멀티페이지 애플리케이션에 적합)
    main: './src/app.js'
  },
  output: {
    path: path.resolve('./dist'),
    filename: '[name].js'   // entiry 의 key 로 filename 을 지정한다
  }
}
```
- package.json
```javascript
"scripts": {
  "build": "webpack"    // 이부분을 추가해준다(빌드를 웹팩으로 하도록 지정)
}
```
- 빌드된 결과물(output) 을 html 에서 로드
```html
<script src="dist/main.js"></script>
```

## loader
- 웹팩이 자바스크립트 파일이 아닌 리소스(HTML, CSS, Images, fonts...)를 모듈로 포함시킬 수 있도록 도와줌
- 사용할 로더를 설치해야 함
```
npm install style-loader css-loader file-loader url-loader
```
- webpack.config.js
```javascript
module: {
    rules: [   // rule 을 정의
      {
        test: /\.css$/,   // 정규식으로 로더를 사용할 형식 정의
        use: [            // 사용할 로더들
          'style-loader',   // 로더는 오른쪽에서 왼쪽 순서로 적용됨(아래에서 위), 로더의 순서를 규칙에 맞춰야 하는 경우 있음
          'css-loader'
        ]
      },
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'url-loader',
        options: {      
          publicPath: './dist/',
          name: '[name].[ext]?[hash]',
          limit: 10000    // 10kb 미만은 url 로더 사용
        }
      }
    ]
  }
```
