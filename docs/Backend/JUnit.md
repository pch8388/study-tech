# Parmeterized test
- 테스트 시에 같은 테스트에서 여러개의 파라미터로 테스트를 할 수 있도록 지원
- 다양한 방법으로 지원하고 있어 테스트 코드의 간결성을 유지하도록 도와줌

## MethodSource
- method 를 통해 파라미터를 정의할 수 있음
- 단순히 텍스트로 정의하여 파라미터를 넘기기 힘든 경우, 유용하게 사용가능

```java
@ParameterizedTest
@MethodSource("provideStringForDate")
@DisplayName("오늘을 포함한 이전 날짜라면 그대로 반환")
void checkDay(String searchDate) {
  final String result = argumentValidateUtils.argumentValidate(searchDate);

  assertEquals(searchDate, result);
}

private static Stream<Arguments> provideStringForDate() {
  final String today = DateUtils.today(FORMATTER);

  return Stream.of(
    Arguments.of(today),
    Arguments.of("2020-12-11"),
    Arguments.of("2019-11-20")
  );
}
```

> 이외에도 다양한 방법을 지원함

[참고](https://www.baeldung.com/parameterized-tests-junit-5)
