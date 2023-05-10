@NotBlank, @Length 와 같이 validation과 관련된 annotation을 사용하기 위해서는 spring-boot-starter-validation가 dependency에 포함되어 있어야 한다.
하지만 2년 전에 스프링 부트가 새롭게 릴리즈 되면서 validation 관련 라이브러리가 Spring Web에 기본으로 추가되지 않는 것으로 변경되었다.

참고: https://www.youtube.com/watch?v=cP8TwMV4LjE

따라서 아래와 같이 의존성을 pom.xml에 직접 추가해주어야 한다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

추가적으로, Spring Boot 3점대가 릴리즈 되면서, javax.validation.constraints._ 에서 사용하던 어노테이션들이 jakarta.validation.constraints._ 을 사용하는 것으로 변경되었다.
따라서 앞으로는 jakarta 하위에 있는 annotation들을 사용해야 한다.
(대부분 동일하지만 기존에 사용하던 @Length의 경우 @Size로 대체해야 한다.)
