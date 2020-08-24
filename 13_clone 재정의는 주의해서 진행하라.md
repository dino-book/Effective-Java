## 13 clone 재정의는 주의해서 진행하라

`Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 `믹스인 인터페이스(mixin interface)`지만, 의도한 목적을 제대로 이루지 못했다. 가장 큰 문제는 `clone` 메소드가 선언된 곳이 `Cloneable`이 아닌 `Object`이고, 그마저도 `protected`라는 데 있다. 그래서 `Cloneable`을 구현하는 것만으로는 외부에서 `clone()` 메소드를 호출할 수 없다.

`Cloneable` 인터페이스는 `Object`의 `clone()` 메소드의 동작 방식을 결정한다. `Cloneable`을 구현한 클래스에서 `clone()`을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedException`을 던진다.

<br />

`Cloneable`을 구현한 클래스는 `clone()` 메소드를 `public`으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. 이 기대를 만족시키려면 그 클래스와 모든 상위 클래스는 복잡하고, 강제할 수 없고, 허술하게 기술된 프로토콜을 지켜야만 하는데, 그 결과로 깨지기 쉽고, 위험하고, 모순적인 메커니즘이 탄생한다. 생성자를 호출하지 않고도 객체를 생성할 수 있게 되는 것이다.

<br />

`clone()` 메소드의 일반 규약은 다음과 같다.

