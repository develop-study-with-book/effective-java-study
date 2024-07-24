## 아이템 34. int 상수 대신 열거 타입을 사용하라

### 열거타입 이전

#### 정수 열거 패턴
열거 타입을 지원하기 전에는 정수 열거 패턴을 사용하였는데, 상당히 취약한 방법이다.

```java
public static final int APPLE_FUJI         = 0;
public static final int APPLE_PIPPIN       = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

단점
- 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.
- 깨지기 쉽다

#### 문자열 열거 패턴

```java
public static final int APPLE_FUJI = "APPLE_FUJI";
```

- 문자열 하드코딩은 오타가 발생해도 컴파일러가 체크해주지 못해 런타임 버그를 유발한다.

### 열거 패턴

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
```

- 실제로는 클래스이고, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
- 싱글턴을 보장해준다.

## 장점

- 컴파일타임 타입 안전성을 제공한다.
    - 다른 타입의 값을 넘기려면 컴파일 오류가 난다
- **각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다**
- 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.
    - 상수 이름을 문자열로 반환
    - 재정의도 가능

    ```java
     //재정의한 toString, 이름을 소문자로 출력한다.
        @Override
        public String toString() {
            return this.name().toLowerCase();
        }
    ```

- 열거 타입에 임의의 메서드나 필드를 추가할 수 있고, 임의의 인터페이스를 구현하게 할 수도 있다.

#### 데이터와 메서드를 갖는 열거 타입

```java
public enum Planet {
    MERCURY(3.302e+23,2.439e6),
    VENUS(4.869e+24,6.052e6),
  ;

    private final double mass;
    private final double radius;
    //표면중력
    private final double surfaceGravity;

    //중력상수
    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {
        return mass;
    }

    public double radius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}

```

- 열거 타입 상수 각각을 특정 데이터와 연결 지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장한다.
- 열거 타입은 불변이라 필드를 final로 선언하고 public 보단 private으로 선언하자.

#### 열거타입의 이모저모
- 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values 메서드 제공 ( 선언된 순서대로 제공 )
- 열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성된다.
- 열거 타입에서 상수를 하나 제거해도 해당 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다.
    - 참조하는 클라이언트에는 컴파일 오류가발생
- 열거 타입에서 제공해주는 기능들은 접근 제한자를 이용해서 제한적으로 이용해라
    - 같은 패키지내에서만 사용한다면 package-private 으로 선언
- 널리 쓰이는 열거 타입은 톱 레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만들어라

#### 상수별 메서드 구현

```java
public enum Operation {
    PLUS,MINUS,TIMES,DIVDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVDE:
                return x / y;
        }
        throw new AssertionError("알 수 없는 연산:" + this);
    }
}
```
- 마지막 throw 문은 도달할 일이 없지만 기술적으로 도달할 수 있기 때문에 선언을 해줘야 한다
- 위 방식의 깨지기 쉽다
    - 새로운 상수를 추가하면 case문도 추가해줘야 한다
    - case문 추가를 깜빡하면 런타임 에러 발생

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
}
```
- 상수 별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공
- 추상 메서드를 선언하고 상수에서 자신에 맞게 재정의
    - 상수 추가시 반드시 정의해줘야 하기 때문에 깜빡할 일이 없다.

```java
private static final Map<String, Operation> stringToEnum =
            Stream.of(Operation.values())
                    .collect(Collectors.toMap(Operation::toString, operation -> operation));

//Optional로 반환하여 값이 존재하지않을 상황을 클라이언트에게 알린다.
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```
- stringToEnum에 값이 생성되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때다.
- 열거 타입의 toString 메서드를 재정의할꺼면, 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공해주자.
  - toString 메서드가 열거 타입 상수를 문자열로 변환하는 반면, fromString 메서드는 그 반대로 문자열을 열거 타입 상수로 변환하기 위해서이다.

```java
enum Ensemble {
    ONE, TWO;

    private static final Map<String, Color> map = new HashMap<>();

    Ensemble() {
    // 컴파일 에러 발생
        map.put(this.name(), this);
    }
}

```
- **열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다.**
- 위는 컴파일 에러가 발생한다
- 열거 타입의 상수가 초기화되는 동안, 열거 타입의 인스턴스가 완전히 준비되지 않았기 때문이다.
- 정적 블록을 사용하여 초기화가 완료된 후 추가할 수 있다.
    - 열거 타입이 모두 초기화 된 후 static 블럭이 실행된다.

```java
enum Ensemble {
    ONE, TWO, THREE;

