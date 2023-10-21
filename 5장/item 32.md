# 아이템 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라. 

> 실체화 불가타입(매개변수화된 타입 등)으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보낸다.
- 실체화 불가 타입은 런타임에 컴파일타임보다 타입 정보가 적다. 
- 메서드 선언 시 실체화 불가 타입으로 가변인수 매개변수 선언하거나
- 메서드 호출 시 가변인수 매개변수가 실체화 불가 타입으로 추론되는 경우 컴파일 경고 발생

### 제네릭과 varargs 를 혼용하면 타입 안전성이 깨진다.
~~~java
public class Danger {
  public static void main(String[] args) {
      List<String> stringList = new ArrayList<>();
      stringList.add("Hello");
  
      dangerous(stringList);
  }
  
  public static void dangerous(List<String>... lists) {
      List<Boolean> booleanList = Arrays.asList(true, false, true);
      Object[] objects = lists;
      objects[0] = booleanList;             // 힙 오염 발생 부분 : list[0] 가 List<Boolean> 를 참조
      String s = lists[0].get(0);           // 여기서 ClassCastException 발생 : 보이지 않는 형변환
  }
}
~~~

### 제네릭 배열을 직접 생성하는건 허용하지 않으면서, 제네릭 varargs 매개변수를 받는 메서드 선언 가능한 이유는?
> 제네릭이나 매개변수화 타입의 가변인수 매개변수를 받는 메서드는 실무에서 유용하기 때문이다. 

자바 라이브러리에서도 사용하고 있다. 
~~~ java
Arrays.asList(T... a)
Collections.addAll(Collection<? super T> c
~~~

### 타입 안전하게 작성했어도 나오는 경고는 어떻게 하면 좋을까?
> @SafeVarargs
- 자바 7에서 추가되었고, 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨겨준다.
- 이 어노테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장한다는 의미
  - 그러니까, 타입 안전한 게 확실하지 않으면 절대 사용하면 안 된다!!!!!

### 안전한지는 어떻게 확신하지?
- 가변인수 메서드를 호출하면 제네릭 배열이 만들어진다는 사실을 까먹지 말자! 
- 메서드가 가변인수를 담는 배열에 아무것도 저장하지 않아야 한다.
- 가변인수를 담은 배열의 참조가 밖으로 노출되지 않아야 한다. 
- 즉, varargs 매개변수 배열은 순수하게 인수를 전달하는 일만해야 한다.

### 근데... 매개변수 배열에 아무것도 저장하지 않고도 타입 안전성을 깰 수 있다. 
- 제네릭 가변인수 매개변수 배열에 다른 메서드가 접근할 수 있다면, 안전하지 않다. 
~~~Java
public class PickTwo {

  public static void main(String[] args) {
    String[] result = pickTwo("A", "B", "C");
    System.out.println(result);
  }

  static <T> T[] pickTwo(T a, T b, T c) {
    switch (ThreadLocalRandom.current().nextInt(3)) {
      case 0 : return toArray(a, b);
      case 1 : return toArray(b, c);
      case 2 : return toArray(a, c);
    }
    throw new AssertionError();
  }

  static <T> T[] toArray(T... args) {
    return args;
  }
}
~~~
- 힙 오염의 원인 'toArray' 메서드
  - 제네릭 varargs 를 이용하여 T[] 를 반환하도록 선언
  - 런타임에서는 Object[] 타입의 배열을 반환한다. 
- 런타임에서 ClassCastException 발생
- 반환되는 배열의 실제타입은 Object[] 
- main 에서 실제 할당은 String[] 
- String 은 Object 의 하위 타입이 아니기 때문에 형변환 불가

### 그러니까, @SafeVarargs 는... 
- 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs 를 사용하자. 
  - 재정의할 수 없는 메서드에 사용 
  - 정적 메서드, final 인스턴스 메서드, private 인스턴스 메서드 허용
- 힙 오염 경고가 뜬다면 안전한지 확인하고 사용하라.
  - varargs 매개변수 배열에 아무것도 저장하지 않았는지 확인
  - 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않았는지 확인

### @SafeVarargs 사용하지 않고도 가변인수 매개변수 효과를 누리는 법
> 가변인수 매개변수 대신 List 매개변수로 대체할 수 있다.  
~~~java
public class PickTwo {

  public static void main(String[] args) {
      List<String> result = pickTwo("A", "B", "C"); // 배열 없이 제네릭만 사용하여 타입 안전
      System.out.println(result);
  }
  
  static <T> List<T> pickTwo(T a, T b, T c) {
      switch (ThreadLocalRandom.current().nextInt(3)) {
          case 0: return List.of(a, b);
          case 1: return List.of(b, c);
          case 2: return List.of(a, c);
      }
      throw new AssertionError();
  }
}

~~~
- 정적 팩터리 메서드 List.of 를 활용하여 임의의 개수의 인수를 넘긴 에시 
- 장점 
  - 컴파일러가 이 메서드의 타입 안전성을 검증한다. 
- 단점 
  - 클라이언트 코드가 살짝 지저분해진다. 
  - 속도가 조금 느려질 수 있다.


## 결론 

가변인수와 제네릭은 같이 사용하지 않는 것이 좋다. 
메서드에 제네릭 (혹은 매개변수화된) 가변인수 매개변수를 사용한다면, 
가장 먼저 그 메서드가 타입 안전한지 확인하고, @SafeVarargs 를 달아 사용하자. 
---
* 가변인수(varargs) 
  * variable number of arguments 
  * 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해준다. 
  * 가변인수 메서드를 호출하면 가변인수를 담기 위한 "**배열**"이 자동으로 생성된다. 