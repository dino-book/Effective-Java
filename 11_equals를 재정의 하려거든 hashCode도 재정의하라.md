## 11 equals를 재정의 하려거든 hashCode도 재정의하라

`equals()`를 재정의한 모든 클래스에서 `hashCode()`도 재정의해야 한다. 그렇지 않으면 `hashCode()` 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 `HashMap`이나 `HashSet` 같은 컬렉션의 원소로 사용할 때 문제를 일으키게 될 것이다.

> - `equals()` 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 `hashCode()` 메소드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없다.
> - `equals()`가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode()`는 똑같은 값을 반환해야 한다.
> - `equals()`가 두 객체를 다르다고 판단했더라도, 두 객체의 `hashCode()`가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

`hashCode()` 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

<br />

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

m.get(new PhoneNumber(707, 867, 5309)); // null이 반환된다.
```

`PhoneNumber` 클래스는 `hashCode()`를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두 번째 규약을 지키지 못한다.

<br />

좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다. 이것이 바로 `hashCode()`의 세 번째 규약이 요구하는 속성이다. 이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다. 다음은 좋은 `hashCode()`를 작성하는 간단한 요령이다.

1. `int` 변수 `result`를 선언한 후 값 `c`로 초기화한다. 이떄 `c`는 해당 객체의 첫 번째 핵심 필드를 2.1 방식으로 계산한 해시코드다(여기서 핵심 필드란 `equals()` 비교에 사용되는 필드를 말한다).

2. 해당 객체의 나머지 핵심 필드 `f` 각각에 대해 다음 작업을 수행한다.

   1) 해당 필드의 해시코드 `c`를 계산한다.

      a. 기본 타입 필드라면, `Type.hashCode(f)`를 수행한다. 여기서 `Type`은 해당 기본 타입의 박싱 클래스다.

      b. 참조 타입 필드이면서 이 클래스의 `equals()` 메소드가 이 필드의 `equals()` 메소드를 재귀적으로 호출해 비교한다면, 이 필드의 `hashCode()`를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 `hashCode()`를 호출한다. 필드의 값이 `null`이면 0을 사용한다.
   
      c. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.2 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0)를 사용한다. 모든 원소가 핵심 원소라면 `Arrays.hashCode()`를 사용한다.
   
   2) 단계 2.1에서 계산한 해시코드 `c`로 `result`를 갱신한다.
   
   > result = 31 * result + c;
   
3. `result`를 반환한다.

<br />

단계 2.2의 곱셈 `31 * result`는 필드를 곱하는 순서에 따라 `result` 값이 달라지게 한다. 그 결과 클래스에 비슷한 필드가 여러 개일 때 해시 효과를 크게 높여 준다. 곱할 숫자를 31로 정한 이유는 31이 홀수이면서 소수이기 때문이다. 만약 이 숫자가 짝수이고 오버플로가 발생한다면 정보를 잃게 된다. 2를 곱하는 것은 시프트 연산과 같은 결과를 내기 때문이다.

```java
@Override public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

<br />

`Object` 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해 주는 정적 메소드인 `hash`를 제공한다. 이 메소드를 활용하면 앞서의 요령대로 구현한 코드와 비슷한 수준의 `hashCode()` 메소드를 단 한 줄로 작성할 수 있다. 하지만 아쉽게도 속도는 더 느리다. 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야 하기 때문이다. 그러니 `hash` 메소드는 성능에 민감하지 않은 상황에서만 사용해야 한다.

```java
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

<br />

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다. 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해 둬야 한다. 해시의 키로 사용되지 않는 경우라면 `hashCode()`가 처음 불릴 때 계산하는 지연 초기화 전략을 사용해도 좋다. 필드를 지연 초기화 하려면 그 클래스를 스레드 안전하게 만들도록 신경 써야 한다.

```java
private int hashCode;

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

<br />

성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다. 속도야 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수 있다.