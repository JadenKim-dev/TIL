자바에서 int 끼리 사칙연산을 하면 해당 데이터의 타입은 int가 된다.
이 때 해당 연산의 결과가 int의 max 값을 넘어선 경우에도 자동으로 long 타입으로 형변환 되지 않는다.
이로 인해 int overflow가 발생하여 예상치 못한 연산 결과가 도출될 수 있다.

```java
long value = 1024 * 1024 * 1024 * 80;

-> 연산 결과: 0
```

위에서 의도한 연산 결과를 도출하기 위해서는 아래와 같이 피연산자들 중 하나를 long 타입으로 해서 적용해야 한다.

```java
long value = 1024 * 1024 * 1024 * 80L;

-> 연산 결과: 9223372036854775807
```

참고 자료: https://stackoverflow.com/questions/1494862/multiplying-long-values
