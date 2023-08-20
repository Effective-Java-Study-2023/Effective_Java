# Item 3 - private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 싱글턴이란?

    인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

## 1. public static final

<hr>

```JAVA
public class Example {
	public static final Example INSTANCE = new Example();
	private Example() { ... }
}
```

private 생성자는 public static final 필드인 Example.INSTANCE를 초기화할 때 한번만 호출됨.

public 이나 protected 생성자가 없다.
> Example 클래스가 초기화될 때 만들어진 인스턴스가 하나뿐임이 보장됨.

예외
> 권한이 있는 클라이언트는 리플렉션 API(item 65)인 AccessibleObject.setAccessible 을 이용해 private 생성자 호출 가능.

방어
> 생성자를 수정하여 두 번째 객체가 생성될 때 예외를 던지게 하면 됨.

장점

    해당 클래스가 싱글턴임이 API에 명백히 드러남.
    public static 필드가 final 키워드 이기 때문에 다른 객체 참조 불가능.
    
    간결함.


## 2. 정적 팩터리 

<hr>

```JAVA
public class Example {
	private static final Example INSTANCE = new Example();
	private Example() { ... }
	public static Example getInstance() { return INSTANCE; }
}
```

Example.getInstance() 는 항상 같은 객체의 참조를 반환함.
> 인스턴스는 단 한번만 만들어지지만, 리플렉션을 통한 예외는 똑같이 적용됨.

장점

    API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있음.
    > 유일한 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있음.

    원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있음. (item 30)

    정적 팩터리 메서드 참조를 공급자(supplier)로 사용할 수 있음. (item 43, 44)
    > Example::getInstance를 Supplier<Example>로 사용할 수 있음.

```JAVA
// 제네릭 싱글턴 팩터리
public static UnaryOperator<Object> IDENTIFY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<Object> identifyFunction() {
    return (UnaryOperator<T>) IDENTIFY_FN;
}
```
```JAVA
// 공급자 (supplier)
public calss ExampleClass {
        // 정적 팩터리 메서드의 메서드 참조를 공급자(Supplier)로 사용
        Supplier<Example> exampleSupplier = Example::createInstance;

        // 공급자를 이용하여 객체 생성
        Example instance = exampleSupplier.get();
}
```
이러한 장점들이 필요하지 않다면 public 필드 방식이 좋음.

## 1과 2의 직렬화 (12장 참고)

<hr>

둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것으로는 부족함.

모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메서드를 제공해야 함. (item 89)
＞ 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어짐.

해결
> readResolve 메서드 추가
역직렬화한 객체는 무시하고 클래스 초기화할 때 만들어진 Example 인스턴스를 반환함.

> 따라서 Example 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 필요가 없으니 모든 인스턴스 필드를 transient로 선언해야 함.
사실 readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 trandient로 선언해야 함. 
<br>
> MutablePeriod 공격(item 88)과 비슷한 방식으로 readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남기 때문. 

```JAVA
//싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve() {
	// 진짜 Example 을 반환하고, 가짜 Example 는 가비지 컬렉터에 맡김.
	return INSTANCE;
}
```

## 3. 열거 타입

<hr>

```JAVA
public enum Example {
	INSTANCE;
}
```

public 필드 방식과 비슷하지만 더 간결함.

추가적인 노력 없이 직렬화할 수 있음.
> 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽히 막아줌.

대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법임.
> 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없음.
>
> 열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있음.