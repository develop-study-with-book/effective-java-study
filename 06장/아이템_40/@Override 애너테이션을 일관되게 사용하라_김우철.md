## @Overrid 애너테이션을 일관되게 사용하라

### @Overried란

```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}

```
- @Override 애너테이션
  - 상위 타입에 정의된 메서드를 재정의할 때 사용
  - 컴파일러가 메서드 오버라이딩을 확인할 때 사용하는 메타 어노테이션
- @Target(ElementType.METHOD)
  - 메서드 선언에만 달수 있음
- @Retention(RetentionPolicy.SOURCE)
  - 컴파일러가 @Override를 찾아 컴파일 시점에 메서드를 올바르게 재정의 했는지 검사


### @Overried 예시

```java
public class Person {

    private final String lastName;
    private final String firstName;

    public Person(String lastName, String firstName) {
        this.lastName = lastName;
        this.firstName = firstName;
    }

    @Override
    public int hashCode() {
        return Objects.hash(lastName, firstName);
    }

    //@Override
    public boolean equals(Person obj) {
        return this.lastName.equals(obj.lastName) && this.firstName.equals(obj.firstName);
    }

    public static void main(String[] args) {
        Person kimwoocheol = new Person("kim", "woocheol");
        Person cloneKimwoocheol = new Person("kim", "woocheol");
        Set<Person> set = new HashSet<>();
        set.add(kimwoocheol);
        set.add(cloneKimwoocheol);
        System.out.println("size: " + set.size()); // 2
    }
}
```

- lastName과 firstName이 같으면 같은 객체로 간주
- HashSet은 hashCode → equals(Object obj) 메서드 순으로 비교 하는데, equals(Object obj)가 아닌 equals(Person obj)로 정의하여 재정의가 아닌 다중 정의가 되어 버렸다.
- 따라서, lastName과 firstName이 같더라도 equals(Object obj)로 비교하기 때문에 다른 객체로 간주한다.

```java
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Person p)) {
            return false;
        }
        return this.lastName.equals(p.lastName) && this.firstName.equals(p.firstName);
    }
```

- 위와 같이 equals(Object obj)로 재정의해야 제대로 동작한다.

### @Overried를 잘 달자

- **잘못된 코드에서 @Override를 달았으면 컴파일 타임에서 오류를 조기에 발견할 수 있었을 것이다.**
- 상위 클래스의 메서드를 재정의 하려는 모든 메서드에 @Override 애너테이션을 달자.
  - 한가지 에외는 구체 클래스에서 상위 클래스의 추상 메서드를 재정의하지 않았으면 컴파일러가 바로 알려줘서 달지 않아도 되지만 일괄적으로 붙이는것도 상관없다.

### 결론

실수 방지를 위해 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달자