## 전통적인 for 문보다는 for-each 문을 사용하라

### 전통적인 for문

```java
// 코드 58-1
// 전통적인 for문으로 컬렉션을 순회
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
   Element e = i.next();
   // e로 무언가를 한다.
}
```

```java
// 코드 58-2
// 전통적인 for문으로 배열을 순회
for (int i = 0; i < a.length; i++) {
   // a[i]로 무언가를 한다.
}
```

- while문 보다는 낫지만 원소들의 인덱스는 필요없고 순수 원소들 뿐 필요하다면 위 코드는 가장 베스트는 아니다
- 코드가 지저분해지고, 쓰이는 요소들이 늘어나면 잘못 쓸 가능성이 있다.

### for-each(향상된 for 문)

- 위 문제는 향상된 for 문(enhanced for statement)을 사용하면 해결된다.

```java
// 코드 58-3
// 컬렉션과 배열을 순회하는 올바른 관용구
for (Element e : elements) {
   // e로 무언가를 한다.
}
```

- 콜론(:)은 “안의(in)”라고 읽는다.
- 앞선 코드와 속도는 별다른 차이가 없다.

### 버그 찾기

- 향상된 for 문은 컬렉션을 중첩해 순회할 때 이점이 커진다
- 다음 코드에서 버그를 찾아보자

```java
// 코드 58-4
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, 
          NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
   for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
       deck.add(new Card(i.next(), j.next()));
```

- 버그
    - 바깥 컬렉션의 반복자에서 next 메서드가 너무 많이 불린다
    - 마지막 줄의 i.next()는 숫자 하나당 한번만 불려야 된다.
    - 따라서, 바깥쪽 반복문에서 호출됬어야 한다

```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
   Suit suit = i.next();
   for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
       deck.add(new Card(suit, j.next()));
}
```

- 해결 방법은 바깥 반복문의 바깥 원소를 저장하는 변수를 하나 추가한다.
- for-each문을 사용하면 간단히 해결된다.

```java
for (Suit suit : suits) // Suit 하나 선택
   for (Rank rank : ranks) // 선택된 Suit에 대해 모든 Rank 순회
       deck.add(new Card(suit, rank));
```

### for-each문을 사용할 수 없는 상황

#### 파괴적인 필터링(destructive filtering)

- 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다
- 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.

- Iterator 방식

```java
// 짝수 제거하기
Collection<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6));
for (Iterator<Integer> it = numbers.iterator(); it.hasNext();) {
    if (it.next() % 2 == 0) {
        it.remove();  // 반복자의 remove() 메서드 사용
    }
}
// 결과: [1, 3, 5]
```

- **for-each 문은 사용 불가**

```java
List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4));

// ConcurrentModificationException 발생!
for (Integer number : numbers) {
    if (number % 2 == 0) {
        numbers.remove(number);  // 순회 중 삭제 불가
    }
}

// for-each의 실제 내부 동작
Iterator<Integer> iterator = numbers.iterator();
    while (iterator.hasNext()) {
Integer number = iterator.next();
      if (number % 2 == 0) {
// (원인) Iterator가 아닌 List에서 직접 제거
        numbers.remove(number);  // List의 modCount 값이 증가
// Iterator가 저장한 modCount와 List의 실제 modCount가 달라짐
      }
              }
```

- 반복을 마치기 전에 요소를 제거하여 예외가 발생
  - `Iterator`는 컬렉션의 일관성을 보장하기 위해 `modCount`라는 변수를 사용
  - `numbers.remove(number);`에서 List의 modCount 값이 증가
  - Iterator가 저장한 modCount와 List의 실제 modCount가 달라짐
  - 다음 `iterator.next();` 에서 이를 인지하고 예외 발생

- 위 문제 해결책

1. removeIf

```java
Collection<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6));
numbers.removeIf(number -> number % 2 == 0);  // 짝수 제거
// 결과: [1, 3, 5]
```

1. iterator의 remove사용

```java
for (Iterator<Integer> iterator = integers.iterator(); iterator.hasNext();) {
    Integer integer = iterator.next();
    if(integer == 2) {
        iterator.remove();
    }
}
```

#### 변형(transforming)

- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

#### 병렬 반복(parallel iteration)

- 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다

### Iterable 인터페이스

- for-each 문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.

```java
public interface Iterable<E> {
   // 이 객체의 원소들을 순회하는 반복자를 반환한다.
   Iterator<E> iterator();
}
```

### 핵심 정리

```
💡
전통적인 for문과 비교했을 때 for-each 문은 명료하고, 유연하고, 버그를 예방해준다. 
성능 저하도 없다. 제한이 없다면 1순위로 for-each문 사용을 고려하자
```