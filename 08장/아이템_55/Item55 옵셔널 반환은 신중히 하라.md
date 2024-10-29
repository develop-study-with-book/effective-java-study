## Item55 옵셔널 반환은 신중히 하라

### 자바8이전

- 자바8 이전에는 특정 조건에서 값을 반환할 수 없을 때 예외를 던지거나, null을 반환하였다.
- 예외는 너무 남발할 가능성이 있고 스택 트레이스에 대한 비용이 있다.
    - 호출 스택을 계속 탐색하고, 정보를 모아 stacktrace로 만드는 과정의 비용이 많이 든다.
    - [https://velog.io/@hope0206/Java의-예외-생성-비용-비용-절감-방법](https://velog.io/@hope0206/Java%EC%9D%98-%EC%98%88%EC%99%B8-%EC%83%9D%EC%84%B1-%EB%B9%84%EC%9A%A9-%EB%B9%84%EC%9A%A9-%EC%A0%88%EA%B0%90-%EB%B0%A9%EB%B2%95)
- null을 반환하면 호출하는 쪽에서 null 처리 코드를 추가해야 한다

### 자바8이후

- 자바8 이후에는 값이 없을때 예외나 null 대신 Optional<T>를 사용하면 된다.
- Optional을 사용하지 않는 코드

```java
// 컬렉션에서 최댓값을 구하는 코드(컬렉션이 비었으면 예외를 던짐)
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("빈 컬렉션");
    
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    
    return result;
}
```

- 빈 컬렉션을 건네면 예외를 던진다.

```java
public static <E extends Comparable<E>>
    Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();
    
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    
    return Optional.of(result);
}
```

- 빈 컬렉션을 건네면 Optional<E>를 반환한다.(추천)
- 빈 옵셔널은 Optional.empty()로 만들고, 값이 든 옵셔널은 Optional.of(value)로 생성한다.
- null 값도 허용하는 옵셔널은 Optional.ofNullable(value)를 사용하자
- **옵셔널을 반환하는 메서드에서는 절대 null을 반환하면 안된다.**
    - null 체크를 피하려고 Optional을 쓰는데, 다시 null 체크를 해야 하기 때문이다.

### 스트림에서의 옵셔널

- 스트림의 종단 연산 중 상당수가 옵셔널을 반환한다

```java
public static <E extends Comparable<E>>
    Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

- 스트림의 max 연산이 옵셔널을 생성해준다.

### 옵셔널 기준

- 옵셔널은 검사 예외와 취지가 비슷하다
    - API 사용자에게 특별한 처리가 필요한 상황을 강제하는 메커니즘
    - 예외처리(checked): try-catch 강제
    - Optional: Optional 메서드를 반드시 사용해야 함
- 옵셔널은 반환 값이 없을 수도 있음을 API 사용자에게 명확히 알려줘야 할 때 사용한다

- 클라이언트가 옵셔널에서 원하는 값을 받지 못했을때 선택 사항

```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

- orElse로 기본값 설정

```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

- orElseThrow로 예외를 던짐

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

- 값이 있다고 확신하면 .get으로 꺼내기
- 하지만 값이 없다면 NoSuchElementException이 발생

### 여러가지 메서드들

- 기본값을 설정하는 비용이 커서 부담되면 Supplier<T>를 인수로 받는 orElseGet을 사용하자.
    - orElse는 Optional에 값이 있든 없는 호출하고 orElseGet은 값이 없을 때만 Supplier를 실행한다(지연 평가 방식)
- 값이 처음 필요할 때 Supplier<T>를 사용해 생성하므로 초기 설정 비용을 낮출 수 있다
- filter, map, flatMap, ifPresent 등등 특별한 메서드들이 많이 있다.
- `filter(Predicate<? super T> predicate)`:
    - Optional 값이 존재하고 주어진 조건(predicate)을 만족하면 그 값을 포함한 Optional을 반환
    - 조건을 만족하지 않으면 빈 Optional을 반환
- `map(Function<? super T, ? extends U> mapper)`:
    - Optional 값이 존재하면 주어진 매핑 함수를 적용한 새로운 Optional을 반환
    - 값이 없으면 빈 Optional을 반환
- `flatMap(Function<? super T, Optional<U>> mapper)`:
    - map과 유사하지만, 매핑 함수가 Optional을 반환할 때 사용
    - 중첩된 Optional을 평탄화
    - 예: optional.flatMap(s -> Optional.of(s.length()))
- `ifPresent(Consumer<? super T> consumer)`:
    - Optional 값이 존재하면 주어진 Consumer를 실행
    - 값이 없으면 아무 일도 하지 않는다
    - 예: optional.ifPresent(System.out::println)
- 옵셔널이 채워져 있으면 true, 비어 있으면 false를 반환하는 isPresent 메서드들은 신중히 사용해야 한다.
    - isPresent를 사용하면 if문으로 null 체크하는 로직과 별반 차이가 없다.
    - 앞서 언급한 메서드들고 상당 수 대체할 수 있고 더 짧고 명확한 코드를 작성할 수 있다

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " + (parentProcess.isPresent() ?
    String.valueOf(parentProcess.get().pid()) : "N/A"));
```

- 위 코드를 Optional의 map으로 변환 가능

```java
System.out.println("부모 PID: " + 
    ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

### Optional에 stream()

- 스트림을 사용한다면 옵셔널들을 Stream<Optional<T>>로 받아서, 그중 채워진 옵셔널들에서 값을 뽑아  Stream<T>에 건네 담아 처리하는 경우도 있다.

```java
// Stream<Optional<String>> 타입의 스트림
Stream<Optional<String>> streamOfOptionals = Stream.of(
    Optional.of("value1"),
    Optional.empty(),
    Optional.of("value2")
);

// 옵셔널에 값이 있다면 그 값을 꺼내 스트림에 매핑
Stream<String> stringStream = streamOfOptionals
    .filter(Optional::isPresent)
    .map(Optional::get)
```

- 자바9에선 옵셔널에 stream() 메서드가 추가되었다.
    - 옵셔널을 스트림으로 변환해주는 어댑터
- 이를 flatMap과 조합하면 다음과 같이 바꿀 수 있다.

```java
 Stream<Optional<String>> streamOfOptionals = Stream.of(
           Optional.of("A"),
           Optional.empty(),
           Optional.of("B"),
           Optional.empty(),
           Optional.of("C")
       );

      List<String> result = streamOfOptionals
           .flatMap(Optional::stream)  // Stream<String>으로 변환
           .collect(Collectors.toList());  // ["A", "B", "C"]

```

### Optional 무조건 이득인가?

- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다.
- Optional<T>는 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T>를 반환한다.
- Optional의 대가
    - 새로 할당하고 초기화해야 하는 객체
    - 그 안에서 값을 꺼내려면 메서드를 호출하는 단계를 추가로 거침

### int, long, double전용 옵셔널 클래스

- Optional<Integer> 말고 OptionalInt, OptionalLong, OptionalDouble을 사용하자.

### Optional 부적절한 상황

- 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다.
- 인스턴스 필드로 Optinal을 갖는 경우는 나쁘지만 가끔 적절한 상황도 있다.

```java
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)
    private final int servings;     // (회, 총 n회 제공량)
    private final int calories;     // (1회 제공량당)
    private final int fat;          // (g/1회 제공량)
    private final int sodium;       // (mg/1회 제공량)
    private final int carbohydrate; // (g/1회 제공량)
