## 81 wait와 notify보다는 동시성 유틸리티를 애용하라

`wait()`와 `notify()`는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용한다.

`java.util.concurrent`의 고수준 유틸리티는 세 범주로 나눌 수 있다. `실행자 프레임워크`, `동시성 컬렉션`, `동기화 장치`다.

<br />

동시성 컬렉션은 `List`, `Queue`, `Map` 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다. 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다. 따라서 동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.

동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 일 역시 불가능하다. 그래서 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메서드들이 추가되었다.

동시성 컬렉션은 동기화한 컬렉션을 낡은 유산으로 만들어 버렸다. 대표적인 예로, 이제는 `Collections.synchronizedMap`보다는 `ConcurrentHashMap`을 사용하는 게 훨씬 좋다.

<br />

컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 (즉, 차단되도록) 확장되었다. 예를 들어 `Queue`를 확장한 `BlockingQueue`에 추가된 메서드 중 `take`는 큐의 첫 원소를 꺼낸다. 이때 만약 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다. 이런 특성 덕에 `BlockingQueue`는 작업 큐(생산자-소비자 큐)로 쓰기에 적합하다. 작업 큐는 하나 이상의 생산자 스레드가 작업을 큐에 추가하고, 하나 이상의 소비자 스레드가 큐에 있는 작업을 꺼내 처리하는 형태다. `ThreadPoolExecutor`를 포함한 대부분의 실행자 서비스 구현체에서 이 `BlockingQueue`를 사용한다.

<br />

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해 준다. 가장 자주 쓰이는 동기화 장치는 `CountDownLatch`와 `Semaphore`다. `CyclicBarrier`와 `Exchanger`는 그보다 덜 쓰인다. 그리고 가장 강력한 동기화 장치는 바로 `Phaser`다.

카운트다운 래치는 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다. `CountDownLatch`의 유일한 생성자는 `int` 값을 받으며, 이 값이 래치의 `countDown` 메서드를 몇 번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다.

<br />

```java
// 동시 실행 시간을 재는 간단한 프레임워크
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CoutnDownLatch(concurrency);
    
    for (int = 0; i < concurrency; i++) {
        executor.execute(() -> {
            // 타이머에게 준비를 마쳤음을 알린다.
            ready.countDown();
            try {
                // 모든 작업자 스레드가 준비될 때까지 기다린다.
                start.await();
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                // 타이머에게 작업을 마쳤음을 알린다.
                done.countDown();
            }
        });
    }
    
    ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
    long startNanos = System.nanoTime();
    start.countDown(); // 작업자들을 깨운다.
    done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
    return System.nanoTime() - startNanos;
}
```

`ready` 래치는 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지할 때 사용한다. 통지를 끝낸 작업자 스레드들은 두 번째 래치인 `start`가 열리기를 기다린다. 마지막 작업자 스레드가 `ready.countDown()`을 호출하면 타이머 스레드가 시작 시각을 기록하고 `start.countDown()`을 호출하여 기다리던 작업자 스레드들을 깨운다. 그 직후 타이머 스레드는 세 번째 래치인 `done`이 열리기를 기다린다. `done` 래치는 마지막 남은 작업자 스레드가 동작을 마치고 `done.countDown()`을 호출하면 열린다. 타이머 스레드는 `done` 래치가 열리자마자 깨어나 종료 시각을 기록한다.

`time()` 메서드에 넘겨진 실행자는 `concurrency` 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다. 그렇지 못하면 이 메서드는 결코 끝나지 않을 것이다. 이런 상태를 `스레드 기아 교착상태(thread starvation deadlock)`라 한다. `InterruptException`을 캐치한 작업자 스레드는 `Thread.currentThread().interrupt()` 관용구를 사용해 인터럽트를 되살리고 자신은 `run()` 메서드에서 빠져나온다. 이렇게 해야 실행자가 인터럽트를 적절하게 처리할 수 있다.

<br />

새로운 코드라면 언제나 `wait()`와 `notify()`가 아닌 동시성 유틸리티를 써야 한다. 하지만 어쩔 수 없이 레거시 코드를 다뤄야 할 때도 있을 것이다. `wait()` 메서드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다. 락 객체의 `wait()` 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다.

```java
// wait() 메서드를 사용하는 표준 방식
synchronized (obj) {
    while (<조건이 충족되지 않았다>) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
    }
    
    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

<br />

`wait()` 메서드를 사용할 때는 반드시 대기 반복문 관용구를 사용하라. 반복문 밖에서는 절대로 호출하지 않는다. 이 반복문은 `wait()` 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다.

대기 전에 조건을 검사하여 조건이 이미 충족되었다면 `wait()`을 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치다. 만약 조건이 이미 충족되었는데 스레드가 `notify()` 혹은 `notifyAll()` 메서드를 먼저 호출한 후 대기 상태로 빠지면, 그 스레드를 다시 깨울 수 있다고 보장할 수 없다.

한편, 대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는 것은 안전 실패를 막는 조치다. 만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 락이 보호하는 불변식을 깨뜨릴 위험이 있다. 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 몇 가지 있으니, 다음이 그 예다.

- 스레드가 `notify()`를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.
- 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 `notify()`를 호출한다. 공개된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출된다.
- 깨우는 스레드는 지나치게 관대해서, 대기 중인 스레드 중 일부만 조건이 충족되어도 `notifyAll()`을 호출해 모든 스레드를 깨울 수도 있다.
- 대기 중인 스레드가 (드물게) `notify()` 없이도 깨어나는 경우가 있다.