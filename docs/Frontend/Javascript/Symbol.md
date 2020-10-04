# 특징
- 모든 심볼을 Symbol.prototype 을 상속
- primitive type
- 기본적인 열거대상에서 제외(iterable 하게 사용하지 못함)
- 기본타입이지만 암묵적 형변환을 허용하지 않음
- 항상 고유한 값
    ```javascript
    const foo = Symbol('a');
    const bar = Symbol('a');
    console.log(foo === bar); // false
    console.log(foo == bar);  // false

    const bar2 = bar;
    console.log(bar === bar2); // true
    ```
- Symbol.for 를 사용하면 전역으로 존재하는 global symbol table 의 목록을 참조
    ```javascript
    const foo = Symbol.for('a');
    const bar = Symbol.for('a');
    console.log(foo === bar); // true
    ```

# 사용
- 항상 고유한 값이기 때문에 Symbol을 이용하여 private 한 property 를 만들 수 있음
    ```javascript
    const x = () => {
        const k = Symbol('a');
        return {
            [k]: 10
        };
    };

    const y = x();
    console.log(y);  // {Symbol(a) : 10}
    ```
    - Symbol 은 항상 고유한 값이기 때문에 위의 예제에서 y 의 Symbol(a) 에 접근할 수 있는 방법이 없다
    - public 하게 공개 하려면 Symbol 값 자체를 외부로 노출 시킬수 있다.
    ```javascript
    const x = () => {
        const k = Symbol('a');
        return {
            [k]: 10,
            z: k
        };
    };

    const y = x();
    console.log(y[y.z]);  // 10
    ```

# 반복(iterator) 심볼
- Symbol.iterator
- Symbol.asyncIterator

# 정규표현식 심볼
- Symbol.match
- Symbol.replace
- Symbol.search
- Symbol.split