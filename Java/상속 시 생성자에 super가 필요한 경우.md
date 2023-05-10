아래와 같이 생성자를 정의하고 있는 클래스가 있다고 하자

```java
public class Parent {
    private String name;

    public Parent(String name) {
        this.name = name;
    }
}
```

이 클래스를 상속 받는 자식 클래스를 아래와 같이 만들었다.

```java
public class Child extends Parent {
}
```

이렇게 할 경우 다음의 에러메시지를 담은 컴파일 에러가 발생하게 된다.  
`There is no default constructor available in 'com.company.Person'`

이러한 에러가 발생하는 이유는, 부모 클래스에서 생성자를 정의하므로써 기본 생성자가 만들어지지 않기 때문이다.  
(기본 생성자는 아무런 인자를 받지 않고 그저 클래스의 인스턴스를 생성하기만 하는 생성자이다.)

자식 클래스의 생성자에서 부모 클래스에서 정의해둔 필드들에 값을 채워 넣기 위해서는,  
아래와 같이 super 문을 통해 상위 클래스의 생성자를 호출해줘야 한다.

```java
public class Child extends Parent {
    public Child(String name) {
        super(name);
    }
}
```
