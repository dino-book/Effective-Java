## 56 공개된 API 요소에는 항상 문서화 주석을 작성하라

API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다. 직렬화할 수 있는 클래스라면 직렬화 형태에 관해서도 적어야 한다.

<br />

메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다. 상속용으로 설계된 클래스의 메서드가 아니라면 (그 메서드가 어떻게 동작하는지가 아니라) 무엇을 하는지를 기술해야 한다. 문서화 주석에는 클라이언트가 해당 메서드를 호출하기 위한 전제조건을 모두 나열해야 한다. 또한 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건도 모두 나열해야 한다.

일반적으로 전제조건은 `@throws` 태그로 비검사 예외를 선언하여 암시적으로 기술한다. 비검사 예외 하나가 전제조건 하나와 연결되는 것이다. 또한 `@param` 태그를 이용해 그 조건에 영향받는 매개변수에 기술할 수도 있다.

전제조건과 사후조건뿐만 아니라 부작용도 문서화해야 한다. 부작용이란 사후조건으로 명확히 나타나지는 않지만 시스템의 상태에 어떠한 변화를 가져오는 것을 뜻한다. 예컨대 백그라운드 스레드를 시작시키는 메서드라면 그 사실을 문서에 밝혀야 한다.

<br />

메서드의 계약을 완벽히 기술하려면 모든 매개변수에 `@param` 태그를, 반환 타입이 `void`가 아니라면 `@return` 태그를, 발생할 가능성이 있는 모든 예외에 `@throws` 태그를 달아야 한다.

관례상 `@param` 태그와 `@return` 태그의 설명은 해당 매개변수가 뜻하는 값이나 반환값을 설명하는 명사구를 쓴다. 역시 관례상 `@param`, `@return`, `@throws` 태그의 설명에는 마침표를 붙이지 않는다.

```java
/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant time.
 * In some implementations it may run in time proportional to the
 * element position.
 * 
 * @param index - index of element to return; must be non-negative
 *        and less then the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException - if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int index);
```

 <br />

각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명으로 간주된다. 요약 설명은 반드시 대상의 기능을 고유하게 기술해야 한다. 헷갈리지 않으려면 한 클래스 안에 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안된다. 다중정의된 메서드가 있다면 특히 더 조심해야 한다.