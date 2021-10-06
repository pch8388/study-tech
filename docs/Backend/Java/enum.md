# Enum 에 abstract method 를 정의
enum에 abstract method를 정의하여 각 인스턴스마다 구현만 제공할 수 있다

```java
public enum AbstractMethodEnum {
  PRINT {
    @Override
    String apply(String str) {
        return str + " enum test";
    }
  };

  abstract String apply(String data);
}

class AbstractMethodEnumTest {
  @DisplayName("enum 에 abstract method 구현")
  @Test
  void apply() {
    assertEquals("abstract method enum test",
        AbstractMethodEnum.PRINT.apply("abstract method"));
  }
}
```
- 활용하면 다양한 외부 메서드를 받아와서 사용할 수 있다