# 특징
- function 키워드 대신 사용 가능
- this 가 lexical this : 언제나 상위 스코프의 this 를 가리킴
    - bind, call, apply 등으로 this 변경 불가능

# 주의점 
1. 객체 내의 메소드를 화살표 함수로 정의하면 this 가 의도와 다르게 적용될 수 있다
    ```javascript
    const book = {
        name: 'Javascript',
        getName: () => console.log(`book name is ${this.name}`);
    };

    book.getName();   // book name is undefined => this 가 window
    ```
    - 아래와 같은 형태로 표현하는 것이 좋음
    ```javascript
    const book = {
        name: 'Javascript',
        getName: () {
            console.log(`book name is ${this.name}`);
        }
    };

    book.getName();   // book name is Javascript
    ```

2. prototype 정의
    ```javascript
    const book = {
        name: 'Javascript'
    };

    Object.prototype.getName = () => console.log(`book name is ${this.name}`);

    book.getName();   // book name is undefined => this 가 window
    ```
    - 아래와 같은 형태로 표현하는 것이 좋음
    ```javascript
    const book = {
        name: 'Javascript'
    };

    Object.prototype.getName = function () {
        console.log(`book name is ${this.name}`);
    };

    book.getName();   // book name is Javascript
    ```

3. 생성자 함수 : 화살표 함수는 생성자 함수로 사용 불가
    - 생성자 함수는 prototype 프로퍼티로 constructor 를 가져야 하는 데, 화살표 함수는 이것이 없음

4. addEventListener 콜백함수
    ```javascript
    const button = document.querySelector('button');
    button.addEventListener('click', () => {
        this.innerHtml = 'clicked';     // this => window
    });
    ```
    - 아래와 같은 형태로 표현하는 것이 좋음
    ```javascript
    const button = document.querySelector('button');
    button.addEventListener('click', ({target}) => {
        target.innerHtml = 'clicked';
    });
    ```