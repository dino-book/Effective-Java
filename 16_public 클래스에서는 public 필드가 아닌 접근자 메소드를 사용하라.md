## 16 public 클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라

이따금 인스턴스 필드들을 모아놓는 일 외에는 아무 목적도 없는 클래스를 작성할 때가 있다.

```java
class Point {
    public double x;
    public double y;
}
```

<br />

이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다. API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다. 철저한 객체 지향 프로그래머는 이런 클래스를 상당히 싫어해서 필드들을 모두 `private`으로 바꾸고 `public 접근자(getter)`를 추가한다.

```java
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

<br />

하지만 `package-private` 클래스 혹은 `private` 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다. 그 클래스가 표현하려는 추상 개념만 올바르게 표현해 주면 된다. 이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다. 클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다. 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다.

<br />

`public` 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각이 아니다. API를 변경하지 않고서는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다. 단, 불변식을 보장할 수 있게 된다.

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    
    public final int hour;
    public final int minute;
    
    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IllegalArgumentException("시간: " + hour);
        }
        if (minute < 0 || minute >= MINUTE_PER_HOUR) {
            throw new IllegalArgumentException("분: " + minute);
        }
        this.hour = hour;
        this.minute = minute;
    }
}
```

