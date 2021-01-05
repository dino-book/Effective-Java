## 54 null이 아닌, 빈 컬렉션이나 배열을 반환하라

```java
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}


List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON)) {
    System.out.println("성공");
}
```

코드가 `null`을 반환한다면, 클라이언트는 이 `null` 상황을 처리하는 코드를 추가로 작성해야 한다. 컬렉션이나 배열 같은 컨테이너가 비었을 때 `null`을 반환하는 메서드를 사용할 때면 항시 이와 같은 방어 코드를 넣어 줘야 한다. 클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있다. 한편, `null`을 반환하려면 반환하는 쪽에서도 이 상황을 특별히 취급해 줘야 해서 코드가 더 복잡해진다.

<br />

때로는 빈 컨테이너를 할당하는 데도 비용이 드니 `null`을 반환하는 쪽이 낫다는 주장도 있다. 하지만 이는 두 가지 면에서 틀린 주장이다. 첫 번째, 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못 된다. 두 번째, 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

<br />

사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨린다면, 매번 똑같은 빈 '불변' 컬렉션을 반환한다. 단, 이 역시 최적화에 해당하니 꼭 필요할 때만 사용한다.

```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList() 
        : new ArrayList<>(cheesesInStock);
}
```

<br />

배열을 쓸 때도 마찬가지다. 길이가 0인 배열을 반환한다. 보통은 단순히 정확한 길이의 배열을 반환하기만 하면 된다. 그 길이가 0일 수도 있을 뿐이다.

```java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

<br />

이 방식이 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언해 두고 매번 그 배열을 반환하면 된다. 길이 0인 배열은 모두 불변이기 때문이다.

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

