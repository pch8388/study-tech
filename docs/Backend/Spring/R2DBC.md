# oracle 연동
[driver](https://github.com/oracle/oracle-r2dbc)

현재(2021-10-24) r2dbc 오라클 드라이버는 0.3 버전이지만 스프링에서 지원가능한 버전은 0.1
- R2DBC SPI 0.9 버전 이상이 필요한데 스프링에서 제공하는 것은 0.8x
- 스프링에서 쓰려면 커스텀 설정이 많이 필요한 듯 해보임
- 그냥 쓰려고 하면 R2dbcDialectProvider 구현체를 제공하라고 한다
  - Mysql, PostgreSQL, 심지어 ms-sql 도 지원되지만 oracle 은 dialect 를 따로 제공해야 하는 것으로 보임
  