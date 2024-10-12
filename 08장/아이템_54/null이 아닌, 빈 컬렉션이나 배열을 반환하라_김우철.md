## null이 아닌, 빈 컬렉션이나 배열을 반환하라

### 컨테이너(컬렉션, 배열 등)가 비어있으면 null을 반환하지 말자

#### 나쁜예

```java
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null
           : new ArrayList<>(cheesesInStock);
}
```

- 컬렉션이 비어 있으면 null을 반환 하는 코드

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
    System.out.println("좋았어, 바로 그거야!");
```

- 메서드를 호출하는 클라이언트 쪽에서 null 처리 코드를 추가로 작성해줘야 해서 좋지 않다.

### 빈컨테이너 비용이 있으니 null을 반환한다는 주장

- 빈 컨테이너 할당시 비용이 드니 null 반환이 낫다 라는 주장도 있다
- 두가지 면에서 틀렸다.

#### 성능

- 위 정도의 성능 차이는 신경 쓸 수준이 못된다.

#### 할당
- 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다

- 우선 일반적인 방법은 아래와 같이 하면 된다.
```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
           : new ArrayList<>(cheesesInStock);
}
```
- 위 방법은 새로 할당하지 않고 반환하는 방법이다.
- Collections.emptyList()을 사용하여 빈 ‘불변’ 컬렉션을 반환하면 된다

### 배열

- 배열을 사용할 때도 null을 반환하지 말고 길이가 0인 배열을 반환하라

```java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

- 성능이 떨어질 것 같으면 배열을 미리 선언해두고 매번 같은 배열을 반환 하자

```java
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```
- 단순 성능 개선 목적이면 toArray에 넘기는 배열을 미리 할당하는 건 추천하지 않는다.
- 오히려 성능이 떨어진다는 연구 결과도 있다.

### 핵심정리

```
null이 아닌, 빈 배열이나 컬렉션을 반환하라. null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 성능도 좋은 것도 아니다.
```