> 이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
>
> `x.clone() != x`
>
> 또한 다음 식도 참이다.
>
> `x.clone().getClass() == x.getClass()`
>
> 하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다.
>
> 한편, 다음 식도 일반적으로 참이지만 역시 필수는 아니다.
>
> `x.clone().equals(x)`
>
> 관례상 이 메소드가 반환하는 객체는 `super.clone()`을 호출해 얻어야 한다. 이 클래스와 (`Object`를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
>
> `x.clone().getClass() == x.getClass()`
>
> 관례상 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 `super.clone()`으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

강제성이 없다는 점만 빼면 생성자 연쇄와 비슷한 메커니즘이다. 즉, `clone()` 메소드가 `super.clone()`이 아닌 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 불평하지 않을 것이다. 하지만 이 클래스의 하위 클래스에서 `super.clone()`을 호출한다면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 `clone()` 메소드가 제대로 동작하지 않게 된다. `clone()`을 재정의한 클래스가 `final`이라면 걱정해야 할 하위 클래스가 없으니 이 관례는 무시해도 안전하다. 하지만 `final` 클래스의 `clone()` 메소드가 `super.clone()`을 호출하지 않는다면 `Cloneable`을 구현할 이유도 없다. `Object`의 `clone()` 동작 방식에 기댈 필요가 없기 때문이다.

<br />

제대로 동작하는`clone()` 메소드를 가진 상위 클래스를 상속해 `Cloneable`을 구현했을 때, `super.clone()`은 원본의 완벽한 복제본일 것이다. 클래스에 정의된 모든 필드는 원본 필드와 똑같은 값을 가진다. 모든 필드가 기본 타입이거나 불변 객체를 참조한다면, 이 객체는 완벽히 우리가 원하는 상태라 더 손볼 것이 없다. 그런데 쓸데없는 복사를 지양한다는 관점에서 보면 굳이 `clone()` 메소드를 제공하지 않는 게 좋다.

```java
@Override public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

이 메소드가 동작하게 하려면 `PhoneNumber` 클래스 선언에 `Cloneable`를 구현한다고 추가해야 한다.

<br />

앞서의 구현이 클래스가 가변 객체를 참조하는 순간 무너진다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
    
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.size(elements, 2 * size + 1)
        }
    }
}
```

`clone()` 메소드가 단순히 `super.clone()`의 결과를 그대로 반환한다면, 반환된 `Stack` 인스턴스의 `size` 필드는 올바른 값을 갖겠지만 `elements` 필드는 원본 `Stack` 인스턴스와 똑같은 배열을 참조할 것이다. 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다는 이야기다. 따라서 프로그램이 이상하게 동작하거나 `NullPointerException`을 던질 것이다.

<br />

`clone()` 메소드는 사실생 생성자와 같은 효과를 낸다. 즉, `clone()`은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다. 그래서 `Stack`의 `clone()` 메소드는 제대로 동작하려면 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 `elements` 배열의 `clone()`을 재귀적으로 호출하는 것이다.

```java
@Override public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

<br />

`elements` 필드가 `final`이었다면 앞선 방식은 작동하지 않는다. `final` 필드에는 새로운 값을 할당할 수 없기 때문이다. 이는 근본적인 문제로, 직렬화와 마찬가지로 `Cloneable` 아키텍처는 '가변 객체를 참조하는 필드는 `final`로 선언하라'는 일반 용법과 충돌한다.

<br />

`clone()`을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있다.

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
    
    @Override public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = buckets.clone();
            return result;
        } catch (CloneNotSupportException e) {
            throw new AssertionError();
        }
    }
}
```

복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다. 이를 해결하려면 각 버킷을 구성하는 연결 리스트를 복사해야 한다.

<br />

다음은 일반적인 해법이다.

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }
    }
    
    @Override public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++) {
                if (buckets[i] != null) {
                    result.buckets[i] = buckets.deepCopy();
                }
            }
            
            return result;
        } catch (CloneNotSupportException e) {
            throw new AssertionError();
        }
    }
}
```

`HashTable`의 `clone()` 메소드는 먼저 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순회하며 비지 않은 각 버킷에 대해 깊은 복사를 수행한다. 이때 `Entry`의 `deepCopy()` 메소드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다.

<br />

하지만 연결 리스트를 복제하는 방법으로는 그다지 좋지 않다. 재귀 호출 때문에 리스트의 연결 원소 수만큼 스택 프레임을 소비하여, 리스트가 길면 스택 오버플로를 일으킬 위험이 있기 때문이다. 이 문제를 피하려면 `deepCopy()`를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정해야 한다.

```java
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next)
    }
    
    return result;
}
```

<br />

`super.clone()`을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메소드를 호출하는 방법도 있다. `HashTable` 예에서라면, `buckets` 필드를 새로운 버킷 배열로 초기화한 다음, 원본 테이블에 담긴 모든 키-값 쌍에 대해 복제본 테이블의 `put()` 메소드를 호출해 둘의 내용이 똑같게 해 주면 된다.

이처럼 고수준 API를 활용해 복제하면 보통은 간단하고 제법 우아한 코드를 얻게 되지만, 아무래도 저수준에서 바로 처리할 때보다는 느리다. 또한 `Cloneable` 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회하기 때문에 `Cloneable` 아키텍처와는 어울리지 않는 방식이기도 하다.

<br />

생성자에서는 재정의될 수 있는 메소드를 호출하지 않아야 하는데, `clone()` 메소드도 마찬가지다. 만약 `clone()`이 하위 클래스에서 재정의한 메소드를 호출하게 되면 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 크다. 따라서 앞 문단에서 언급한 `put()` 메소드는 `final`이거나 `private`이어야 한다. `Object`의 `clone()`은 `CloneNotSupportedException`을 던진다고 선언했지만 재정의한 메소드에서는 그렇지 않다. `public`인 `clone()` 메소드에서는 `throws` 절을 없애야 한다. 검사 예외를 던지지 않아야 그 메소드를 사용하기 편하기 때문이다.

<br />

상속용 클래스는 `Cloneable`을 구현해서는 안 된다. `Object`의 방식을 따라해 `clone()` 메소드를 구현해 `protected`로 두고 `CloneNotSupportedException`을 던질 수 있다 선언할 수 있다. `clone()`을 동작하지 않게 구현해 두고 하위 클래스에서 재정의하지 못하게 할 수도 있다. 그러나 `Cloneable`을 구현한 스레드 안전 클래스를 작성할 때난 `clone()` 메소드 역시 적절히 동기화해 줘야 한다. `Object`의 `clone()` 메소드는 동기화를 신경 쓰지 않았다. 그러니 `super.clone()` 호출 외에 다른 할 일이 없더라도 `clone()`을 재정의하고 동기화해 줘야 한다.

<br />

`Cloneable`을 이미 구현할 클래스를 확장한다면 어쩔 수 없이 `clone()`을 잘 작동하도록 구현해야 한다. 그렇지 않은 상황에서는 복사 생성자와 복사 팩토리라는 더 나은 객체 복사 방식을 제공할 수 있다. 복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.

```java
public Yum(Yum yum) {
    ...
}
```

<br />

복사 팩토리는 복사 생성자를 모방한 정적 팩토리다.

```java
public static Yum newInstance(Yum yum) {
    ...
}
```

복사 생성자와 복사 팩토리는 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다는 장점도 있다.