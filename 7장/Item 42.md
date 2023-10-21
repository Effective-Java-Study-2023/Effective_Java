# Item 42 - 익명 클래스보다는 람다를 사용하라

<hr>

## 람다식이란?

<hr>

람다식(Lambda Expression)이란 함수를 하나의 식(expression)으로 표현한 것이다. 
함수를 람다식으로 표현하면 메소드의 이름이 필요 없기 때문에, 람다식은 익명 함수(Anonymous Function)의 한 종류라고 볼 수 있다.


기존의 방식에서는 함수를 선언할 때 다음과 같이 선언하였다.
```Java
// 기존의 방식
접근제어자 반환티입 메소드명(매개변수) {
	실행문
}

// 예시
public String hello() {
    return "Hello World!";
}
```

하지만 람다 방식으로는 위와 같이 메소드 명이 불필요하며, 다음과 같이 괄호() 와 화살표-> 를 이용해 함수를 선언하게 된다.
```Java
// 람다 방식
(매개변수, ... ) -> { 실행문 ... }

// 예시
() -> "Hello World!";
```

람다식의 특징

    람다식 내에서 사용되는 지역변수는 final이 붙지 않아도 상수로 간주된다.
    람다식으로 선언된 변수명은 다른 변수명과 중복될 수 없다.

람다식의 장점

- 코드를 간결하게 만들 수 있다.
- 식에 개발자의 의도가 명확히 드러나 가독성이 높아진다.
- 함수를 만드는 과정없이 한번에 처리할 수 있어 생산성이 높아진다.
- 병렬프로그래밍이 용이하다.

람다식의 단점
- 람다를 사용하면서 만든 무명함수는 재사용이 불가능하다.
- 디버깅이 어렵다.
- 람다를 남발하면 비슷한 함수가 중복 생성되어 코드가 지저분해질 수 있다.
- 재귀로 만들경우에 부적합하다.

## 익명 클래스와 람다

<hr>

예전 Java에서 함수 타입을 표현하기 위해 추상 메서드를 하나만 담는 인터페이스를 사용했다. 이러한 인터페이스의 인스턴스는 특정 함수 혹은 동작을 나타내는데 사용했다.

JDK 1.1 등장 이후 이러한 함수 객체를 만드는 주요 수단이 익명 클래스가 되었다.

### 익명 클래스

익명 클래스(anonymous class)는 말그대로 이름이 없는 클래스이다. 
이러한 익명 클래스를 사용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다. 
즉석에서 필요한 구현을 만들어 사용할 수 있다.

```Java
public class Car {
    private final String name;
    private final String brand;
    
    public Car() {
        this.name = "name";
        this.brand = "brand";
    }

    public Car(String name, String brand) {
        this.name = name;
        this.brand = brand;
    }

    public void sale() {
        System.out.println(name + "팔림");
    }
}
```
기본적으로 인스턴스 생성을 위해서는 `new` 키워드를 사용하여 진행한다. 여기서 인스턴스는 `new Car("car", "kia")`는 클래스 이름 `Car`를 가진다. 
즉 이름을 가진 클래스이다.
```Java
Car car = new Car("car", "kia");
```


이름이 없는 클래스는 아래와 같다.

```Java
Car car = new Car() {
    @Override
    public void sale(){
        System.out.println("어떤 차가 팔림");
    }
};
```
인스턴스 생성을 위해 `new Car()`로 인하여 인스턴스의 이름을 가지고 있다고 판단할 수 있지만 이후 등장하는 `{}`로 인하여 이름이 없다고 판단한다.

해당 인스턴스는 Car 클래스를 상속받는 형태를 띄고 있다. 그렇기 때문에 Car 클래스의 메서드를 자유롭게 재정의할 수 있다. 
위에서 나타나는 Car는 클래스 이름이 아닌 단순히 상속 받을 클래스의 이름을 나타낼 뿐이다.

