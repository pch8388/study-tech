# double-checked locking
- 일반적으로 멀티쓰레드 환경에서 lazy initialization 을 위해 사용되는 기법
- 멀티쓰레드 환경에서도 안전하게 singleton pattern 사용

## 예제
- [출처](https://en.wikipedia.org/wiki/Double-checked_locking#cite_note-bdec-2)
```java
// java 1.5 이상의 버전에서만 정상 작동함
class Foo {
    private volatile Helper helper;
    public Helper getHelper() {
        Helper localRef = helper;
        if (localRef == null) {
            synchronized (this) {
                localRef = helper;
                if (localRef == null) {
                    helper = localRef = new Helper();
                }
            }
        }
        return localRef;
    }

    // other functions and members...
}
```

- volatile : 항상 데이터를 main memory 에서만 read / write 하도록 보장
  - 쓰지 않으면 cpu 레지스터에 데이터를 캐시해두고 사용할 가능성이 있음
    - 데이터 정합성이 더 중요한 멀티쓰레드 환경에서는 성능을 일부 포기하고 사용할 수도 있음