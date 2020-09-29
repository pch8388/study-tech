# 클로저
- 클로저는 함수가 선언되었을 당시의 lexical environment 와의 조합
- 해당 함수의 scope 가 아닌 상위 scope 의 변수를 자유변수라고 하는 데, 이 자유변수를 사용하는 함수를 클로저라고 함
- 클로저가 사용하는 자유변수의 실행컨텍스트가 종료되어도 변수객체 영역은 gc 의 대상이 되지 않고 메모리상에 남아 참조할 수 있게 됨
```javascript
function foo() {
    let num = 1;
    return () => num+1;
}

var bar = foo();
console.log(bar());  // 2
```
- 캡슐화, 은닉화 측면에서 사용할 수도 있음
- 너무 많이 쓰면 메모리 낭비등의 문제 발생
