# Item 29 - 이왕이면 제네릭 타입으로 만들라

JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운 편이지만, 제네릭 타입을 새로 만드는 일은 조금 더 어렵다.

```Java 
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
    
    public boolean isEmpty() {
        return size == 0;
    }
    
    public void ensureCapacity() {
        if (elements.length == size) 
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}

```

클라이언트는 스택에서 꺼낸 객체를 형변환해야 하는데, 이때 런타임 오류가 발생할 수 있다.

![image](https://github.com/been1118/Effective_Java/assets/123082067/4266e97b-169e-4748-ae55-5f1ab09b9988)


![image](https://github.com/been1118/Effective_Java/assets/123082067/22b35b4a-3fde-44df-954c-4da5023a842a)



## 제네릭으로

<hr>

우선 Object를 Generic으로

![image](https://github.com/been1118/Effective_Java/assets/123082067/597d5c4b-b70f-4f97-bb60-faae54f2ea60)


Generic은 실체화할 수 없다.
E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다는 것이다.

## 해결방법 - 1

<hr>

배열을 Object로 생성하고 Generic으로 형 변환

![image](https://github.com/been1118/Effective_Java/assets/123082067/86f240de-1bc4-4ba1-bad6-cb964ad007b1)


배열을 E[]로 선언하여 E타입만 받음을 명시해준다. (타입 안전)

> 컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없어 경고를 보낸다. 따라서 이 비검사 형변환이 프로그램의 타입 안전성을 해치지 않음을
> 스스로 확인해야 한다. 위 Stack<E> 코드에서는 배열을 private 필드에 저장하고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없고,
> push 메서드로는 E타입만을 인자로 받는다.

- 하지만 런타임에는 E[]가 아닌 Object[]로 동작한다.
> - 제네릭은 컴파일 이후 Object로 바뀐다. (타입 소거)

- 따라서 힙 오염의 가능성이 존재한다.

## 힙 오염이란? (Item 32)

<hr>

매개변수 유형이 다른 서로 다른 타입을 참조할 때 발생하는 문제이다.

컴파일은 되지만, 런타임에 ClassCastException을 발생시킨다.
> 예를 들면 List<Integer> a를 선언해 놨는데, List<String>으로 선언된 다른 변수 b가
> List<Integer>를 참조할 경우에
> a는 int list니까 int를 다뤄야 함.
> b는 String list이기 때문에 String을 다뤄야 함.
> 둘의 참조가 같음으로 꺼내는 타입이 달라질 수 있음.(a에 int를 넣고 b에서 꺼내면 String으로 꺼내야 함)

![image](https://github.com/been1118/Effective_Java/assets/123082067/4b51111b-20be-410f-abb6-7e33fcf188e5)


List<String>에서 String으로 꺼내기 때문에 컴파일은 되지만,
실제 데이터는 int이므로 ClassCastException이 발생한다.

## 해결방법 - 2

<hr>

elements의 타입을 Object[]로 바꾸고 pop에서 형변환

![image](https://github.com/been1118/Effective_Java/assets/123082067/81954a00-870d-4cf8-a617-7b8f0427660c)


방법 1과 마찬가지로 E는 실체화 불가 타입으므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다.

따라서 직접 증명하고 경고를 숨길 수 있다.
push에서 E타입만 허용하므로 이 형변환은 안전하다.

- 애초에 Object[] 이므로 힙 오염의 가능성이 없다.
- push로 E만 들어오므로 Object[]에 저장되는 데이터가 모두 E타입임이 보장된다.
- 메서드 호출시마다 타입 캐스팅을 해주어야 한다.

## 비교

<hr>

방법 1이 가독성이 더 좋고, 오직 E타입의 인스턴스만 받음을 확실히 어필한다.
방법 1에서는 형 변환을 배열 생성 시 단 한 번만 해주면 되지만, 방법 2에서는 배열에서 원소를 읽을 때마다 해줘야 한다. 

따라서 현업에서는 방법 1을 더 선호한다고 한다.
하지만 배열의 런타임 타입과 컴파일 타입이 달라서 힙 오염이 발생할 가능성이 있기 때문에
방법 2를 고수하는 프로그래머가 있기도 하다.

## 제네릭 타입 사용 시 주의점

<hr>

- 타입 매개변수 제약이 없다. (기본 타입 제외)
  - 기본 타입 사용 시에는 박싱된 타입으로 우회해서 사용하자.

- 타입 매개변수에 제약을 둘 수도 있다. (제한적 타입 매개변수, bounded type parameter)
```Java
<E extends Delayed>
// Delayed의 하위 타입만을 담을 수 있음.
```

## 정리

<hr>

- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
- 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라.
- 그런 경우 제네릭 타입으로 만들어야 하는 경우가 많다.
