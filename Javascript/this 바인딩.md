1. 객체의 메서드 호출시 메서드를 호출한 객체로 바인딩

```javascript
const foo = {
    getName: function() {
        return `foo this ==> ${this.name}`;
    }
};

foo.name = 'test foo';
console.log(foo.getName());  // foo this ==> test foo
```

2. 함수를 호출할 때 전역으로 바인딩

```javascript
function bar(){
    return this.name;
}

window.name = 'window';
bar.name = 'bar';
console.log(bar());   // window
```

3. 생성자 함수를 호출할 때 생성된 함수
  - 생성자 함수 동작 방식
    1. 빈 객체 생성(이 객체에 this 바인딩)
      => 생성자 함수의 prototype 프로퍼티가 가리키는 객체를 자신의 프로토타입 객체로 설정
    2. this 를 통한 프로퍼티 생성
    3. 생성된 객체 리턴

```javascript
function Foo(name) {
    this.name = name;
}

const fooObj = new Foo('foo');
console.log(fooObj.name);  // foo
```
