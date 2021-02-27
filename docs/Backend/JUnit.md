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

# TestMethod Order
- Test 를 일정 순서에 따라 해야할 경우
```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class ServiceTest {
  @Order(1)
  @ParammeterizedTest
  // csv file 을 읽어와서 테스트 반복수행
  @CsvFileSource(resource = "data.csv", numLinesToSkip = 1)
  void first_test(String first, String second) {
      //...
  }
  @Order(2)
  @ParammeterizedTest
  // csv file 을 읽어와서 테스트 반복수행
  @CsvFileSource(resource = "data2.csv", numLinesToSkip = 1)
  void second_test(String first, String second) {
      //...
  }
}
```
- 위의 경우 first_test 메서드 수행 후 second_test 가 수행되는 것을 항상 보장한다

# 테스트 전용 config 클래스 설정
```java
@Import(ServiceTest.TestConfig.class)
class ServiceTest {

  @TestConfiguration
  static class TestConfig {
    @Bean
    @Primary  // primary 로 지정하여 더 우선순위로 주입시킨다
    public InjectService injectService() {
      return new InjectService();
    }
  }
}
// 특정 인터페이스를 구현하는 mock 클래스를 구현하여 구현부를 바꿔치기한다.
class InjectService implements Service {
  //...
}
```