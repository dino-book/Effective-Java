## 14 Comparable을 구현할지 고려하라

`compareTo()`는 `Object`의 메소드가 아니라 `Comparable` 인터페이스의 메소드이다. 또한 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다. `Comparable`을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다. 그래서 `Comparable`을 구현한 객체들의 배열은 다음처럼 손쉽게 정렬할 수 있다.

```java
Arrays.sort(a);
```

<br />

`compareTo()` 메소드의 일반 규약은 `equals()`의 규약과 비슷하다.

> 이 객체외 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 `ClassCaseException`을 던진다.
>
> 다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수를 뜻한다. 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환한다.
>
> - `Comparable`을 구현한 클래스는 모든 x, y에 대해 `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`여야 한다(따라서 `x.compareTo(y)`는 `y.compareTo(x)`가 예외를 던질 때에 한해 예외를 던져야 한다).
> - `Comparable`을 구현한 클래스는 추이성을 보장해야 한다. 즉, `(x.compareTo(y) > 0 && y.compareTo(z))`이면 `x.compareTo(z) > 0`이다.
> - `Comparable`을 구현한 클래스는 모든 z에 대해 `x.compareTo(y) == 0`이면 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`다.
> - `(x.compareTo(y) == 0) == (x.equals(y))`여야 한다. 이는 필수는 아니지만 꼭 지키는 게 좋다.

위 세 규약은 `compareTo` 메소드로 수행하는 동치성 검사도 `equals` 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻한다. 그래서 주의사항도 똑같다. 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 `compareTo` 규약을 지킬 방법이 없다. 

마지막 규약은 `compareTo` 메소드로 수행한 동치성 테스트의 결과가 `equals`와 같아야 한다는 것이다. 이를 잘 지키면 `compareTo`로 줄지은 순서와 `equals`의 결과가 일관되게 된다. 

<br />

`Comparable`은 타입을 인수로 받는 제네릭 인터페이스이므로 `compareTo` 메소드의 인수 타임은 컴파일 타입에 정해진다. 입력 인수의 타입을 확인하거나 형변환할 필요가 없다는 뜻이다. 인수의 타입이 잘못됐다면 컴파일 자체가 되지 않는다.

`compareTo` 메소드는 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교한다. 객체 참조 필드를 비교하려면 `compareTo` 메소드를 재귀적으로 호출한다. `Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 `비교자(Comparator)`를 대신 사용한다.

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
}
```

<br />

클래스에 핵심 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해진다. 가장 핵심적인 필드부터 비교해나가자.

```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0) {
            result = Short.compare(lineNum, pn.lineNum);
        }
    }
    
    return result;
}
```

<br />

자바 8에서는 `Comparator` 인터페이스가 일련의 비교자 생성 메소드와 팀을 꾸려 메소드 연쇄 방식으로 비교자를 생성할 수 있게 되었다. 하지만 이는 약간의 성능 저하가 뒤따른다.

```java
private static final Comparator<PhoneNumber> COMPARATOR =
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenConparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