    private static final int VALUE = 42; // 상수 변수
    private static int counter = 0; // 상수 아닌 변수

    Ensemble() {
        System.out.println(VALUE); // OK
        System.out.println(counter); // 컴파일 에러 발생
        System.out.println(TWO); // 컴파일 에러 발생
    }
}
```
- **열거 타입의 정적 필드 중 열거 타입의 생성자에서 접근할 수 있는 것은 상수 변수뿐이다.**
- 상수가 아닌 counter는 접근할 수 없다.
    - 상수 풀에 저장된 값들은 컴파일 시에 이미 결정되어 있지만, 상수가 아닌 변수들은 런타임에 초기화되기 때문이다.
        - 열거 타입이 초기화 되는 동안 counter가 초기화가 안되었을 수도 있기 때문에 컴파일 오류 발생
- 생성자에서 같은 열거 타입의 상수에도 접근할 수 없다.
    - ONE 상수의 생성자가 호출될 때, TWO 상수는 아직 초기화되지 않았을 수 있기 때문에, 컴파일 단에서 오류가 발생한다

### 전략 열거 타입 패턴

- 상수별 메서드 구현에는 열거 타입 상수 끼리 코드를 공유하기 어렵다.
- 공유할 일이 생기는 경우 전략 열거 타입 패턴을 사용하자

```java
public enum PayrollDay {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        //기본 급여
        int basePay = minutesWorked * payRate;
		//잔업수당
        int overtimePay;
        switch (this) {
        	//주말
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            //주중
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }
}
```

위 코드의 단점
- 새로운 열거 타입을 추가하려면 switch문에 case문을 추가해야 한다.

상수별 메서드를 구현하여 제공할때 3가지 방법
  - 계산 코드를 모든 상수에 중복해서 넣기
    - 중복 코드 발생하여 로직 변경시 모두 수정해줘야 한다.
  - 계산 코드를 평일용과 주말용으로 나눠 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 호출
    - (괜찮지 않나..?)
  - 평일 잔업수당 계산 메소드를 구현해두고 주말 상수에서만 재정의해서 쓰는 방법
    - 주말용인 새로운 상수 추가시 재정의를 하지 않으면 평일용 코드를 물려 받아서 잘못된 계산이 될 수 있다.


해결책
- 중첩 열거 타입을 활용하여 새로운 상수를 추가할 때 ‘전략’을 선택하도록 하는 방법이 있다.

```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked,payRate);
    }

    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    
```
- 잔업 수당 계산을 private 중첩 열거 타입으로 옮긴다(PayType)
- PayrollDay 열거 타입의 생성자에서 이 중 적당한 것을 선택한다
- PayrollDay 열거 타입은 잔업수단 계산을 전략 열거 타입에 위임하여, switch 문이나 상수별 메서드 구현이 필요 없게 된다.
- switch문 보다 복잡하지만 더 안전하고 유연하다

#### switch문이 좋은 선택일 때

- switch 문은 열거 타입의 상수별 동작을 구현하는데는 적합하지 않다.
- 그러나, 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch문이 좋은 선택이 될 수 있다.
    - 상수별로 상수를 반환할 때..?

```java
public static Operation inverse(Operation operation) {
        switch (operation) {
            case PLUS:
                return Operation.MINUS;
            case MINUS:
                return Operation.PLUS;
            case TIMES:
                return Operation.DIVDE;
            case DIVDE:
                return Operation.TIMES;
        }
        throw new AssertionError("알 수 없는 연산 : " +operation);
}
```
- inverse 메서드는 Operation 열거 타입의 반대 연산을 반환하는 메서드

### 열거 타입을 언제 사용하나요

- 고정된 집합으로 정의되는 상수 집합일 때
    - 태양계 행성, 한 주의 요일, 체스 말
- 제한된 값들만 허용하여 코드의 안전성과 가독성을 높이고자 할
    - 메뉴 아이템, 연산 코드, 명령줄 플래그

- 열거 타입의 성능은 정수 상수와 별반 다르지 않다.
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.
- 열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었다.

```text
💡 열거 타입은 정수 상수보다 읽기 쉽고 안전하다. 열거 타입은 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때 필요하다. 드물게 하나의 메서드가 상수별로 다르게 동작해야 할 때는, switch문 대신 상수별 메서드 구현을 사용하자. 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.
```
