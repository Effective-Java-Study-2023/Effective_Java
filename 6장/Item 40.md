# Item 40 - @Override 애너테이션을 일관되게 사용하라

## @Override

<hr>

메서드 선언이 상위 유형의 메서드 선언을 재정의하기 위한 것임을 알려준다. 메서드에 해당 애너테이션이 달린 경우 컴파일러는 다음 조건 중 하나 이상이 충족되지 않는 한 오류 메시지를 생성해야 한다.

* 메서드는 상위 유형에 선언된 메서드를 재정의하거나 구현한다.
* 메서드에는 Object에 선언된 모든 공용 메서드의 시그니처가 재정의된다.

아래는 실제 Override의 구현 코드이다.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

* `@Target(ElementType.METHOD)`: 애너테이션의 적용 범위이다. `ElementType.METHOD`는 메서드에만 적용이 가능하다.
* `Retention(RetentionPolicy.SOURCE)`: Java 컴파일러가 애너테이션을 다루는 방법을 기술한다. 즉 어느 시점 까지 영향을 미치는지 결정한다. `RetentionPolicy.SOURCE`의 경우 컴파일 전까지 유효하다. 컴파일 이후에는 사라진다. 그렇기 때문에 컴파일 이후 빌드된 바이트코드에서는 `@Override`존재를 확인할 수 없다.
* `@interface`: 애너테이션 타입 정의를 위해 작성한다.

## @Override를 선언하지 않은 메서드

<hr>

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == this.first &&
                bigram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size()); //260
    }
}
```

Bigram이라는 문자 2개를 갖는 클래스에 a~z까지 26개의 문자를 넣고, 10개씩 만든다음에 HashSet에 삽입했다.
생각해보면 26개가 나올 것 같지만, 실제로는 260개가 발생한다.

왜 그럴까?

HashSet은 내부적으로 equals 메서드를 기반으로 객체의 논리적 동치적(equals) 검사를 실시한다.
하지만 자세히 보면 equals메서드의 파라미터 타입이 Bigram이다. equals 메서드를 재정의 한게 아니라 Overloading 한 꼴이다.

equals를 재정의 하려면 파라미터 타입이 Object이어야 한다.


```java
public class Object {
    ...
    public boolean equals(Object obj) {
        return (this == obj);
    }
    ...
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
    ...
}
```

```java
@Override
public boolean equals(Bigram bigram) {
        return bigram.first == this.first &&
        bigram.second == this.second;
        }
```

위와 같이 변경하고 컴파일 해보면,

    Error:(15, 5) java: method does not override or implement a method from a supertype

이러한 컴파일 에러가 발생한다.

컴파일러는 해당 메서드를 재정의한 메서드로 인식하지 않고 `오버로딩`한 것으로 인식한다.


잘못된 부분을 명확히 알려주므로 곧장 수정할 수 있다.


## 메서드 재정의를 위한 조건

<hr>

* 상속 관계에서 발생한다.
* 접근 제어자, 리턴 타입, 매개변수가 모두 일치해야 한다.
* 접근 제어자는 확장은 가능하지만 축소는 불가능하다. 예를들어 상위 클래스에 접근 제어자가 `protected`인 메서드를 재정의할 경우 `protected`, `public`으로 가능하다.


사실 `@Override`를 붙이지 않는다고 재정의가 불가능한 것은 아니다. 앞서 언급한 것 처럼 애너테이션은 단순히 컴파일러에게 알려주기 위한 주석일 뿐이고 자체적으로 특정 기능을 수행하는 것은 아니다.

```Java
@Override
public boolean equals(Object bigram) {
    if(!(o instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) bigram;
    return b.first == this.first &&
        b.second == this.second;
}
```

없어도 정상적으로 동작하지만 책에서는 @Override를 달 것을 권장한다. 컴파일 시점에 `재정의한 메서드가 잘못 작성되지 않았는지 확인`할 수 있을 뿐만 아니라 `개발자들에게 해당 메서드가 재정의 되었다는 것을 명시`적으로 알려준다.

## 정리

`@Override`는 컴파일 시점에 우리에게 `많은 정보를 제공`해준다. 해당 메서드가 재정의된 메서드임을 확인할 수 있고, 메서드를 잘못 작성한다면 친절하게 오류 메시지를 보여준다.

그러니 `상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 명시`하자. 추가적으로 IDE를 사용하면 편리하게 상위 클래스의 추상 메서드들을 재정의할 때 자동적으로 명시하도록 도와준다.