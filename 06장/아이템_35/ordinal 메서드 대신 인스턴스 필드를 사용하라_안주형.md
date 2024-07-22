
# 이펙티브 자바(Effective Java) 아이템 35: ordinal 메서드 대신 인스턴스 필드를 사용하라

## 개요
아이템 35에서는 열거형(enum)의 `ordinal` 메서드를 사용하는 대신, 명시적으로 정의된 인스턴스 필드를 사용할 것을 권장한다. `ordinal` 메서드는 열거형 상수가 열거형에서 정의된 순서를 반환하지만, 이 값에 의존하면 유지보수와 가독성 측면에서 문제가 발생할 수 있다.

## 문제점

### 순서에 의존하는 코드는 깨지기 쉽다
- **정렬된 순서의 변경**: 열거형의 상수 순서가 변경되면, `ordinal` 값을 사용하는 코드는 예기치 않은 동작을 할 수 있다.
- **가독성 저하**: `ordinal` 값에 의존하는 코드는 의미를 파악하기 어렵다. 코드 리뷰 시 의도를 이해하기 힘들다.

### 예제 코드: ordinal 사용
```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {
        return ordinal() + 1;
    }
}
```
- **단점**: `ordinal`에 의존하여 뮤지션 수를 계산하면, 열거형 상수의 순서가 변경될 때 코드가 의도한 대로 동작하지 않을 수 있다.

## 해결책

### 명시적인 필드 사용
- **명시적 필드**: 열거형 상수에 의미 있는 필드를 추가하여, `ordinal` 값 대신 이를 사용한다.
- **유지보수 용이**: 상수의 순서가 변경되어도 코드의 동작에 영향을 미치지 않는다.

### 예제 코드: 명시적 필드 사용
```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8), NONET(9), DECTET(10);

    private final int numberOfMusicians;

    Ensemble(int size) {
        this.numberOfMusicians = size;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```
- **장점**: `numberOfMusicians` 필드를 사용하여 열거형 상수의 순서와 관계없이 의미 있는 값을 제공한다.

## 요약
아이템 35는 열거형의 `ordinal` 메서드에 의존하지 말고, 명시적으로 정의된 인스턴스 필드를 사용할 것을 권장한다. 이는 코드의 가독성과 유지보수성을 높여주며, 상수 순서 변경에 따른 예기치 않은 동작을 방지할 수 있다.
