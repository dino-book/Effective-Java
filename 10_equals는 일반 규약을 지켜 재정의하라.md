## 10 equals는 일반 규약을 지켜 재정의하라

`equals()` 메소드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있어서 자칫하면 끔찍한 결과를 초래한다. 문제를 회피하는 가장 쉬운 길은 아예 재정의하지 않는 것이다. 그냥 두면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다. 그러니 다음 상황 중 하나에 해당한다면 재정의하지 않는 것이 최선이다.

- **각 인스턴스가 본질적으로 고유하다.**

  값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다(예 - `Thread` 클래스).

- **인스턴스의 `논리적 동치성(logical equality)`을 검사할 일이 없다.**

- **상위 클래스에서 재정의한 `equals()`가 하위 클래스에도 딱 들어맞는다.**

- **클래스가 `private`이거나 `package-private`이고 `equals()` 메소드를 호출할 일이 없다.**

<br />

`객체 식별성(object identity; 두 객체가 물리적으로 같은가)`이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 `equals()`가 논리적 동치성을 비교하도록 재정의되지 않았을 때 `equals()`를 재정의한다. 주로 `Integer`나 `String`처럼 값을 표현하는 클래스들이 여기 해당한다.

값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 `equals`를 재정의하지 않아도 된다. `Enum`도 여기 해당한다. 이런 클래스에서는 어차피 놀리적으로 같은 인스턴스가 2개 이상 만들어지지 않으니 논리적 동치성과 객체 식별성이 사실상 같은 의미가 된다.

<br />

`equals()` 메소드를 재정의할 때는 반드시 일반 규약을 따라야 한다.

> equals() 메소드는 동치관계(equivalence relation)을 구현하며, 다음을 만족한다.
>
> - **반사성(reflexivity)** : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
> - **대칭성(symmetry)** : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
> - **추이성(transitivity)** : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 z.equals(x)도 true다.
> - **일관성(consistency)** : null이 아닌 모든 참조 값 x, y에 대해, x.eqauls(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
> - **null-아님** : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

<br />

`Object` 명세에서 말하는 동치관계란 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산이다. 이 부분집합을 `동치류(equivalence class; 동치 클래스)`라 한다. `equals()` 메소드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 교환할 수 있어야 한다.

<br />

**반사성**은 객체는 자기 자신과 같아야 한다는 뜻이다. 이 요건을 어긴 클래스의 인스턴스를 컬렉션에 넣은다음 `contains()` 메소드를 호출하면 방금 넣은 인스턴스가 없다고 답할 것이다.

<br />

**대칭성**은 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다. 다음은 대칭성을 위배한 코드를 보여 준다.

```java
public final class CaseInsensitiveString {
    private final String s;
    
    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
    
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return s.eqaulsIgnoreCase(
            (CaseInsonsitiveString o).s);
        }
        if (o instanceof String) {
            return s.equalsIgnoreCase((String) o);
        }
        return false;
    }
}

CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

`cis.equals(s)`는 true를 반환한다. 문제는 `CaseInsensitiveString`의 `eqauls()`는 일반 `String`을 알고 있지만 `String`의 `eqauls()`는 `CaseInsensitiveString`의 존재를 모른다는 데 있다. 따라서 `s.eqauls(cis)`는 false를 반환하여, 대칭성을 위반한다.

`equals()` 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다. 이 문제를 해결하려면 `CaseInsensitiveString`의 `equals()`를 `String`과도 연동하겠다는 꿈을 버려야 한다.

```java
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).equalsIgnoreCase(s);
}
```

<br />

**추이성**은 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같으면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻이다. 아래는 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 예시를 보여 준다. `equals()` 비교에 영향을 주는 정보를 추가한 것이다.

```java
public class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

```java
public class ColorPoint extends Point {
    private final Color color;
    
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}

Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

위 코드는 대칭성을 위배한다. 일반 `Point`를 `ColorPoint`에 비교한 결과와 그 둘을 바꿔 비교한 결과가 다를 수 있다. `Point`의 `equals()`는 색상을 무시하고, `ColorPoint()`의 `equals()`는 입력 매개변수의 클래스 종류가 다르다며 매번 false만 반환할 것이다. `p.equals(cp)`는 true를, `cp.equals(p)`는 false를 반환한다.

<br />

```java
@Override public boolean equals(Object o) {
    if (!(o instanceof Point)) {
        return false;
    }
    
    // o가 일반 Point면 색상을 무시하고 비교한다.
    if (!(o instanceof ColorPoint)) {
        return o.equals(this);
    }
    
    // o가 ColorPoint면 색상까지 비교한다.
    return super.equals(o) && ((ColorPoint) o).color == color;
}


