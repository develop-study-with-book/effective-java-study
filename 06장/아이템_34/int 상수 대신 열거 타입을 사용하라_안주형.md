# 이펙티브 자바(Effective Java) 아이템 34: 열거형을 사용하라

## 개요
아이템 34에서는 자바에서 int 상수를 사용하는 것보다 열거형(enum)을 사용하는 것이 더 나은 이유를 설명한다.
열거형을 사용하면 코드의 가독성, 안전성, 유지보수성 등이 향상된다.

## 열거형(Enum)의 장점

### 타입 안전성(Type Safety)
- **타입 안정성 보장**: 열거형을 사용하면 특정 집합의 명명된 값만 사용할 수 있다. 이는 잘못된 값이 사용될 가능성을 줄여주는데 예를 들어, `Apple.FUJI`와 같이 명확한 값을 사용할 수 있다.
- **컴파일타임 오류 검출**: 열거형을 사용하면 잘못된 값 사용 시 컴파일타임에 오류를 검출할 수 있어 안전하다.

```java
// int 상수를 사용하는 경우
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

int appleType = APPLE_FUJI;

// 잘못된 값 사용 가능
appleType = 3; // 컴파일타임 오류 없음

// 열거형을 사용하는 경우
public enum Apple {
    FUJI, PIPPIN, GRANNY_SMITH
}

Apple appleType = Apple.FUJI;

// 잘못된 값 사용 시 컴파일타임 오류
// appleType = 3; // 컴파일 오류
```

### 가독성(Readability)
- **명시적 이름**: 열거형은 명시적인 이름을 가지므로, 코드의 의도가 명확하게 전달된다. 따라서 코드 리뷰나 유지보수 시에 큰 도움이 된다.
- **의미 있는 표현**: `0`, `1`, `2`와 같은 숫자 상수 대신 `Apple.FUJI`, `Apple.PIPPIN`, `Apple.GRANNY_SMITH`와 같은 의미 있는 이름을 사용할 수 있다.

```java
// int 상수를 사용하는 경우
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

int appleType = APPLE_FUJI; // 의도를 이해하기 어렵다

// 열거형을 사용하는 경우
public enum Apple {
    FUJI, PIPPIN, GRANNY_SMITH
}

Apple appleType = Apple.FUJI; // 의도가 명확하다
```

### 값의 구체화(Fixed Set of Constants)
- **고정된 상수 집합**: 열거형은 고정된 집합의 상수를 정의하므로, 그 집합의 멤버가 변경되지 않음을 보장한다. 이를 통해 예측 가능한 코드 작성이 가능하다.
- **코드의 안정성**: 열거형의 값은 변경될 수 없으므로, 코드의 안정성이 향상된다.
```java
// int 상수를 사용하는 경우
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

// 잘못된 값 사용 가능
int appleType = 4; // 논리적으로 잘못된 값

// 열거형을 사용하는 경우
public enum Apple {
    FUJI, PIPPIN, GRANNY_SMITH
}

Apple appleType = Apple.FUJI;

// 열거형의 멤버는 고정되어 있어 잘못된 값을 사용할 수 없다
// appleType = 4; // 컴파일 오류
```


### 메서드 추가 가능(Method Addition)
- **관련 동작 정의 가능**: 열거형은 메서드를 포함할 수 있어, 관련된 동작을 함께 정의할 수 있다. 이를통해 관련 로직을 열거형 내부에 캡슐화할 수 있다.
- **복잡한 로직 처리**: 각 열거형 상수에 대해 고유한 동작을 정의할 수 있어, 복잡한 로직도 깔끔하게 처리할 수 있다.
```java
public enum Apple {
    FUJI, PIPPIN, GRANNY_SMITH;

    public String getColor() {
        switch (this) {
            case FUJI: return "Red";
            case PIPPIN: return "Green";
            case GRANNY_SMITH: return "Green";
            default: throw new AssertionError("Unknown apple: " + this);
        }
    }
}

Apple appleType = Apple.FUJI;
String color = appleType.getColor(); // "Red"
```

```java
public enum Operator {
    PLUS("+", (num1, num2) -> Integer.valueOf(num1 + num2)),
    MINUS("-", (num1, num2) -> Integer.valueOf(num1 - num2)),
    MULTIPLY("*", (num1, num2) -> Integer.valueOf(num1 * num2)),
    DIVIDE("/", (num1, num2) -> Integer.valueOf(num1 / num2));

    private final String operator;
    private final BiFunction<Integer, Integer, Integer> calculator;

    Operator(String symbol, BiFunction<Integer, Integer, Integer> calculator) {
        this.operator = symbol;
        this.calculator = calculator;
    }

    public String getOperator() {
        return operator;
    }

    public static Integer calculator(String operator, Integer num1, Integer num2) {
        return getOperator(operator).calculator.apply(num1, num2);
    }

    private static Operator getOperator(String operator) {
        return Arrays.stream(values())
                .filter(o -> o.operator.equals(operator))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("사칙 연산 기호만 가능합니다."));
    }
}

```


### 필드 추가 가능(Field Addition)
- **관련 데이터 저장**: 열거형은 필드를 가질 수 있어, 각 열거형 상수와 관련된 데이터를 저장할 수 있다. 예를 들어, 색상, 가격 등의 속성을 열거형에 추가할 수 있다.
- **상태 유지**: 열거형 인스턴스마다 고유한 상태를 유지할 수 있다.
```java
public enum Apple {
    FUJI("Red"), PIPPIN("Green"), GRANNY_SMITH("Green");

    private final String color;

    Apple(String color) {
        this.color = color;
    }

    public String getColor() {
        return color;
    }
}

Apple appleType = Apple.FUJI;
String color = appleType.getColor(); // "Red"
```


### 기본적인 기능 제공(Built-in Features)
- **자동 제공 메서드**: 열거형은 자동으로 제공되는 `toString()`, `name()`, `ordinal()` 등의 메서드를 통해 관련 기능을 쉽게 구현할 수 있다.
- **다양한 활용**: 이러한 기본 메서드를 활용하여 열거형을 다양한 상황에서 유용하게 사용할 수 있다.
```java
public enum Apple {
    FUJI, PIPPIN, GRANNY_SMITH
}

Apple appleType = Apple.FUJI;

// 기본 제공 메서드 사용
String name = appleType.name(); // "FUJI"
int ordinal = appleType.ordinal(); // 0
String toString = appleType.toString(); // "FUJI"
```

## 요약
아이템 34는 자바에서 int 상수 대신 열거형(enum)을 사용하는 것이 중요하다고 강조한다. 열거형을 사용하면 타입 안전성, 가독성, 유지보수성 등 여러 면에서 장점을 얻을 수 있다.
따라서 자바 개발에서는 enum을 적극 활용하여 코드의 품질과 안정성을 높이는 것이 권장된다.
