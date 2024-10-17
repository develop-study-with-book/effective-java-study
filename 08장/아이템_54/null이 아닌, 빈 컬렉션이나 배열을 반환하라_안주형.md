
# 이펙티브 자바 아이템 54: null이 아닌, 빈 컬렉션이나 배열을 반환하라

## 개요
메서드에서 컬렉션이나 배열을 반환할 때 `null`을 반환하는 것보다 빈 컬렉션 또는 빈 배열을 반환하는 것이 더 바람직하다. `null`을 반환하면 호출하는 쪽에서 `null` 체크를 해야 하므로 코드가 복잡해지고, 실수로 `null` 처리를 하지 않으면 `NullPointerException`이 발생할 수 있다. 반면, 빈 컬렉션이나 배열을 반환하면 이러한 문제를 예방할 수 있다.

## 핵심 내용

### 1. null 반환의 문제점
`null`을 반환하는 메서드는 호출자에게 불필요한 `null` 체크를 강요한다. 이로 인해 코드가 복잡해지고, `null` 처리가 누락되면 `NullPointerException`이 발생할 수 있다. 예를 들어, 메서드가 `null`을 반환할 수 있다면 호출자는 항상 다음과 같은 코드로 `null` 체크를 해야 한다.

#### 예시 코드: null 반환 시 문제점

```java
public class NullReturnExample {
    public static List<String> getItems(boolean condition) {
        if (condition) {
            return List.of("item1", "item2");
        } else {
            return null; // null을 반환
        }
    }

    public static void main(String[] args) {
        List<String> items = getItems(false);

        if (items != null) {  // null 체크
            for (String item : items) {
                System.out.println(item);
            }
        }
    }
}
```

위 코드에서 `getItems` 메서드가 `null`을 반환할 수 있기 때문에, 호출자는 항상 `null` 체크를 해야 한다. 만약 `null` 처리가 누락되면 `NullPointerException`이 발생할 수 있다.

### 2. 빈 컬렉션 또는 배열 반환의 장점
`null`을 반환하는 대신, 빈 컬렉션이나 배열을 반환하면 호출자는 불필요한 `null` 체크를 할 필요가 없다. 호출자가 데이터를 처리할 때 빈 컬렉션이나 배열을 처리하는 것은 매우 간단하기 때문에 코드의 가독성과 안정성이 높아진다.

#### 예시 코드: 빈 컬렉션 반환

```java
public class EmptyCollectionExample {
    public static List<String> getItems(boolean condition) {
        if (condition) {
            return List.of("item1", "item2");
        } else {
            return List.of(); // 빈 컬렉션 반환
        }
    }

    public static void main(String[] args) {
        List<String> items = getItems(false);

        for (String item : items) {  // null 체크가 필요 없음
            System.out.println(item);
        }
    }
}
```

위 코드에서는 `getItems` 메서드가 빈 리스트를 반환하므로 호출자는 `null` 체크를 하지 않고 바로 리스트를 처리할 수 있다.

### 3. 빈 배열 반환
배열도 마찬가지로 `null` 대신 빈 배열을 반환하는 것이 좋다. 자바에서는 배열을 생성할 때 기본적으로 빈 배열을 쉽게 반환할 수 있으며, 이를 통해 `null` 처리로 인한 번거로움을 줄일 수 있다.

#### 예시 코드: 빈 배열 반환

```java
public class EmptyArrayExample {
    public static String[] getItems(boolean condition) {
        if (condition) {
            return new String[] {"item1", "item2"};
        } else {
            return new String[0]; // 빈 배열 반환
        }
    }

    public static void main(String[] args) {
        String[] items = getItems(false);

        for (String item : items) {  // null 체크가 필요 없음
            System.out.println(item);
        }
    }
}
```

위 코드에서는 `getItems` 메서드가 빈 배열을 반환하므로, 호출자는 `null` 체크를 하지 않고 바로 배열을 처리할 수 있다.

### 4. 성능에 미치는 영향
빈 컬렉션이나 배열을 반환하는 것은 성능에 미치는 영향이 매우 적다. 빈 리스트나 배열은 자바에서 상수로 캐싱할 수 있기 때문에, 매번 새로 객체를 생성하는 것보다 비용이 적게 든다. 따라서 성능에 큰 영향을 주지 않으며, 오히려 코드의 간결함과 안정성을 향상시킬 수 있다.

자바의 `Collections.emptyList()`와 같은 API는 빈 리스트를 캐싱하여 반환하기 때문에 성능 저하 없이 안전하게 사용할 수 있다.

#### 예시 코드: Collections.emptyList() 사용

```java
import java.util.Collections;
import java.util.List;

public class EmptyListExample {
    public static List<String> getItems(boolean condition) {
        if (condition) {
            return List.of("item1", "item2");
        } else {
            return Collections.emptyList(); // 빈 리스트 반환
        }
    }

    public static void main(String[] args) {
        List<String> items = getItems(false);

        for (String item : items) {  // null 체크가 필요 없음
            System.out.println(item);
        }
    }
}
```

위 코드에서 `Collections.emptyList()`는 캐싱된 빈 리스트를 반환하므로, 메모리나 성능에 큰 영향을 미치지 않는다.

### 5. 불변 컬렉션 반환의 중요성
빈 컬렉션을 반환할 때, 불변 컬렉션을 반환하는 것이 중요하다. 불변 컬렉션은 외부에서 변경할 수 없으므로, 안전하게 사용할 수 있다. 자바에서 제공하는 `List.of()`나 `Collections.emptyList()`는 불변 리스트를 반환하므로 이러한 문제를 예방할 수 있다.

## 요약
- 메서드에서 `null`을 반환하지 말고, 빈 컬렉션이나 배열을 반환하라.
- `null`을 반환하면 호출자가 불필요하게 `null` 체크를 해야 하고, `NullPointerException`이 발생할 수 있다.
- 빈 컬렉션이나 배열을 반환하면 코드가 더 간결하고 안전해진다.
- 성능에 미치는 영향이 거의 없으므로, 빈 컬렉션을 안전하게 반환할 수 있다.
- 불변 컬렉션을 반환하여 외부에서의 변경을 방지하라.

빈 컬렉션이나 배열을 반환하는 것은 코드의 품질과 안전성을 높이는 중요한 방법이다.
