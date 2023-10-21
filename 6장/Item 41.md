# Item 41 - 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시 해주는 인터페이스를 마커 인터페이스라고 한다.
Serializeable 인터페이스가 가장 좋은 예이다.

![image](https://github.com/Effective-Java-Study-2023/Effective_Java/assets/123082067/e3779251-719b-4d2a-837e-193578b9c140)

아무런 메서드도 없다.
단지 타입 정보만 알려주는 인터페이스이다.

자바에서 직렬화를 사용하고 싶은 클래스에 Serializable을 붙여주면된다.

    직렬화 : 자바 시스템 내부에서 사용되는 Object 또는 Data를 외부의 자바 시스템에서도 사용할 수 있도록 byte 형태로 데이터를 변환하는 기술

Serializable 인터페이스를 구현한 클래스는 ObjectOutputStream.writeObject 메서드를 통해 직렬화할 수 있다.

```Java
public final void writeObject(Object obj) throws IOException {
    if (enableOverride) {
        writeObjectOverride(obj);
        return;
    }
    try {
        writeObject0(obj, false);
    } catch (IOException ex) {
        if (depth == 0) {
            writeFatalException(ex);
        }
        throw ex;
    }
}
```

```Java
public void writeObject0(Object obj, boolean unshared) throws IOException {
    ...
    if (obj instanceof String) {
        writeString((String) obj, unshared);
    } else if (cl.isArray()) {
        writeArray(obj, desc, unshared);
    } else if (obj instanceof Enum) {
        writeEnum((Enum<?>) obj, desc, unshared);
    } else if (obj instanceof Serializable) {
        writeOrdinaryObject(obj, desc, unshared);
    } else {
        if (extendedDebugInfo) {
            throw new NotSerializableException(
                cl.getName() + "\n" + debugInfoStack.toString());
        } else {
            throw new NotSerializableException(cl.getName());
        }
    }
    ...
}
...
```

Serializable 을 구현하지 않은 객체를 직렬화하려고하면 다음과 같은 코드에서 에러가 발생한다.

이렇게 단순히 타입 체크 정도만하고 있어, 마커 인터페이스라 부르는 것이다.

마커 애너테이션(item 39)이 등장하면서 마커 인터페이스를 사용 안 할것 같지만 마커 인터페이스가 두 가지 더 나은점이 있다.

## 마커 인터페이스의  장점 1

<hr>

마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 그렇지 않다.

마커 인터페이스는 어엿한 타입이기 때문에, 마커 애너테이션을 사용했다면 런타임에야 발견될 오류를 컴파일타임에 잡을수 있다.

## 마커 인터페이스의  장점 2

<hr>

적용대상을 더 정밀하게 지정할 수 있다는 것이다.

적용대상을(@Target)을 ElementType.TYPE으로 선언한 애너테이션은 모든 타입(클래스, 인터페이스, 열거 타입, 애너테이션)에 달수있지만

특정 인터페이스를 구현한 클래스에만 적용하고 싶을때는 마커 인터페이스를 사용하면 된다.

## 마커 인터페이스를 언제 사용해야할까?

<hr>

클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수 등)에 마킹해야 할 때 애너테이션을 쓸수 밖에 없다.

마커를 클래스나 인터페이스에 적용해야 하고 이 마킹이된 객체를 매개변수로 받는 메서드를 작성할 일이 있다면 마커 인터페이스를 사용해서 컴파일타임에 오류를 잡아낼 수 있다.

위에서 살펴본 ObjectOutputStream.writeObject는 매개변수로 Serializable이 아닌 Object 객체를 받도록 설계되어 런타임시 문제를 확인하므로 이러한 마커 인터페이스의 장점을 살리지 못한 케이스이다.

## 정리

<hr>

- 마커 인터페이스와 마커 애너테이션은 각자의 쓰임이 있다.

- 새로 추가하는 메서드 없이 단지 타입 정의가 목적이라면 마커 인터페이스를 선택하자

- 마커 애너테이션을 사용할 때 @Target(ElementType.TYPE)인 마커 애너테이션을 작성하고 있다면,
  마커 애너테이션을 정말 사용해야 하는지? 마커 인터페이스를 사용할 수 있는지 생각해보고
  웬만하면 마커 인터페이스를 사용하도록 하자