## 09 try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에는 `close()` 메소드를 호출해 직접 닫아 줘야 하는 자원이 많다. 자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 한다.

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 `try-finally`가 쓰였다.

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

자원이 둘 이상이면 `try-finally` 방식은 너무 지저분하다. 뿐만 아니라 여기에는 미묘한 결점이 있다. 

예외는 `try` 블록과 `finally` 블록 모두에서 발생할 수 있는데, 예컨대 기기에 물리적인 문제가 발생한다면 `firstLineOfFile()` 메소드 안의 `readLine()` 메소드가 예외를 던지고, 같은 이유로 `close()` 메소드도 실패할 것이다. 이런 상황이라면 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다. 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 된다.

<br />

이러한 문제들은 자바 7이 투척한 `try-with-resources` 덕에 모두 해결되었다. 이 구조를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야 한다.

```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
    OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(bufl)) >= 0) {
            out.write(buf, 0, n);
        }
    }
    
}
```

`try-with-resources` 버전이 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다. `readLine()`과 `close()` 호출 양쪽에서 예외가 발생한다면, `close()`에서 발생한 예외는 숨겨지고 `readLine()`에서 발생한 예외가 기록된다. 이렇게 숨겨진 예외들도 그냥 버려지지는 않고, 스택 추적 내역에 숨겨졌다는 꼬리표를 달고 출력된다. 또한 자바 7에서 `Throwable`에 추가된 `getSusppressed` 메소드를 이용하면 프로그램 코드에서 가져올 수도 있다.