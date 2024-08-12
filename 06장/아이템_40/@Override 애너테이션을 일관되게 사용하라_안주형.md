
# 이펙티브 자바 아이템 40: `@Override` 애너테이션을 일관되게 사용하라

## 개요
아이템 40에서는 `@Override` 애너테이션을 일관되게 사용해야 하는 이유와 방법을 설명한다. `@Override` 애너테이션은 메서드가 상위 클래스 또는 인터페이스의 메서드를 재정의하고 있음을 나타낸다. 이 애너테이션을 일관되게 사용하면 코드의 가독성을 높이고, 실수를 방지할 수 있다.

## 핵심 내용

### 1. `@Override` 애너테이션의 역할
`@Override` 애너테이션은 메서드가 상위 클래스 또는 인터페이스의 메서드를 재정의하고 있음을 컴파일러에게 알리는 역할을 한다. 이를 통해 컴파일러는 해당 메서드가 실제로 상위 클래스의 메서드를 재정의하고 있는지 확인할 수 있으며, 그렇지 않을 경우 컴파일 오류를 발생시킨다.

### 2. 실수를 방지하는 `@Override`
`@Override` 애너테이션을 사용하면 오타나 잘못된 매개변수 목록으로 인해 메서드가 재정의되지 않는 문제를 방지할 수 있다. 만약 상위 클래스의 메서드를 정확히 재정의하지 않았다면, 컴파일러가 이를 경고하거나 오류로 처리해준다.

#### 예시 코드

```java
public class SuperClass {
    public void someMethod() {
        // ...
    }
}

public class SubClass extends SuperClass {

    // @Override가 없다면, 아래 메서드는 잘못된 오버로딩으로 인식될 수 있다.
    public void someMethod(int param) {
        // ...
    }
}
```

위 코드에서 `SubClass`는 `someMethod(int param)` 메서드를 추가하지만, 이는 `SuperClass`의 `someMethod()`를 재정의한 것이 아니라 새로운 메서드를 정의한 것이다. `@Override`를 사용하지 않으면 이러한 실수를 쉽게 놓칠 수 있다.

### 3. 인터페이스 메서드 재정의 시 `@Override` 사용
Java 6부터는 인터페이스의 메서드를 구현할 때도 `@Override` 애너테이션을 사용할 수 있다. 이를 통해 인터페이스 메서드 구현 시 발생할 수 있는 실수를 방지할 수 있다.

#### 예시 코드

```java
public interface MyInterface {
    void doSomething();
}

public class MyClass implements MyInterface {

    @Override
    public void doSomething() {
        // 인터페이스 메서드의 올바른 구현
    }
}
```

### 4. 모든 재정의 메서드에 `@Override`를 사용해야 하는 이유
`@Override` 애너테이션을 모든 재정의 메서드에 사용하는 것은 권장 사항이다. 이를 통해:
- 코드의 일관성을 유지할 수 있다.
- 상위 클래스 또는 인터페이스의 메서드를 재정의하고 있음을 명확히 알 수 있다.
- 실수를 방지하고, 컴파일러의 검증을 받을 수 있다.

### 5. `@Override` 애너테이션을 사용하지 않으면 발생할 수 있는 문제
`@Override`를 사용하지 않으면, 실수로 메서드를 정확히 재정의하지 않거나 상위 클래스 또는 인터페이스의 메서드를 잘못된 형태로 재정의하는 경우가 발생할 수 있다. 이러한 실수는 런타임 오류를 유발할 수 있으며, 디버깅을 어렵게 만든다.

#### 예시 코드

```java
public class SuperClass {
    public void someMethod() {
        // ...
    }
}

public class SubClass extends SuperClass {

    // 오타로 인해 재정의가 아닌 새로운 메서드가 생성되었다.
    public void somMethod() {
        // ...
    }
}
```

위 코드에서 `SubClass`는 `SuperClass`의 `someMethod()`를 재정의하려 했으나, 오타로 인해 `somMethod()`라는 새로운 메서드를 정의했다. `@Override`를 사용했다면, 컴파일러가 오류를 발생시켜 실수를 바로잡을 수 있었을 것이다.

## 요약
- `@Override` 애너테이션은 메서드가 상위 클래스 또는 인터페이스의 메서드를 재정의하고 있음을 나타낸다.
- 이 애너테이션을 사용하면 코드의 가독성을 높이고, 실수를 방지할 수 있다.
- 모든 재정의 메서드에 `@Override`를 사용하는 것이 권장된다.
- 이를 통해 실수를 방지하고, 컴파일러의 검증을 받을 수 있다.

일관되게 `@Override` 애너테이션을 사용함으로써 코드의 품질을 높이고 유지보수성을 개선할 수 있다.