ColorPoint(p1) = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint(p3) = new ColorPoint(1, 2, Color.BLUE);
```

이 방식은 대칭성은 지켜 주지만, 추이성을 깨버린다. `p1.equals(p2)`와 `p2.equals(p3)`는 true를 반환하는데 `p3.equals(p1)`가 false를 반환한다. 또한 이 방식은 무한 재귀에 빠질 위험도 있다. `Point`의 하위 클래스로 `SmellPoint`를 만들고, `equals()`는 같은 방식으로 구현했다고 해 보자. 그런 다음 `myColorPoint.equals(mySmellPoint)`를 호출하면 `StackOverflowError`를 일으킨다.

사실 이 현상은 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제다. 구체 클래스를 확장해 새로운 값을 추가하면서 `equals()` 규약을 만족시킬 방법은 존재하지 않는다.

<br />

`equals()` 안의 `instanceof` 검사를 `getClass` 검사로 바꾸면 규약도 지키고 값도 추가하면서 구체 클래스를 상속할 수 있는 것처럼 보인다.

```java
@Override public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass()) {
        return false;
    }
    
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

`Point`의 하위 클래스는 정의상 여전히 `Point`이므로 어디서든 `Point`로 활용될 수 있어야 한다. 그런데 이 방식에서는 그렇지 못하다. 어떤 타입에 있어서 중요한 속성이면 그 하위 타입에서도 마찬가지로 중요하고, 따라서 그 타입의 모든 메소드가 하위 타입에서도 똑같이 잘 작동해야 하는 `리스코프 치환 원칙(Liskov substitution principle)`을 위반한다.

<br />

구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 하나 있다. "상속 대신 컴포지션을 사용하라"는 조언을 따르면 된다. `Point`를 상속하는 대신 `Point`를 `ColorPoint`의 `private` 필드로 두고, `ColorPoint`와 같은 위치의 일반 `Point`를 반환하는 뷰(view) 메소드를 `public`으로 추가하는 방식이다.

```java
public class ColorPoint {
    private final Point point;
    private final Color color;
    
    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }
    
    public Point asPoint() {
        return point;
    }
    
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.point.equals(color);
    }
}
```

<br />

**일관성**은 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻이다. 가변 객체는 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있는 반면, 불변 객체는 한번 다르면 끝까지 달라야 한다. 클래스를 불변으로 만들기로 했다면 `equals()`가 한번 같다고 한 객체와는 영원히 같다고 답하고, 다르다고 한 객체와는 영원히 다르다고 답하도록 만들어야 한다.

클래스가 불변이든 가변이든 `equals()`의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다. 이런 문제를 피하려면 `equals()`는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 한다.

<br />

**null-아님**은 이름처럼 모든 객체가 null과 같지 않아야 한다는 뜻이다. 동치성을 검사하려면 `equals()`는 건내받은 객체를 적절히 형변환한 후 필수 필드들의 값을 알아내야 한다. 그러려면 형변환에 앞서 `instanceof` 연산자로 입력 매개변수가 올바른 타입인지 검사해야 한다.

```java
// 명시적 null 검사 - 필요 없다.
@Override public boolean equals(Object o) {
    if (o == null) {
        return false;
    }
    ...
}


// 묵시적 null 검사 - 이쪽이 낫다.
@Override public boolean equals(Object o) {
    if (!(o instanceof MyType)) {
        return false;
    }
    MyType mt = (Mytype) o;
    ...
}
```

<br />

### 양질의 equals 메소드 구현 방법

1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

<br />

`float`와 `double`을 제외한 기본 타입 필드는 `==` 연산자로 비교하고, 참조 타입 필드는 각각의 `equals()` 메소드로, `float`와 `double` 필드는 각각 정적 메소드인 `Float.compare()`와 `Double.compare()`로 비교한다. 이 둘을 특별 취급하는 이유는 `Float.NaN`, `-0.0f`, 특수한 부동소수 값 등을 다뤄야 하기 때문이다.

<br />

앞서의 `CaseInsensitiveString` 예처럼 비교하기가 복잡한 필드를 가진 클래스도 있다. 이럴 때는 그 필드의 표준형을 저장해 둔 후 표준형끼리 비교하면 훨씬 경제적이다. 이 기법은 특히 불변 클래스에 제격이다. 가변 객체라면 값이 바뀔 때마다 표준형을 최신 상태로 갱신해 줘야 한다.

어떤 필드를 먼저 비교하느냐가 `equals()`의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교해야 한다. 동기화용 `락(lock)` 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안 된다.