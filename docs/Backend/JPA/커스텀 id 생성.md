# custom id generator
- 기본적으로 JPA 에서는 데이터베이스의 기능을 이용하여 순차적인 id를 생성
- 커스텀하게 증가시키려는 요구사항이 생겨 구현

## IdentifierGenerator
- 사용자가 id 를 커스텀하게 생성하려면 구헌할 수 있는 인터페이스
- Configurable 인터페이스와 같이 사용하면 설정 파라미터에 대한 설정도 가능

### 구현
- 원하는 구현 요건은 varchar2(10) 에 증가하는 값을 제외한 값은 0으로 채워넣음

```java
public class LeftPadPrefixIdentityGenerator implements IdentifierGenerator {

	public static final String QUERY = "select max(%s) from %s";
	private static final String ID_FORMAT = "%10s";
	public static final String TARGET_STRING = " ";
	public static final String FILL_STRING = "0";
	public static final long DEFAULT_VALUE = 0L;
	public static final int INCREASE_VALUE = 1;

	@Override
	public Serializable generate(SharedSessionContractImplementor session, Object obj) {
		String query = String.format(QUERY,
			session.getEntityPersister(obj.getClass().getName(), obj).getIdentifierPropertyName(),
			obj.getClass().getSimpleName());

		final Long max = session.createQuery(query, String.class)
			.stream()
			.filter(Objects::nonNull)
			.findAny()
			.map(Long::parseLong)
			.orElse(DEFAULT_VALUE);

		return String.format(ID_FORMAT, max + INCREASE_VALUE).replace(TARGET_STRING, FILL_STRING);
	}
}

@Entity
public class Pay {
  @Id
  @GeneratedValue(generator = "pay-generator")
  // strategy 에는 풀패키지명을 적어야 함
  @GenericGenerator(name = "pay-generator", strategy = "LeftPadPrefixIdentityGenerator")
  private String id;
}
```

[참고](https://woowabros.github.io/study/2019/01/30/identifier_generator.html)