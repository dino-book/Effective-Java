## 03 private 생성자나 열거 타입으로 싱글턴임을 보증하라

### public static final 방식의 싱글턴

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    
    public void leaveTheBuilding() { }
}
```

private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다. public이나 protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.

해당 클래스가 싱글턴임이 API에 명백히 드러나고, public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다. 간결하다는 장점도 존재한다.

<br />

### 정적 팩토리 방식의 싱글턴

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() {
        return INSTANCE;
    }
    
    public void leaveTheBuilding() { }
}
```

API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있는 유연성이 있다. 또한 정적 팩토리를 제네릭 싱글턴 팩토리로 만들 수 있다. 정적 팩토리 메소드 참조를 공급자로 사용할 수도 있다.

<br />

둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다. 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메소드를 제공해야 한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

<br />

### 열거 타입 방식의 싱글턴

```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() { }
}
```

대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다. 단, 만드려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.