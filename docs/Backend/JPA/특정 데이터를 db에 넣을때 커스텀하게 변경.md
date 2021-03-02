# JPA 에서 엔티티 <-> 데이터베이스 데이터 변환
- JPA 의 엔티티에서 enum 을 사용하였을 때 DB 에 저장하는 옵션은 두가지가 있음
  - @Enumerated(value = EnumType.ORDINAL) : 정렬된 순서의 인덱스
  - @Enumerated(value = EnumType.STRING) : enum 정의 이름
- 특정한 값으로 변환하고 싶을 때 할 수 있는 방법은 AttributeConverter 를 구현하면 됨

## 구현
```java
public enum ApproveType {
  APPROVE("10"), CANCEL("20");

  private final String code;

  ApproveType(String code) {
	this.code = code;
  }

  public String getCode() {
	return code;
  }
}

@Converter(autoApply = true)
public class ApproveTypeConverter implements AttributeConverter<ApproveType, String> {

  /**
  * ApproveType 을 데이터 베이스에 넣을 때 원하는 형태로 변환하여 넣음
  */
  @Override
  public String convertToDatabaseColumn(ApproveType result) {
	if (result == null) {
  	  return null;
	}
	return result.getCode();
  }

  /**
  * ApproveType 을 데이터 베이스에서 가져올 때 원하는 형태로 변환
  */
  @Override
  public ApproveType convertToEntityAttribute(String code) {
	if (code == null) {
  	  return null;
	}

	return Stream.of(ApproveType.values())
		.filter(r -> r.getCode().equals(code))
		.findAny()
		.orElseThrow(IllegalArgumentException::new);
  }
}
```
- 결과적으로 "10" 이나 "20" 이란 값이 저장되고, 데이터베이스에서 가져올 때는 정상적으로 enum type 으로 매핑된다