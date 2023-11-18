# 아이템 62. 다른 타입이 적절하다면 문자열 사용을 피하라

### 문자열은 다른 값 타입을 대신하기에 적합하지 않다.
데이터가 수치형이라면 int, float 등을 사용하고 예/아니오 형태라면 enum, boolean을 사용하면 좋다.
다룰 데이터가 정말로 문자열인 경우에만 사용하는 것이 좋다.
적절한 타입이 없다면 새로 하나 정의하는 것도 좋다.

### 문자열은 열거 타입을 대신하기에 적합하지 않다.
문자열로 상수를 열거할 때는 enum 타입이 월등히 낫다.

### 문자열은 혼한 타입을 대신하기에 적합하지 않다.
~~~Java
String compoundKey = className + "#" + i.next();
~~~
위의 코드에서 만일 className에 구분 문자(#)이 포함되면 파싱 과정에서 오류가 발생할 것이다.
또 느리고, 귀찮으며 오류 가능성도 커진다.
또한 String이 제공하는 기능에 의존해야 하는 단점이 존재한다.
이럴 때는 private 정적 멤버 클래스로 새로 만드는 것이 낫다.

### 문자열은 권한을 표현하기에 적합하지 않다.
권한(capacity)를 문자열로 표현하는 경우가 종종 있다. 문자열로 키를 사용하는 예제이다.
~~~Java
public class ThreadLocal {
    private ThreadLocal() { } // 객체 생성 불가

    // 현재 스레드의 값을 키로 구분해 저장한다.
    public static void set(String key, Object value);

    // 키가 가리키는 현재 스레드의 값을 반환한다.
    public static Object get(String key);
}
~~~
[ThreadLocal에 대한 설명](https://jake-seo-dev.tistory.com/543)

이 방법의 문제는 스레드를 구분하는 문자열 키가 전역에서 공유된다는 것이다.
의도대로 동작하려면 각 클라이언트가 고유한 키를 제공해야 한다. 그렇지 않으면 의도와 다르게 같은 변수를 공유하게 된다.
따라서 문자열로 권한을 구분하는 것이 아니라 별도의 타입을 만들어야 한다.
~~~Java
public class ThreadLocal {
  private ThreadLocal() {} // 생성자 막기

  // 내부 `Key` 클래스를 이용하여 인스턴스에 무관한 키를 생성할 수 있게 만든다.
  public static class Key { // (권한)
    Key() { }
  }

  // 위조 불가능한 고유 키를 생성한다.
  public static Key getKey() {
    return new Key();
  }

  // 현 스레드의 값을 키로 구분해 저장한다.
  public static void set(Key key, Object value);
  // 현 스레드의 값을 반환한다.
  public static Object get(Key key);
}
~~~

## 정리
- 문자열을 쓰기 전에 문자열보다 더 적절한 데이터 타입이 있는지 생각해보자.
- 문자열은 잘못 사용하면 번거롭고, 더 느리고, 오류 가능성도 크다.
- 기본, 열거, 혼합에 문자열 사용을 자제하자.