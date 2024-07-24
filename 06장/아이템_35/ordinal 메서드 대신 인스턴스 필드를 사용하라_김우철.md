## 아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

### enum의 ordinal 단점
- 모든 열거 타입은 해당 상수가 그 열거 타입서 몇 번째 위치인지를 반환하는 `ordinal`이라는 메서드를 제공한다.

`ordinal`을 잘못 사용한 예

```java
public enum Ensemble {
  SOLO, DUET, TRIO, ...

  public int numberOfMusicians() { return ordinal() + 1; }
}
```

문제점

- 상수 선언을 바꾸는 순간 numberOfMusicians() 오동작
- 이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없다.
- 또한, 값을 중간에 비워둘 수 없다.

### enum의 ordinal 대안

열거타입 상수에 연결된 값은 ordinal 메서드로 얻지말고, 인스턴스 필드에 저장하자.

```sql
public enum Ensemble {
  SOLO(1), DUET(2), TRIO(3), ... OCTET(8), DOUBLE_QUARTET(8)

  private final int numberOfMusicians;
  Ensemble(int size) { this.numberOfMusicians = size; }
  public int numberOfMusicians() { return numberOfMusicians; }
}
```

```java
💡 Enum API 문서의 ordinal의 정의
"대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입기반의 범용 자료구조에 쓸 목적으로 설계되었다." 
따라서 이런 용도가 아니라면 ordinal 메서드는 절대 사용하지 말자.
```
