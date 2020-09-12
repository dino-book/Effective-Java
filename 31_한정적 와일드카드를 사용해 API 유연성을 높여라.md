## 31 한정적 와일드카드를 사용해 API 유연성을 높여라

매개변수화 타입은 불공변이다. 즉, 서로 다른 타입 `Type1`과 `Type2`가 있을 때 `List<Type1>`은 `List<Type2>`의 하위 타입도 상위 타입도 아니다. `List<String>`은 `List<Object>`의 하위 타입이 아니라는 뜻이다. `List<Object>`에는 어떤 객체든 넣을 수 있지만 `List<String>`에는 문자열만 넣을 수 있다. 즉, `List<String>`은 `List<Object>`가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다(리스코프 치환 원칙에 어긋난다).

<br />

```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}

public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
```

이 메소드는 깨끗이 컴파일되지만 완벽하지 않다. `Iterable src`의 원소 타입이 스택의 원소 타입과 일치하면 잘 동작한다. 하지만 `Stack<Number>`로 선언한 후 `pushAll(intVal)`을 호출하면(`intVal`은 `Integer` 타입이다.)  에러가 발생한다. 매개변수화 타입이 불공변이기 때문이다.

<br />

자바는 이런 상황에 대처할 수 있는 한정적 와일드카드 타입이라는 특별한 매개변수화 타입을 지원한다. `pushAll()`의 입력 매개변수 타입은 '`E`의 `Iterable`'이 아니라 '`E`의 하위 타입의 `Iterable`'이어야 하며, 와일드카드 타입 `Iterable<? extends E>`가 정확히 이런 뜻이다.

```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```

<br />

`pushAll()`과 짝을 이루는 `popAll()` 메소드를 작성해 보자.

```java
public void popAll(Collection<E> dst) {
    while(!isEmpty()) {
        dst.add(pop());
    }
}
```

이번에도 마찬가지로 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 일치한다면 말끔히 컴파일된다. 하지만 `Stack<Number>`의 원소를 `Object`용 컬렉션으로 옮기고자 할 경우 컴파일 에러가 발생한다.

<br />

이번에는 `popAll()`의 입력 매개변수의 타입이 '`E`의 `Collection`'이 아니라 '`E`의 상위 타입의 `Collection`'이어야 한다. 와일드카드 타입을 사용한 `Collectino<? super E>`가 정확히 이런 의미다.

```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
} 
```

<br />

유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라. 한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다. 타입을 정확히 지정해야 하는 상황으로, 이때는 와일드카드 타입을 쓰지 말아야 한다.

- **펙스(PECS)** : producer-extends, consumer-super

즉, 매개변수화 타입 `T`가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<? super T>`를 사용한다. `Stack`의 예에서 `pushAll()`의 `src` 매개변수는 `Stack`이 사용할 인스턴스를 생산하므로 `src`의 적절한 타입은 `Iterable<? extends E>`이다. 한편, `popAll()`의 `dst` 매개변수는 `Stack`으로부터 `E` 인스턴스를 소비하므로 `dst`의 적절한 타입은 `Collection<? super E>`이다.