```

- 위 같은 경우는 필수가 아니고 기본타입이라 값을 없음을 나타낼 방법이 마땅치 않기 때문에 선택적 필드의 게터 메서드들이 옵셔널을 반환하게 해주면 좋았다. 아니면 필드 자체를 옵셔널로 했어도 괜챃다

### (부록) Optional을 메서드의 파라미터로 넣는 건 안티패턴

- Optional을 메서드의 파라미터로 넣는 건 안티패턴이다.
1. 메서드에 조건부 로직을 유발한다
- isPresent()에 따른 조건부 로직이 생긴다

```java
public void saveUser(Optional<User> user) {
    if (user.isPresent()) {
        // ...
    }
}
```

1. 파라미터에 누군가가 null로 전달할 위험이 있다.
- Optional을 사용하면 Optional 객체안의 null값만 신경쓰지 Optional 자체가 null인지는 신경을 안쓰게 된다.
- 그에 따라 null 체크를 쉽게 간과할 수 있고 NPE가 발생할 위험이 있다.
- 즉, box 자체가 null일 수 있고, box 안의 값이 비어있을 수도 있는 두 가지 케이스를 모두 고려해야 한다.

```java
 public void processBox(Optional<Box> box) {
      // if(box == null) // 없으면 NPE 발생 가능성이 큼
        box.ifPresentOrElse(
            it -> System.out.println(it.fruit()),
            () -> System.out.println("Box is empty")
        );
    }
```

- 애초에 Optional은 **결과**가 없을 수도 있다는 것을 나타내고, null을 안전하게 사용하기 위해 만들어진 컨테이너 객체다.

참고

- https://c-king.tistory.com/m/590
- https://yeonyeon.tistory.com/224

### 핵심 정리

```
💡

호출할 때마다 반환값이 없을 가능성이 있는 메서드라면 옵셔널을 선택해야 할 수도 있다.
성능에 많이 민감하다면 Optional보단 null을 반환하거나 예외를 던지는 편이 나을 수 있다.

그리고 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다.

```