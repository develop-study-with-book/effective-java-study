## 아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

### 매개변수화 타입 불공변

- 매개변수화 타입은 불공변이다.
- `List<String>`은 `List<Object>`의 하위 타입이 아니다.
  - `List<String>`은 `List<Object>`가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다(리스코프 치환 원칙에 어긋난다)
    - `List<String>`은 String타입만 넣을 수 있다.
    - `List<Object>`는 어떤 객체든 넣을 수 있다.

### 한정적 와일드 카드

```java
public class Stack<E>{
    public void pushAll(Iterable<E> src) {
        for (E e : src)
            push(e);
    }
}

Stack<Number> numberStack = new Stack();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);
```
- `Stack<Number>` 타입의 필드의 pushAll 메서드에 `Iterable<Integer>` 타입을 인자로 넣어줌
  - 이론적으로는 Integer는 Number의 하위 타입이니 잘 동작해야 한다.
  - 하지만 실제로는 에러가 발생하는데, 매개변수화 타입이 불공변이기 때문이다.

- **해결책 : 한정적 와일드카드 타입을 사용하자**
  - 지금 예시는 상한 경계 와일드 카드
```java
public void pushAll(**Iterable<? extends E>** src){
  for(E e : src)
    push(e);
}
Stack<Number> numberStack = new Stack();
Iterable<Integer> integers = ...;
        numberStack.pushAll(integers);
```

- `Iterable<? extends E>` → `Iterable<? extends Number>` → `Iterable<Integer extends Number>`
    - 컴파일 성공
    - **E의 Iterable**가 아니라 **E의 하위 타입의 Iterable** 라는 의미

```java
public void popAll(Collection<E> dst){
        while(!isEmpty())
            dst.add(pop());
    }
}

Stack<Number> numberStack2 = new Stack();
Collection<Object> integers2 = List.of(1, 2, 3);
numberStack2.popAll(integers2); // 컴파일 에러
```
- Collection<Object>는 Collection<Number>의 하위 타입이 아니다
    - 이전 예시와 똑같은 에러

- **해결책 : 한정적 와일드카드 타입을 사용하자**
  - 지금 예시는 하한 경계 와일드 카드

```java
public void popAll(**Collection<? super E>** dst){
        while(!isEmpty())
            dst.add(pop());
    }

Stack<Number> numberStack2 = new Stack();
Collection<Object> integers2 = List.of(1, 2, 3);
numberStack2.popAll(integers2); // 컴파일 에러
```
- 마찬가지로 popAll의 입력 매개변수의 타입이 **`E의 Collection`**이 아니라 **`E의 상위 타입의 Collection`**이어야 한다.

- `Collection<? super Object>` → `Iterable<? extends Object>` → `Iterable<Number extends Object>`
    - 컴파일 성공

- 위 예시의 메시지는 분명하다. **유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.**
  - **한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을게 없다.**
  - 타입을 정확히 지정해야 하는 상황으로, 와일드 카드 타입을 쓰지 말아야 한다
```java
   public static <E> void swap(List<E> list, int i, int j) {
        Collections.swap(list, i, j);
    }
```
⇒ swap 메서드는 리스트의 두 요소를 교환하는데, 두 인덱스 위치의 값을 가져오고(읽음), 두 인덱스 위치에 새로운 값을 설정(씀)한다.

```java
팩스(PECS) : producer-extends, consumer-super
```
- 다음 공식을 외워두면 어떤 와일드카드 타입을 써야 하는지 기억하는 데 도움이 될 것이다.

```
💡 위에서 본 pushAll 메서드는 Stack에서 이용하는 E 인스턴스를 생산하므로, Iterable<? extends E>.
popAll 메서드는 Stack으로부터 E 인스턴스를 소비하므로 Collection<? super E>이다.
```

- 생산자
  - 매개변수로 받은 것을 내가 가진 곳에 쌓는것, 더하는 것
- 소비자
  - 매개변수로 받은 것에 내가 가진걸 꺼내서 추가하는 것
**즉, 매개변수화 타입 T가 생산자라면 <? extends T>를 사용하고, 소비자라면 <? super T>를 사용하라.**

```java
class Stack<E> {
    public void pushAll(Interable<? extends E> src) {
        push(e);
    }
}
```
- <? extends E>로 되어 있으면 E라는 타입보다 하위 타입이 들어오기 때문에 추가해도 안전하다

```java
class Stack<E> {
  public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
      dst.add(pop());
  }
}
```
- <? super E>로 되어 있으면 E라는 타입보다 상위 타입이 들어오기 때문에 **내가 가진 E라는 타입을 꺼내서 매개변수에 추가**해도 안전하다.
  - E: Integer
  - 들어오는 것: Number

### 생성자 예시

#### 예제1
아이템 28의 Chooser 생성자는 다음과 같이 선언

```java
public Chooser(Collection<? extends T> choices) {
    choiceList = new ArrayList<>(choices);
}

List<Integer> intList = List.of(1, 2, 3, 4, 5, 6);
Chooser<Number> chooser = new Chooser<>(intList);
```
- 매개변수로 받은것 내가 가진 변수에 추가하는것 → 생산자(extends)

#### 예제2
- 한정적 와일드카드 타입 사용 전
```java
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
    
Set<Integer> integers3 = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers3, doubles); // 컴파일 에러 발생
```

- Integer와 Double는 Number의 하위 타입이지만..
- 그러나 union은 E로 설정한 Number 타입만 와야하기 때문에 유연하지 않다.
- union 메서드는 인자로 받은걸 추가하는것 -> 생산자(extends)

