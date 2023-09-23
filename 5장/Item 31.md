# 아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

## 제네릭은 불공변
매개변수화 타입은 불공변이다. 예를 들어 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입 또는 상위 타입이라는 관계가 성립될 수 없다.

List<Object>에는 어떠한 객체도 넣을 수 있지만 List<String>에는 문자열만 넣을 수 있다. 즉 List<String>이 List<Object>의 기능을 제대로 수행하지 못하므로 하위 타입이라고 말할 수 없다.

이것이 와일드 카드 타입을 사용해야 하는 이유이다.

~~~Java
static class StackWithGeneric<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public StackWithGeneric() {
        elements = (E[])  new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public void pushAll(Iterable<E> src) {
        for (E e : src) {
            push(e);
        }
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        E result = elements[size];
        elements[size] = null;

        return result;
    }

    private void ensureCapacity() {
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    @Override
    public String toString() {
        return "StackWithGeneric{" +
                "elements=" + Arrays.toString(elements) +
                ", size=" + size +
                '}';
    }
}
~~~
pushAll이라는 메서드를 추가했다.
~~~Java
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
~~~
~~~Java
@Test
public void stackTest() {
    StackWithGeneric<Number> stack = new StackWithGeneric<>();
    List<Integer> integers = List.of(1, 2, 3);
    stack.pushAll(integers);
    System.out.println("stack = " + stack);
}
~~~
다음과 같은 오류가 난다
~~~Java
java: incompatible types: java.util.List<java.lang.Integer> cannot be converted to java.lang.Iterable<java.lang.Number>
~~~
Number 타입의 스택에 Integer는 하위타입이기에 들어갈 수 있다고 생각하지만, 제네릭이 불공변이어서 에러가 난다.

와일드카드 타입을 사용하도록 pushAll 메서드를 수정해보자.

### 생산자(producer) 매개변수에 와일드카드 타입 적용
~~~Java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
~~~
- pushAll()을 위와 같이 바꾸면, 더이상 에러가 나지 않는다.
- 단순히 E만 이용했을 때는 Number만 받을 수 있지만, 이제 Integer도 받을 수 있다.
- ? 와일드카드 타입으로 E를 상속한 아무 타입이나 받아줄 수 있다.
- 제네릭의 불공변 때문에 이렇게 한정적 와일드카드 타입(? extends E)을 이용해주는 것이 좋다.

### 소비자(consumer) 매개변수에 와일드카드 타입 적용
~~~Java
public void popAll(Collection<E> dst) {
    while(size > 0) {
        dst.add(pop());
    }
}

@Test
public void stackTest() {
    StackWithGeneric<Number> stack = new StackWithGeneric<>();
    List<Integer> integers = List.of(1, 2, 3);
    // produce
    stack.pushAll(integers);
    System.out.println("stack = " + stack);

    Collection<Object> objects = new ArrayList<>();
    stack.popAll(objects);
}
~~~
popAll()도 위의 pushAll() 케이스와 비슷하게, 제네릭의 불공변 특성 때문에 에러가 난다.
불공변덕에 StackWithGeneric의 제네릭이 Number였어서 유연성 없이 Collection<Number> 타입만을 인수로 받을 수 있는데, 상위 타입인 Object가 와서 문제인 것이다.
소비자의 경우, 생산자와 다르게 상위 클래스로만 유연하게 확장해야 한다.
상위 객체만이 하위 객체를 받아줄 수 있다. 그러므로 와일드카드에 super 키워드를 사용해야 한다.
~~~Java
public void popAll(Collection<? super E> dst) {
    while(size > 0) {
        dst.add(pop());
    }
}
~~~
이렇게 작성하면 에러 없이 잘 작동한다.

### 생산자와 소비자 패턴에서 제네릭의 형태
- 상위 타입일수록 더 적은 정보를 가지고 있다. 하위 타입일수록 더 구체적인 다른 정보를 많이 가지고 있다.
- 생산자일 때는 하위 타입을 받아도 객체는 필요한 정보만 쓰면 되니, extends를 쓴다.
- 소비자일 때는 소비자의 정보를 받아줄 타입이 필요 하니, super를 쓴다.
- 만일 매개변수가 생산자와 소비자 둘의 역할을 모두 하면, 어느것을 써도 좋을 것이 없기 때문에 그냥 정확한 타입을 써야 한다.
  - 생산을 해야 하는데 상위 객체가 온다면 정보가 부족하고, 소비를 해야 하는데 하위 객체가 온다면 정보를 받아줄 수 없다.

위와 같이 생산자(producer)에는 extends, 소비자(consumer)에는 super를 쓴다고 해서 PECS라고 부른다.

### Comparable의 제네릭 타입
Comparable은 항상 소비자의 입장이다. PECS 원칙에 따라 super를 쓰는 것이 좋다.

아이템 30-7의 코드를 살펴보자.
~~~Java
public static <E extends Comparable<E>> E max(List<E> list){
    if(list.isEmpty()) throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
    E result = null;
    for (E e : list){
        if(result == null || e.compareTo(result)>0) result = Objects.requireNonNull(e);
    }
    return result;
}
~~~
겉보기에는 문제가 없지만 , 다음과 같은 상황에는 문제가 발생한다.
~~~Java
static class Student extends Person {
    private final int grade;

    public Student(String name, int age, int grade) {
        super(name, age);
        this.grade = grade;
    }
}
~~~
- Student 클래스는 Person 클래스와 공통 요소를 가지고 있어 Comparable<Student>를 구현한 것이 아니라, Comparable<Person>을 구현했다.
- 이럴 때는 Person 객체의 비교 기준으로 max 메서드를 사용하려면 아래와 같이 선언해주어야 한다.
~~~Java
public static <E extends Comparable<? super E>> E max(List<E> list);
~~~

### 메서드 선언에 타입 매개변수가 한번만 나온다면 와일드 카드를 쓰자
메서드 선언에 타입 매개변수가 단 한번만 나온다면, 와일드 카드를 썼을 때 그 의도가 더 명확하다.
~~~Java
public void methodSignatureWildcardTest() {
    List<String> strings = new ArrayList<>(List.of("a", "b", "c", "d", "e", "f"));
    swap(strings, 0, 1);
    System.out.println("strings = " + strings);
}

public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
~~~
- 그냥 ? 와일드카드 타입만 쓰면, list.add()가 불가능하기에, showHelper()라는 도우미 메서드를 만들었다.
- 위와 같이 코드를 구성하면, API 외부로는 ? 와일드카드가 보이고, 내부에서는 <E> 타입을 쓰게 된다.

## 정리
- 조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다.
- 널리 쓰일 라이브러리엔 반드시 와일드카드 타입을 적용하자.
- 생산자에는 extends 소비자에는 super를 사용하는 PECS를 기억하자.