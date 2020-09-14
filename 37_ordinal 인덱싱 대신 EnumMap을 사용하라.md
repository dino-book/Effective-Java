## 37 ordinal 인덱싱 대신 EnumMap을 사용하라

이따금 배열이나 리스트에서 원소를 꺼낼 때 `ordinal()` 메소드로 인덱스를 얻는 코드가 있다.

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    
    final String name;
    final LifeCycle lifeCycle;
    
    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }
    
    @Override public String toString() {
        return name;
    }
}

// ordinal()을 배열 인덱스로 사용하는 잘못된 예시

Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

for (int i = 0; i < plantsByLifeCycle.length; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.println("%s: %s%n",
                      Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

동작은 하지만 문제가 한가득이다. 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다. 가장 심각한 문제는 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다는 점이다.

<br />

여기서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일을 한다. 그러니 `Map`을 사용할 수도 있을 것이다.

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = 
    new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
}

for (Plant p : garden) {
    plantsByLifeCycle.get(p.lifeCycle).add(p);
}

System.out.println(plantsByLifeCycle);
```

더 짧고 명료하고 안전하고 성능도 원래 버전과 비등하다. 안전하지 않은 형변환은 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없다. 나아가 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다. `EnumMap`의 성능이 `ordinal()`을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문이다. 내부 구현 방식을 안으로 숨겨서 `Map`의 타입 안전성과 배열의 성능을 모두 얻어낸 것이다.여기서 `EnumMap`의 생성자가 받는 키 타입의 `Class` 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다.

<br />

스트림을 사용해 맵을 관리하면 코드를 더 줄일 수 있다.

```java
// EnumMap을 사용하지 않는 스트림 코드 - EnumMap을 써서 얻은 공간과 성능 이점이 사라진다.
System.out.println(Arrays.stream(garden)
                  .collect(groupingBy(p -> p.lifeCycle)));

// EnumMap을 사용해 데이터와 열거 타입을 매핑하는 스트림 코드
System.out.println(Arrays.stream(garden)
                  .collect(groupingBy(p -> p.lifeCycle,
                         () -> new EnumMap<>(LifeCycle.class), toSet()))))
```

<br />

두 열거 타입 값들을 매핑하느라 `ordinal()`을 두 번이나 쓴 배열이 있을 수 있다.

```java
public enum Phase {
    SOLID, LUQUID, GAS;
    
    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        
        private static final Transitino[][] TRANSITIONS = {
            { null, MELT, SUBLINE },
            { FREEZE, null, BOIL },
            { DEPOSIT, CONDENSE, null }
        };
        
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()]
        }
    }
}
```

앞서 봤던 간단한 정원 예제와 마찬가지로 컴파일러는 `ordinal()`과 배열 인덱스의 관계를 알 도리가 없다. 즉, `Phase`나 `Phase.Transition` 열거 타입을 수정하면서 상전이표 `TRANSITIONS`를 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 날 것이다.

<br />

`EnumMap`을 사용하는 편이 훨씬 낫다. 전이 하나를 얻으려면 이전 상태(from)와 이후 상태(to)가 필요하니, 맵 2개를 중첩하면 쉽게 해결할 수 있다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
        
        private final Phase from;
        private final Phase to;
        
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }
        
        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
            m = Stream.of(values()).collect(groupingBy(t -> t.from,
               () -> new EnumMap<>(Phase.class),
               toMap(t -> t.to, t -> t,
                    (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