정리하면 익명 클래스를 활용한 인스턴스화는 Car 클래스를 상속한 이름이 없는 클래스의 인스턴스일 뿐이다.
익명 클래스는 이름을 가지지 않기 때문에 내부에 생성자 선언이 불가능하다.

### 람다

위에서 언급한 익명 클래스는 다양한 동작을 구현할 수 있지만 부가적인 코드를 많이 가지고 있다. 
더 깔끔한 코드로 동작을 구현하고 전달하기 위해 Java 8은 람다 표현식을 제공한다.

처음 설명했듯이 람다 표현식은 메서드로 전달할 수 있는 함수 객체를 단순화한 것이다. 
람다 표현식은 이름을 가지지 않고 매개변수 리스트, 바디, 반환 형식 등을 가질 수 있다.

아래는 람다 표현식의 특징을 확인할 수 있는 간단한 예제이다.

```Java
List<Integer> numbers = new ArrayList<>(List.of(3, 2, 1, 4, 10, 5, 7));
numbers.sort((o1, o2) -> Integer.compare(o1, o2));
```

이 코드를 람다 표현식이 아닌 익명 클래스로 구현하면 단순히 오름차순 정렬을 위한 기능이지만 부가적인 많은 코드가 필요하다.

```Java
List<Integer> numbers = new ArrayList<>(List.of(3, 2, 1, 4, 10, 5, 7));
numbers.sort(new Comparator<>() {
    
    @Override
    public int compare(Integer o1, Integer o2) {
        return Integer.compare(o1, o2);
    }
});
```

동일하게 오름차순 정렬을 진행하지만 람다식이 더 간결하게 표현이 가능하다.

여기서 람다의 매개변수(o1, o2)의 타입은 Integer, 반환값의 타입은 int이지만 코드에는 언급되지 않는다. 컴파일러가 문맥을 살펴보며 타입을 추론한다. 
타입 추론에 대한 규칙은 많은 내용과 복잡한 과정을 가지고 있기 때문에 자세한 언급은 생략한다.

정리하면 타입을 명시해야 코드가 더 명확할 때를 제외하고는 람다의 모든 매개 변수 타입을 생략하는 것이 보다 더 간결한 표현을 만들어 준다.

### 함수 객체(람다)를 인스턴스 필드에 저장해 상수별 동작을 구현한 열거 타입

```Java
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator operator;

    Operation(String symbol, DoubleBinaryOperator operator) {
        this.symbol = symbol;
        this.operator = operator;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y) {
        return operator.applyAsDouble(x, y);
    }
}
```

람다를 이용하면 열거 타입의 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 동작하는 코드를 쉽게 구현할 수 있다.

단순히 각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘기고, 생성자는 이 람다를 인스턴스 필드로 저장해둔다.

그런 다음 apply 메서드에서 필드에 저장된 람다를 호출하기만 하면 된다.

### 람다의 단점

람다는 이름 없고 문서화를 못 한다. 코드 자체로 동작이 명확하게 설명이 되지 않거나 코드 줄 수가 많아지면 람다 사용을 고민해봐야 한다. 
람다는 한 줄 일 때 가장 좋고 길어야 세 줄 안에 끝내는 것이 좋다.

#### this

람다는 자신을 참조할 수 없다. 람다에서 `this`키워드는 바깥 인스턴스를 가리킨다.

```Java
public class LambdaReference {

    public static void main(String[] args) {
        LambdaReference lambdaReference = new LambdaReference();
        lambdaReference.run(); // LambdaReference
    }

    public void run() {
        Runnable runnable = () -> System.out.println(this);
        runnable.run();
    }

    @Override
    public String toString() {
        return "LambdaReference";
    }
}
```

실행하면 toString 메서드의 LambdaReference를 출력하고 있다.

익명 클래스의 this 키워드는 익명 클래스의 인스턴스 자신을 가리킨다.

