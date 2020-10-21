# 설치
1. node.js 설치
https://poiemaweb.com/nodejs-basics
2. vue 설치
```
npm install vue
```

# vue-cli 를 통한 프로젝트 생성
1. vue-cli 설치
```shell
npm install -g @vue/cli
# g 는 global 설치 옵션
```
2. vue create 프로젝트이름
```shell
vue create front-end
```
3. vue-cli 옵션화면에서 필요한 옵션을 선택
![vue-cli 옵션이미지](images/vue-cli-option.JPG)

## vue-cli 를 통한 개발서버 실행
- webpack-dev-server 기반
- hot module replacement 기능 활용가능 (서버 재시작 없이 저장만하면 적용됨)
```shell
vue-cli-service serve
# package.json 설정을 통해 npm run serve 와 같이 변경된 명령어 사용 가능
```

# 부트스트랩 설정
## 설치 : 사용할 디렉터리
```shell
npm install jquery popper.js bootstrap --save
````

## 웹팩 설정
- 웹팩에 새로운 진입점을 만들어서 모든 서드 파티 스타일을 하나의 css 파일로 묶음
```javascript
module.experts = {
    ...
    configureWebpack: {
        entry: {
            app: './src/main.js',
            style: [
            'bootstrap/dist/css/bootstrap.min.css'
            ]
        }
    }
};
```