- 참고 : Array.prototype.splice(start, deleteCount, item...) => start 부터 deletecount 만큼 삭제, 그 위치에 item 삽입

# 함수
- 함수도 객체
- 함수코드는 함수 객체의 [[Code]] 내부 프로퍼티에 자동으로 저장
- lenght, prototype 프로퍼티를 갖는다
  - name, caller, arguments, __proto__ 프로퍼티도 갖는다
  - caller, callee 등의 함수는 strict 모드에서 작동하지 않음
- 프로토타입으로 Function.prototype 객체를 가짐
  - constructor
  - toString()
  - apply(thisArg, argArray)
  - call(thisArg, [...arg])
  - bind(thisArg, [...arg])
- prototype 프로퍼티 : constructor 프로퍼티만 있는 객체를 가리킴
  - constructor : 자신과 연결된 함수를 가리킴(prototype 프로퍼티가 자신을 가리키는데 자신을 가리키는 함수 자체를 가리킴)
  - prototype -> constroctor -> 함수