```Java
public class AnonymousReference {

    public static void main(String[] args) {
        AnonymousReference anonymousReference = new AnonymousReference();
        anonymousReference.run(); // Runnable
    }

    public void run() {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(this);
            }

            @Override
            public String toString() {
                return "Runnable";
            }
        };
        runnable.run();
    }

    @Override
    public String toString() {
        return "AnonymousReference";
    }
}
```

## 해석에서의 차이점

<hr>

this를 제외하고서는 익명 클래스와 람다는 단순히 표현 방식에만 차이가 있다고 볼 수 있을까?

내부적으로도 차이점이 존재한다.

```Java
 public static void main(String[] args) {
        Comparator<Integer> comparator = new Comparator<>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return Integer.compare(o1, o2);
            }
        };
    }
```
이 코드를 빌드하고 생성된 클래스 파일의 바이트 코드를 살펴보면
따로 생성된 익명 클래스를 INVOKESPECIAL 이라는 OPCODE로 생성자를 호출하는 것을 볼 수 있다.

    INVOKESPECIAL
    Invoke instance method; special handling for superclass, private, and instance initialization method invocations
    
    인스턴스 메서드 호출, 슈퍼 클래스, 개인 및 인스턴스 초기화 메서드 호출에 대한 특수 처리를 의미한다.
    
    정리하면 생성자, private method, super 클래스 호출 등에 사용된다.

반면 람다로 작성된 Comparator에서는 

```Java
public static void main(String[] args) {
        Comparator<Integer> comparator = (o1, o2) -> Integer.compare(o1, o2);
}
```

익명 클래스와는 다른 `INVOKEDYNAMIC`이라는 OPCODE를 활용한다.

    INVOKEDYNAMIC
    Invoke dynamic method.
    
    동적 메서드를 호출한다. Java8 부터 default method, lambda compile시에 사용된다.

람다를 단순히 익명 클래스로 치환하여 해석할 경우 익명 클래스의 특징을 그대로 갖고있다고 생각할 수 있다. 
즉 람다식 마다 클래스가 하나씩 생기고 항상 새로운 인스턴스를 할당하는 것이라고 볼 수 있다는 것이다.

Java 8에서는 함수형 인터페이스라는 개념을 통해 람다 표현식을 작성할 수 있게 되었다.

    함수형 인터페이스는 추상 메서드가 오직 하나인 인터페이스이다.

함수형 인터페이스를 통해 작성된 람다 표현식은 익명 클래스와는 다른 방식으로 해석된다. 
또한 기존에 존재하던 개념을 그대로 사용하기 때문에 아래와 같은 장점들을 가질 수 있다.

> - 기존에도 추상 메서드가 하나인 인터페이스를 많이 사용했기 때문에 호환을 유지할 수 있다.
> 
> - 기존에 존재하던 추상 메서드가 하나인 인터페이스들도 함수형 인터페이스로 동작할 수 있게 된다.
> 
> - 새로운 타입을 추가하지 않아도 람다식을 함수형 인터페이스의 인스턴스로 변환하여 큰 변화 없이 적용이 가능하다.
> 
> - 컴파일러에서 구조적으로 람다식을 함수형 인터페이스로 인식하고 치환할 수 있다.

## 정리

<hr>

익명 클래스가 필요한 경우 함수형 인터페이스가 아닌 타입의 인스턴스를 만들 때만 사용한다. 또한 람다에서 this는 바깥 클래스를 가리키기 때문에 유의해서 사용해야 한다. 이러한 람다는 함수 객체를 아주 쉽게 표현할 수 있으며 익명 클래스와 다르게 새로운 인스턴스를 할당하지 않는다.

람다를 단순히 익명 클래스로 치환하여 해석할 경우 람다식 마다 클래스가 하나씩 생기고 매번 새로운 인스턴스를 할당하는 문제를 동반한다. 이것을 방지하기 위해 함수형 인터페이스를 통해 작성된 람다 표현식은 익명 클래스와는 다른 방식으로 해석된다.

결론은 아이템 제목과 동일하게 익명 클래스보다는 람다를 사용해야 한다.