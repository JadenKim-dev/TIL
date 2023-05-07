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

여기서 @EqualsAndHashCode(of = "id") 어노테이션을 달고 있다.
만약 해당 어노테이션을 달지 않은 상태라면 equal 비교는 객체의 메모리 주소를 비교하는 방식으로 이루어진다.
이럴 경우 동일한 레코드에서 가져온 데이터임에도 불구하고 동등 비교를 하면 not equal이라는 결과를 얻게 된다.

```java
@Test
void equalsTest() {
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    Desk desk = Item.builder()
        .name("Desk").build();
    entityManager.persist(desk);
    transaction.commit();
    entityManager.clear();

    Desk findDesk = entityManager.find(Desk.class, desk.getId());
    assertEquals(cake, findCake); // false
}
```

db 상의 동일한 데이터를 참조하고 있다면 두 객체는 동등하게 판단하는 것이 맞다. 이를 확인하는데 가장 적절한 것은 primary key인 id가 동일한지 판단하는 것이다.
이를 위해 EqualsAndHashCode 의 of 옵션으로 id를 넘겨주면, id 값을 기준으로 동등성 체크가 이루어진다.

만약 of 옵션을 지정해주지 않으면 해당 엔티티 클래스의 모든 필드 값을 동등비교하는 식으로 equals, hashcode가 정의된다.  
하지만 이 때 필드 중 외래키가 있다면 equals 시도 시 마다 연관관계에 있는 엔티티를 조회하게 된다.  
만약 해당 연관관계 엔티티도 동일한 방식으로 equals, hashcode를 정의하고 있고 외래키로 본 클래스를 가지고 있다면, 무한 참조 루프가 발생하게 된다.  
이를 방지하기 위해서는 of 옵션을 지정해줘야 한다.

참고자료: https://velog.io/@park2348190/JPA-Entity%EC%9D%98-equals%EC%99%80-hashCode
