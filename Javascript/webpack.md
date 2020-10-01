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

## 설정파일
- webpack.config.js : 기본설정 파일이름
```javascript
const path = require('path');

module.experts = {
  mode: 'development',
  entry: { // 엔트리 포인트를 여러개 둘 수도 있음
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
