JPA 하에서 엔티티 객체를 만들 때 보통 다음과 같이 어노테이션을 만든다(Builder를 사용)

```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Sample {
    @Id @GeneratedValue
    private Long id;
}
```

이 때 @Builder를 사용하려면 @AllArgsConstructor가 함께 존재해야 한다. 만약 누락할 경우 아래와 같은 에러가 발생한다.
`Lombok @Builder needs a proper constructor for this class`
이러한 에러가 발생하는 이유는, 빌더는 엔티티 클래스의 모든 필드를 초기화할 수 있는 생성자가 정의되어 있어야 하기 때문이다.

그런데 @AllArgsConstructor 어노테이션을 달아두고 @NoArgsConstructor를 달지 않으면 아래와 같은 에러가 발생한다.
`Class 'Account' should have [public, protected] no-arg constructor `

이 에러는 해당 클래스가 @Entity 어노테이션이 붙은 엔티티 클래스여서 발생한다.
엔티티 클래스의 경우 스펙 상 기본 생성자를 요구하고 있다.
위의 예시의 객체에는 @AllArgsConstructor가 이미 붙어 있기 때문에 기본 생성자가 만들어지지 않는다.
따라서 @NoArgsConstructor를 추가로 붙여서 기본 생성자를 만들어주도록 설정해줘야 하는 것이다.

하지만 그렇다고 해서 기본 생성자에 대한 접근을 public으로 열어놓게 되면 개발 과정에서 정의해둔 builder를 사용하지 않고 무분별하게 기본 생성자를 사용할 수 있다.  
이를 방지하기 위해 다음과 같이 옵션을 추가해서 @NoArgsConstructor를 사용한다. `@NoArgsConstructor(access = AccessLevel.PROTECTED)`

여기서 주의할 점은, access level을 private으로 설정하면 안 된다는 것이다.  
연관 관계를 가지고 있는 엔티티를 lazy로 조회 시 Proxy Entity를 만들어서 사용하는데, 접근 권한이 private인 경우에는 프록시 객체 생성 자체가 불가능해진다.
프록시 객체 생성이 정상적으로 이루어지기 위해서는 접근권한을 PROTECTED 또는 PUBLIC 으로 설정해야 한다.

참고 자료:
https://velog.io/@gowjr207/Entity-%EC%97%90-%EC%93%B0%EC%9D%B4%EB%8A%94-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98  
https://erjuer.tistory.com/106
