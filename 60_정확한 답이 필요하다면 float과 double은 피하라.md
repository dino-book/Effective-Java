## 60 정확한 답이 필요하다면 float과 double은 피하라

`float`과 `double`은 과학과 공학 계산용으로 설계되었다. 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 세심하게 설계되었다. 따라서 정확한 결과가 필요할 때는 사용하면 안 된다. `float`과 `double`은 특히 금융 관련 계산과는 맞지 않는다. 0.1 혹은 10의 음의 거듭제곱수를 표현할 수 없기 때문이다.

<br />

예를 들어 주머니에 1.03달러가 있었는데 그중 42센트를 썼다고 해 보자. 남은 돈은 얼마인가?

```java
System.out.println(1.03 - 0.42);
```

안타깝게도 위 코드는 0.6100000000000001을 출력한다.

<br />

```java
public static void main(String[] args) {
    double funds = 1.00;
    int itemsBought = 0;
    for (double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemBought++;
    }
    
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```

프로그램을 실행해 보면 3개를 구입한 후 잔돈은 0.399999999999999달러가 남았음을 알게 된다. 이 문제를 올바로 해결하려면 금융 계산에는 `BigDecimal`, `int`, 혹은 `long`을 사용해야 한다.

<br />

또는 `float`이나 `double`을 사용하지 않는 단위로 바꾼다. 위 예시에선 달러 대신 센트로 단위를 바꿀 수 있다.

```java
public static void main(String[] args) {
    double funds = 100;
    int itemsBought = 0;
    for (double price = 10; funds >= price; price += 10) {
        funds -= price;
        itemBought++;
    }
    
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(센트): " + funds);
}
```