```java
public static <E> Set<E> union(Set<? extend E> s1, Set<? extends E> s2) {
     Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }

Set<Integer> integers3 = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers3, doubles);
```

```
💡 반환 타입은 여전히 Set<E>임에 주목하자. **반환 타입에는 한정적 와일드카드 타입을 사용하면 안된다.**  유연성을 높여주기는커녕 클라이언트 코드에서도 와일드카드 타입을 써야하기 때문이다.
```

- 자바7까지는 타입 추론 능력이 충분히 강력하지 못해서 문맥에 맞는 반환타입(혹은 목표타입)을 명시해야 했다.
```java
Set<Number> numbers = Union.**<Number>**union(integers, doubles);
```

```
💡 매개변수와 인수의 차이
매개변수는 메서드 선언에 정의한 변수이고, 인수는 메서드 호출 시 넘기는 '실젯값'
void add(int **value**) { ... }
add(**10**)
value는 매개변수, 10은 인수

class Set<**T**> { ... }
Set<**Integer**> = ...;
T는 타입 매개변수, Integer는 타입 인수
```

# 와일드 카드 타입 고급

- 기존 max 메서드
```java
public static <E extends Comparable<E>> E max(List<E> list)
```
- `E extends Comparable<E>` 는 재귀적인 한정 타입

- 수정된 max 메서드

```java
public static <E extends **Comparable<? super E>**> E max(**List<? extends E>** list)
```
- PECS 공식 두번 허용
- 매개변수로 받는 list는 리스트를 다양하게 받기 위해  `? extends E` 적용

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
- Comparable는 안에 있는 값을 꺼내서 비교하기 때문에 소비자(super)

- 수정하는게 더 날까? 그렇다.
- 책 예시는 어려워서 강의 예시로 대체

```java
public class Box<T extends Comparable<T>> implements Comparable<Box<T>> {
---
public class IntegerBox extends Box<Integer> {
---
public static <E extends **Comparable<? super E>**> E max(**List<? extends E>** list)
---
    public static void main(String[] args) {
        List<IntegerBox> list = new ArrayList<>();
        list.add(new IntegerBox(10, "keesun"));
        list.add(new IntegerBox(2, "whiteship"));

        System.out.println(max(list));
    }
```
- IntegerBox가 Comparable을 구현하지 않았지만 max를 사용할 수 있다
- 이유는 IntegerBox의 상위 타입인 Box<T>가 Comparable<Box<T>>을 구현했기 때문이다.
- `public static **<IntegerBox extends Comparable<? super IntegerBox>>** IntegerBox max(List<? extends IntegerBox> list)`
    - `<IntegerBox extends Comparable<? super IntegerBox>>` << 해석 해보자면..
        - `? super IntegerBox` == Box
          - IntegerBox extends Comparable<Box>
            - IntegeBox가 Comparable<Box>를 상속한다

---


- 타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다.
- swap 메서드의 두 가지 선언

```java
public static **<E>** void swap(**List<E> list**, int i, int j); // 1. 비한정적 타입 매개변수
---
public static void swap(**List<?> list**, int i, int j)     // 2. 비한정적 와일드 카드
```
- public API라면 간단한 두번째가 낫다.
- 어떤 리스트든 이 메서드에 넘기면 명시한 인덱스의 원소들을 교환해줄것이다. 신경 써야 할 타입 매개변수도 없다.

- 기본 규칙
```
💡 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.
이때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.
```

- 와일드 카드 타입으로했을 때 list. set할 때 컴파일 에러
- **와일드 카드 타입일 때 null 외에는 어떤 값도 넣을 수 없다.**
    - 나는 어떤 타입인지 모르니까 값을 넣을 수 없어!!라는 의미

```
- ?는 비한정적 와일드 카드
  - 타입을 모른다!! 라는 의미
  - 물음표 하나만 쓰는 경우 null 밖에 허용 안함
  - 물음표 하나만 쓰는경우는 권장하지 않는다.
    - 값을 꺼내는건 가능하지만 넣는 건 못함
      - `list.set(j, list.get(i))`
        - list.set은 불가능, list.get은 가능
        - 물음표 타입이 뭔지 모르니까 set을 못함
- **특정한 타입(E)를 쓰던가 혹은 와일드 카드는 한정적 타입으로만 써라**
```

- 해결법은 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용하는 방법이다.
    - 굳이 이렇게까지..?
- 실제 타입을 알아내려면 이 도우미 메서드는 제네릭 메서드여야 한다.

```java
		public static void swap(List<?> list, int i, int j){
        swapHelper(list, i, j);
    }

    // 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
    private static <E> void swapHelper(List<E> list, int i , int j){
        list.set(i, list.set(j, list.get(i)));
    }
```
- swapHelper 메서드는 리스트가 List<E>임을 알고 있다.
- **즉, 이 리스트에서 꺼낸 값의 타입은 항상 E이고, E 타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있다.**
- swap 메서드를 호출하는 클라이언트는 복잡한 swapHelper의 존재를 모른 채 그 혜택을 누리는 것이다.
  - 굳이 이렇게까지..?

- 그냥 <E>로 사용하는게 날것같다..

### 핵심정리
```
💡 조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다. 그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야 한다. PECS 공식을 기억하자. 즉, 생산자(producer)는 extends를 / 소비자(consumer)는 super를 사용한다. Comparable과 Comparator는 모두 소비자라는 사실도 잊지 말자.
```

