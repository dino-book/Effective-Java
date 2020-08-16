## 08 finalizer와 cleaner 사용을 피하라

자바는 두 가지 객체 소멸자를 제공한다. `finalizer`는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다. 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 한다. `cleaner`는 `finalizer`보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

<br />

`finalizer`와 `cleaner`는 즉시 수행된다는 보장이 없다. 객체에 접근할 수 없게 된 후 `finalizer`와 `cleaner`가 실행되기까지 얼마나 걸릴지 알 수 없다. 즉 이 둘로는 제때 실행되어야 하는 작업은 절대 할 수 없다. 예컨대 파일 닫기를 맡기면 중대한 오류를 일으킬 수 있다. 시스템이 동시에 열 수 있는 파일 개수에 한계가 있기 때문이다.

<br />

클래스에 `finalizer`를 달아 두면 그 인스턴스의 자원 회수가 제멋대로 지연될 수 있다. 불행히도 `finalizer` 스레드는 다른 애플리케이션 스레드보다 우선순위가 낮아서 실행될 기회를 제대로 얻지 못한다. 자바 언어 명세는 어떤 스레드가 `finalizer`를 수행할지 명시하지 않으니 이 문제를 예방할 보편적인 해법은 없다. 딱 하나, `finalizer`를 사용하지 않는 것이다. 한편, `cleaner`는 자신을 수행할 스레드를 제어할 수 있다는 면에서 조금은 낫다. 하지만 여전히 백그라운드에서 실행되며 가비지 컬렉터의 통제 하에 있으니 즉각 수행되리라는 보장은 없다.

자바 언어 명세는 `finalizer`나 `cleaner`의 수행 시점뿐 아니라 수행 여부조차 보장하지 않는다. 접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수도 있다는 얘기다. 따라서 프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 `finalizer`나 `cleaner`에 의존해서는 안 된다.

<br />

`finalizer` 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다. 잡지 못한 예외 때문에 해당 객체는 자칫 마무리가 덜 된 상태로 남을 수 있다. `finalizer`와 `cleaner`는 심각한 성능 문제도 동반한다.

<br />

`finalizer`를 사용한 클래스는 `finalizer` 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다. 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 `finalizer`가 수행될 수 있게 된다. 이 `finalizer`는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다. 객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, `finalizer`가 있다면 그렇지도 았다. `final`이 아닌 클래스를 `finalizer` 공격으로부터 방어하려면 아무 일도 하지 않는 `finalize` 메소드를 만들고 `final`로 선언한다.

<br />

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    
    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다.
    private static class State implements Runnable {
        int numJunkPiles; // 방(Room) 안의 쓰레기 수
        
        State(in numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }
        
        // close 메소드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }
    
    // 방의 상태. cleanable과 공유한다.
    private final State state;
    
    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;
    
    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }
    
    @Override public void close() {
        cleanable.clean();
    }
}
```

`close()` 메소드에서 `Cleanable`의 `clean()`을 호출하면 이 메소드에서 `run()`을 호출한다. 혹은 가비지 컬렉터가 `Room`을 회수할 때까지 클라이언트가 `close()`를 호출하지 않는다면 `cleaner`가 `State`의 `run()` 메소드를 호출해 줄 것이다.

`State` 인스턴스는 절대로 `Room` 인스턴스를 참조해서는 안 된다. `Room` 인스턴스를 참조할 경우 순환참조가 생겨 가비지 컬렉터가 `Room` 인스턴스를 회수해 갈 기회가 오지 않는다. `State`가 정적 중첩 클래스인 이유가 여기 있다. 정적이 아닌 중첩 클래스는 자동으로 바깥의 참조를 갖기 때문이다. 이와 비슷하게 람다 역시 바깥 객체의 참조를 갖기 쉬우니 사용하지 않는 것이 좋다.