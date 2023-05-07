@RequiredArgsConstructor의 경우 final이 붙거나 @NotNull 이 붙은 필드에 대한 값을 채워주는 생성자를 자동 생성해주는 어노테이션이다.  
이 덕분에 스프링에서 생성자 주입으로 의존성을 넣어줄 때 간단하게 코딩할 수 있다.

다음은 예시 코드이다.

```java
@Component
@RequiredArgsConstructor
public class AccountValidator {

    private final AccountRepository accountRepository;

}
```

Spring에서는 생성자가 하나만 존재할 경우에는 자동으로 생성자에 @Autowired를 붙여준다.  
따라서 RequiredArgsConstructor를 통해 AccountRepository에 대한 값을 넣어주는 생성자가 만들어지면, 여기에 추가적인 작업할 필요 없이 의존성 주입 구현이 끝난다.

참고 자료: https://mangkyu.tistory.com/125
