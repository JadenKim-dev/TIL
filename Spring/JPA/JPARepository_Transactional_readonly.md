JPARepository의 구현체의 경우 @Transactional(readOnly = true)를 위에 붙이는 경우가 많다.

```java
@Transactional(readOnly = true)
public interface ItemRepository extends JpaRepository<Item, Long> {

    boolean existsByName(String email);

    boolean findByName(String email);

}
```

이렇게 구현하는 이유는 성능 이슈 때문이다.  
readonly 옵션 없이 @Transactional을 붙이게 되면 엔티티 객체에 대한 변경 감지가 이루어지고, 스냅샷을 저장한다.

하지만 JPARepository의 조회 관련 메서드들에게 까지 이러한 기능이 작동되는 것은 불필요하고, 비효율적이다.
따라서 보통 성능 개선을 위해서 JPARepository의 구현체에는 @Transactional(readOnly = true)을 붙이는 경우가 많다.  
만약 일부 메서드에서 객체 수정이 필요하다면 해당 객체만 특정해서 @Transactional을 달아준다. (readonly 옵션의 기본값은 false이다.)

```java
@Transactional(readOnly = true)
public interface ItemRepository extends JpaRepository<Item, Long> {

    boolean existsByName(String name);

    boolean findByName(String name);

    @Transactional
    void deleteByName(String name);

}
```
