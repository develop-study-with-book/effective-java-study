## 아이템 36. 비트 필드 대신 EnumSet을 사용하라

### 구닥다리 비트필드 열거 상수

```java
public class Text {
  public static final int STYLE_BOLD         = 1 << 0; // 1 -> 0001
  public static final int STYLE_ITALIC       = 1 << 1; // 2 -> 0010
  public static final int STYLE_UNDERLINE     = 1 << 2; // 4 -> 0100
  public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8 -> 1000

 private int styles;

  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
  public void applyStyles(int styles) {
     styles = styles;
  }

}
```

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```
- 위와 같은 식으로 비트별 OR을 사용해 여러 상수를 하나의 집합으로 모을 수 있다. 이렇게 만들어진 집합을 비트 필드라고 한다.

```
  private int styles;

  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
  public void applyStyles(int styles) {
     styles = styles;
  }

applyStyles(STYLE_BOLD | STYLE_ITALIC) 0001 | 0010을 수행하면 0011이 되고 3이 생성
=> int styles에 3이 대입

applyStyles(STYLE_BOLD | STYLE_UNDERLINE) 0001 | 0100을 수행하면 0101이 되고 5가 생성
=> int styles에 5가 대입

applyStyles(STYLE_BOLD | STYLE_STRIKETHROUGH) 0001 | 1000을 수행하면 1001이 되고 9를 생성
=> int styles에 9가 대입

applyStyles(STYLE_BOLD | STYLE_ITALIC | STYLE_UNDERLINE | STYLE_STRIKETHROUGH);
=> int styles에 15(1111)가 대입
//...
```
- 적용할 글씨체를 인자에 선언 후 OR 연산을 하면 된다.


#### 장점
- 메모리를 절약할 수 있다.
    - 위 예시에서 어떤 글씨체를 적용한 건지를 나타내는 것이므로 int 대신 boolean 타입을 사용할 수도 있다.

    ```java
    public class Text {
      public boolean STYLE_BOLD         = 1 << 0; // 1 -> 0001
      public boolean STYLE_ITALIC       = 1 << 1; // 2 -> 0010
      public boolean STYLE_UNDERLINE     = 1 << 2; // 4 -> 0100
      public boolean STYLE_STRIKETHROUGH = 1 << 3; // 8 -> 1000
    ```

    - boolean 타입은 1byte == 8 bit이기 때문에 비트 집합을 사용할 때보다 메모릴 더 사용한다.

#### 단점

- 해석하기 어렵다

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC); // BOLD | UNDERLINE은 3
text.applyStyles(3); // 0011(3)을 봤을때 어떤 글씨체가 적용되었는지 해석이 어렵다
```

- 여러 상태가 저장된 비트 필드를 순회하는 것은 까다롭다.
    - **로직이 복잡해서 까다롭다고 하는건가?**

```java
  public static final int STYLE_BOLD          = 1 << 0; // 1 -> 0001
  public static final int STYLE_UNDERLINE     = 1 << 2; // 4 -> 0100

STYLE_BOLD | STYLE_UNDERLINE // 0101 (정수 4)

// 비트 필드를 하나씩 순회하면서 어떤 비트를 가지고 있는지 검사
int styles = STYLE_BOLD | STYLE_UNDERLINE
for (int i = 0; i < 4; i++) {
  int mask = 1 << i;
    if ((styles & mask) != 0) { // AND 연산 결과 0이 아니면 해당 비트가 적용되어 있음
      System.out.println("Style " + mask + " is applied.");
    }
} 

// 출력
Style 1 is applied.
Style 4 is applied.

```

- 확장성에 취약하다.
    - int 타입은 32비트다. 처음에 int로 구현했는데 32비트보다 더 필요한 일이 생기면 long 타입으로 코드를 변경해야 비트 수를 늘릴 수 있다.

## EnumSet

- 위의 단점을 EnumSet을 사용하면 해결할 수 있다.
- EnumSet은 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

```java
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE, STRIKEROUGH }

 private Set<Style> styles;

  public void applyStyles(Set<Style> styles) {
     styles = styles;
  }
}
```

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

- EnumSet.of를 사용해서 적용하려는 글씨체를 표현할 수 있다.

## 장점

- 가독성이 뛰어나다.
- Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.
- 확장성이 좋다.
    - 집합의 개수에 따라 64개를 기준으로 작거나 같으면 RegularEnumSet, 크면 JumboSet을 사용하도록 만들어져 있다

    ```java
      public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
            Enum<?>[] universe = getUniverse(elementType);
            if (universe == null)
                throw new ClassCastException(elementType + " not an enum");
    
            if (universe.length <= 64)
                return new RegularEnumSet<>(elementType, universe);
            else
                return new JumboEnumSet<>(elementType, universe);
        }
    ```

    - 집합의 개수가 64개 이하이면 RegularEnumSet, 65개 이상이면 JumboSet을 사용한다.
- EnumSet 의 내부는 비트 벡터로 구현되어 있다.  RegularEnumSet는 64비트로 표현하는 long 타입의 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
- EnumSet 내부의 removeAll, retainAll도 비트를 효율적으로 처리할 수 있는 산술연산을 써서 구현했다.

##  불변 EnumSet

- 불변 EnumSet은 자바 진영에서 지원해주지 않고 있다. 불변으로 사용하려면 Collections.unmodifiableSet을 사용하여 EnumSet을 감싸자.

## 요약

```text
💡 열거할 수 있는 타입을 한데 모아 집합 형태로 사용하려면 비트 필드를 사용하지 말고 EnumSet을 사용하자. 비트 필드의 단점을 모두 커버 하였으니!!
```