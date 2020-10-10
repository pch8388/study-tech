# 호출 스케줄링
- 일정 시간이 지난후 원하는 함수를 호출할 수 있도록 함
- setTiemout : 일정 시간이 지난 후 함수 실행
- setInterval : 일정 시간 간격을 두고 함수 실행
- 대부분의 경우에 setTimeout 을 쓰는 것이 맞다고 생각됨
    - setInterval 은 실행시 지연 간격을 보장하지 않음
    - setTimeout 을 중첩실행하여 setInterval 을 대처할 수 있음
- clearTimeout : 스케줄링 취소 함수
    ```javascript
    let timerId = setTimeout(() => console.log('test'), 1000);
    clearTimeout(timerId);  // setTimeout 안의 함수가 실행되지 않음
    ```

## 중첩 setTimeout
- setInterval 을 대처할 수 있음
    ```javascript
    let timeId = setTimeout(function go() {
        console.log('go');
        timeId = setTimeout(go, 1000); // 여기의 실행이 종료되면 다음 호출을 스케줄링 함
    });
    ```
> setTimeout / setInterval 에 함수를 넘기면, 함수에 대한 내부 참조가 새롭게 만들어지고 이 참조정보가 스케줄러에 저장되기 때문에 가지비 컬렉션의 대상이 되지 않음


참고 : [https://ko.javascript.info/settimeout-setinterval]