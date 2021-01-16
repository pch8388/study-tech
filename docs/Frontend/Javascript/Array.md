## ES 6 에서 추가된 Array 의 함수
- Array.from() : 배열이 아닌 것을 배열로 만듬
    ```javascript
    Array.from('Zero'); // ['Z','e','r','o']
    ```

- Array.of(인자) : 인자를 요소로 갖는 배열을 만듬

    ```javascript
    Array.of('Zero', 'Nero', 'Hero'); // ["Zero", "Nero", "Hero"]
    ```

- 배열.fill(값, 시작위치, 끝위치) : 배열의 요소를 특정 값으로 채운다

    ```javascript
    [1, 3, 5, 7, 9].fill(4); // [4, 4, 4, 4, 4]
    [1, 3, 5, 7, 9].fill(4, 1, 3); // [1, 4, 4, 7, 9]
    ```

- 배열.find(함수) : 배열 안의 요소를 찾는다. 조건에 만족하는 첫번째 요소 반환

    ```javascript
    [1, 2, 3, 4, 5].find(function(number) {
      return number % 2 === 0;
    }); // 2
    [{ name: 'Zero' }, { name: 'Nero' }, { name: 'Hero' }].find(function(person) {
      return person.name === 'Zero';
    }); // { name: 'Zero' }
    ```

- 배열.findIndex(함수) : 배열안의 요소의 인덱스 반환

    ```javascript
    [1, 2, 3, 4, 5].findIndex(function(number) {
      return number % 2 === 0;
    }); // 1
    [{ name: 'Zero' }, { name: 'Nero' }, { name: 'Hero' }].findIndex(function(person) {
      return person.name === 'Zero';
    }); // 0
    ```

- 배열.copyWithin(목표, 시작점, 끝점) : 목표의 위치에 시작~끝까지의 배열을 덮어씌움

    ```javascript
    [1, 2, 3, 4, 5].copyWithin(1, 2); // [1, 3, 4, 5, 5]
    [1, 2, 3, 4, 5].copyWithin(1, 2, 3); // [1, 3, 3, 4, 5]
    ```