## 34 int 상수 대신 열거 타입을 사용하라

```java
// 정수 열거 패턴 - 상당히 취약하다!
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

`정수 열거 패턴(int enum pattern)` 기법에는 단점이 많다. 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다. 오렌지를 건네야 할 메소드에 사과를 보내고 동등 연산자로 비교하더라도 컴파일러는 아무런 경고 메시지를 출력하지 않는다.

정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다. 평범한 상수를 나열한 것뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다. 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다. 다시 컴파일하지 않은 클라이언트는 실행이 되더라도 엉뚱하게 동작할 것이다.

<br />

정수 대신 문자열 상수를 사용하는 변형 패턴도 있다. `문자열 열거 패턴(string enum pattern)`이라 하는 이 변형은 더 나쁘다. 상수의 의미를 출력할 수 있다는 점은 좋지만, 경험이 부족한 프로그래머가 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만들기 때문이다. 이렇게 하드코딩한 문자열에 오타가 있어도 컴파일러는 확인할 길이 없으니 자연스럽게 런타임 버그가 생긴다. 문자열 비교에 따른 성능 저하 역시 당연한 결과다.

<br />

`열거 타입(enum type)`을 통해 열거 패턴의 단점을 씻어 줄 수 있다.

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

<br />

자바 열거 타입을 뒷받침하는 아이디어는 단순하다. 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다. 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 `final`이다. 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다. 다시 말해 열거 타입은 인스턴스 통제된다. 싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 거꾸로 열거 타입은 싱글턴을 일반화할 형태라고 볼 수 있다.

열거 타입은 컴파일 타임 타입 안전성을 제공한다. `Apple` 열거 타입을 매개변수로 받는 메소드를 선언했다면, 건네받은 참조는 (`null`이 아니라면) `Apple`의 세 가지 값 중 하나임이 확실하다. 다른 타입의 값을 넘기려 하면 컴파일 오류가 난다.

열거 타입에는 각자의 이름 공간이 있어서 이름이 같은 상수도 평화롭게 공존한다. 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다. 공개되는 것이 오직 필드의 이름뿐이라, 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다. 열거 타입에서 상수를 하나 제거하면, 제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다.

<br />

열거 타입에서는 상수마다 동작이 달라져야 하는 상황도 있을 것이다.

```java
// 값에 따라 분기하는 열거 타입

public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;
    
    public double apply(double x, double y) {
        switch(this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```

동작은 하지만 그리 예쁘지는 않다. 마지막의 `throw` 문은 실제로는 도달할 일이 없지만 기술적으로는 도달할 수 있기 때문에 생략하면 컴파일조차 되지 않는다. 더 나쁜 점은 깨지기 쉬운 코드라는 사실이다. 예컨대 새로운 상수를 추가하면 해당 `case` 문도 추가해야 한다.

<br />

다행히 열거 타입은 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다. 열거 타입에 `apply()`라는 추상 메소드를 선언하고 각 상수별 클래스 몸체, 즉 각 상수에서 자신에 맞게 재정의하는 방법이다. 이를 `상수별 메소드 구현(constant-specific method implementation)`이라 한다.

```java
public enum Operation {
    PLUS("+") { 
        public double apply(double x, double y) { return x + y; }
    }, 
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    }, 
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    }
    
    private final String symbol;
    
    Operation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override public String toString() {
        return symbol;
    }
    
    public abstract double apply(double x, double y);
}
```

<br />

열거 타입에는 상수 이름을 받아 그 이름에 해당하는 상수를 반환해 주는 `valueOf(String)` 메소드가 자동 생성된다. 한편, 열거 타입의 `toString()` 메소드를 재정의하려거든, `toString()`이 반환하는 문자열을 해당 열거 타입 상수로 변환해 주는 `fromString()` 메소드도 함께 제공하는 걸 고려해 보자. 다음 코드는 모든 열거 타입에서 사용할 수 있도록 구현한 `fromString()`이다(단, 타입 이름을 적절히 바꿔야 하고 모든 상수의 문자열 표현이 고유해야 한다).

```java
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(toMap(Object::toString, e -> e));

public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

<br />

한편, 상수별 메소드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

```java
// 값에 따라 분기하여 코드를 공유하는 열거타입

enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int baseDay = minutesWorked * payRate;
        
        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY:
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 :
                (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        
        return basePay + overtimePay;
    }
}
```

분명 간결하지만, 관리 관점에서는 위험한 코드다. 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 `case` 문을 잊지 말고 쌍으로 넣어줘야 하는 것이다.

상수별 메소드 구현으로 급여를 정확히 계산하는 방법은 두 가지다. 첫째, 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣으면 된다. 둘째, 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메소드로 작성한 다음 각 상수가 자신에게 필요한 메소드를 적절히 호출하면 된다. 두 방식 모두 코드가 장황해져 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.

<br />

가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 하는 것이다. 잔업수당 계산을 `private` 중첩 열거 타입으로 옮기고 `PayrollDay` 열거 타입의 생성자에서 이중 적당한 것을 선택한다. 그러명 `PayrollDay` 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, `switch` 문이나 상수별 메소드 구현이 필요 없게 된다. 이 패턴은 `switch` 문보다 복잡하지만 더 안전하고 유연하다.

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), 
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), 
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);
    
    private final PayType payType;
    
    PayrollDay(PayType payType) {
        this.payType = payType;
    }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };
        
        abstract int overtimePay(int mins, int payRate);
        
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

