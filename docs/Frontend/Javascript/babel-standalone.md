## 개요
- 바벨은 브라우저에서 지원되지 않는 문법을 지원하는 문법으로 변경 시켜주는 도구
- ie 에서는 es6 문법 지원이 되지 않는 부분이 많아서 사용하게 됨
- 바벨이나 웹팩 등의 도구를 이용하여 개발시에 패키징하여 배포하는 것이 요즘의 추세이지만, 현재 개발팀 여건상 stand alone 모드로 사용

## 사용법
```html
<!-- 추가 -->
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<!-- 사용시 타입을 text/babel 로 -->
<script type="text/babel">
  const a = () => console.log('test ie');
  a();
</script>
```

[바벨 공식페이지](https://babeljs.io/setup#installation)
