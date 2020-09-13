## 36 비트 필드 대신 EnumSet을 사용하라

열거한 값들이 주로 (단독이 아닌) 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해 왔다.

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0;
    public static final int STYLE_ITALIC = 1 << 1;
    public static final int STYLE_UNDERLINE = 1 << 2;
    public static final int STYLE_STRIKETHROUGH = 1 << 3;
    
    public void applyStyles(int styles) {
        ...
    }
}
```

<br />

다음과 같은 식으로 비트별 `OR`을 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 `비트 필드(bit field)`라 한다.

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

<br />

비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다. 하지만 비트 필드는 정수 열거 상수의 단점을 그대로 지니며, 추가로 다음과 같은 문제까지 안고 있다.

비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다. 마지막으로, 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선택해야 한다. API를 수정하지 않고는 비트 수를 더 늘릴 수 없기 때문이다.

<br />

`java.util` 패키지의 `EnumSet` 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해 준다. `Set` 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 `Set` 구현체와도 함께 사용할 수 있다. 하지만 `EnumSet`의 내부는 비트 벡터로 구현되었다. 원소가 총 64개 이하라면, 즉 대부분의 경우에 `EnumSet` 전체를 `long` 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여 준다.

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    public void applyStyles(Set<Style> styles) {
        ...
    }
}

text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

