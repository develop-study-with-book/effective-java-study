## 정확한 답이 필요하다면 float와 double은 피하라

### float과 double

- float과 double 타입은 과학과 공학 계산용으로 설계되었다.
    - 이진 부동 소수점 연산
    - 넓은 범위의 수를 빠르게 정밀한 ‘근사치’로 계산
- float과 double 타입은 정확한 계산을 요구하는 금융 관련 계산과는 맞지 않다.

```java
System.out.println(1.03 - 0.42);
// 예상값: 0.61
// 출력값: 0.61000000000001
```

```java
System.out.println(1.00 - 9 * 0.10);
// 예상값: 0.10
// 출력값: 0.099999999999998
```

```
💡
위와 같은 결과가 발생하는 이유
컴퓨터는 모든 숫자를 이진수로 저장한다.
0.1은 이진수로 무한 소수가 된다. (0.0001100110011...)
컴퓨터는 무한한 공간이 없다. 그래서 double 타입에 ‘제한된 공간’만 준다.
64비트 안에 모든 것을 표현해야 하다 보니, 무한히 반복되는 이진수를 어느 시점에서 잘라서 정확한 값이 아닌 '근사값'이 저장되는 것이다.
```

- 반올림을 하면 되는거 아니야 라고 생각할 수 있지만 그래도 틀릴 수 있다. 아래 예시를 보자

```java
// 1달러를 가지고 있고 10센트부터 시작해서 + 10센트씩 늘려가면서
// 구매할 수 있는 개수와 잔돈을 출력 
// 0.1 + 0.2 + 0.3 + 0.4 = 1.0
public static void main(String[] args) {
   double funds = 1.00;
   int itemsBought = 0;
   for (double price = 0.10; funds >= price; price += 0.10) {
       funds -= price;
       itemsBought++;
   }
   System.out.println(itemsBought + "개 구입");
   System.out.println("잔돈(달러):" + funds);
  // 출력결과(잘못된 결과)
  // 3개 구입
  // 잔돈(달러):0.3999999999999999
}
```

- 0.1 + 0.2 + 0.3 + 0.4 = 1.0 총 4개를 구매하고 잔돈은 0이 나와야 맞다.
- 하지만 0.3999999999999999 결과가 나왔다. 이런 경우는 반올림을 해도 정확한 값을 구할 수 없다.

### BigDecimal

- 금융 계산에서는 BigDecimal, int 혹은 long을 사용해야 한다.

```java
    public static void main(String[] args) {
        final BigDecimal TEN_CENTS = new BigDecimal(".10");

        int itemsBought = 0;
        BigDecimal funds = **new BigDecimal("1.00");**
        for (BigDecimal price = TEN_CENTS;
            funds.compareTo(price) >= 0;
            price = price.add(TEN_CENTS)) {
            funds = funds.subtract(price);
            itemsBought++;
        }
        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + funds);
// 출력결과
// 4개 구입
// 잔돈(달러): 0.00
    }
```

- 앞선 코드에서 double 타입을 BigDecimal로 교체만 했는데 정확한 답이나왔다.

#### BigDecimal 단점

- 기본 타입보다 쓰기가 훨씬 불편하고, 훨씬 느리다

#### int 혹은 long

- BigDecimal 대안으로 int 혹은 long 타입을 쓸 수 있지만, 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 한다.
- 이번 예에서는 모든 계산을 달러 대신 센트로 수행하면 해결 된다.

```java
public static void main(String[] args) {
   int itemsBought = 0;
   int funds = 100;
   for (int price = 10; funds >= price; price += 10) {
       funds -= price;
       itemsBought++;
   }
   System.out.println(itemsBought + "개 구입");
   System.out.println("잔돈(센트): " + funds);
}
```

### 핵심 정리                                                                                                                       ㅁ

```
💡
정확한 답이 필요한 계산에서는 float나 double을 피하라.
코딩시의 불편함이나 성능 저하를 신경쓰지 않고 소수점을 다루고 싶으면 BigDecimal을 사용하라. 반올림 모드를 기가 막히게 지원해준다.
반면, 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라. 
아홉자리 십진수면 int, 열여덟 자리 십진수면 long, 열 여덞자리가 넘어가면 BigDecimal을 사용하라
```
