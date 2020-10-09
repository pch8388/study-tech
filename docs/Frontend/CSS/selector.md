# 기본문법
```css
selector {
    property: value;
}
```

# selector
- \* : 모든 태그 선택
    ```css
    * {
        color: green;
    }
    ```
- type : 태그 이름을 지정
    ```css
    li {
        color: blue;
    }
    ```
- id : id 지정
    ```css
    #id {
        color: pink;
    }
    ```
- class : 클래스 지정
    ```css
    .red {
        width: 100px;
        height: 100px;
        background: yellow;
    }
    ```
- 상태 선택자
    ```css
    button:hover {
        color: red;
        background: beige;
    }
    ```
- 속성 값에 따라 선택
    ```css
    a[href="naver.com"] {
        color: purple;
    }
    ```

# 선택자 우선순위
1. !important
2. 인라인 스타일 속성에 지정한 규칙 그룹
3. 아이디
4. 클래스 / 속성 / 가상 선택자
5. 요소 선택자
6. 전체 선택자

- 가장 마지막에 연결/삽입된 스타일 시트가 앞의 스타일시트보다 높은 우선순위를 가짐