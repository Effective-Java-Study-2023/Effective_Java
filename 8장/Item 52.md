# Item 52 - 다중정의는 신중히 사용하라

<hr>

## 다중정의?

<hr>

오버로딩, 리턴 타입과 메서드는 같으나, 파라미터의 개수나 타입이 다른 메서드를 작성하는 것.

```Java
public class Main {
    public static String example(Set<?> s) {
        return "set";
    }

    public static String example(List<?> l) {
        return "list";
    }

    public static String example(Collection<?> c) {
        return "collection";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };
        
        for (Collection<?> c : collections) {
            System.out.println(example(c));
        }
    }
}
```
```Java
/*      
실행결과
collection
collection
collection
 */
```
collections 배열의 타입이 Collection 타입이기 때문에 마지막 example 메서드가 실행된다.

## 재정의는 다중정의라고 할 수 있는가?

<hr>

```Java
class Wine {
    String name() {
        return "Wine";
    }
}

class SparklingWine extends Wine {
    @Override
    String name() {
        return "SparklingWine";
    }
}
class Champagne extends SparklingWine {
    @Override
    String name() {
        return "Champagne";
    }
}
public class Main {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(),
                new SparklingWine(),
                new Champagne()
        );

        for (Wine w : wineList) {
            System.out.println(w.name());
        }
    }
}
```

```Java
/*
실행결과
Wine
SparklingWine
Champagne
 */
```

같은 타입의 Wine 이지만, 최하위 클래스가 정의한 name()메서드가 호출된다.

    오버라이딩의 경우 런타입 시점에서 정해짐 = 동적
    오버로딩의 경우 컴파일 시점에서 정해짐 = 정적


## 다중정의의 단점

<hr>

사용자가 파라미터를 넘기면서 어떤 오버로딩 메서드가 호출될지 모른다면 오작동을 야기할 수 있음.


## 해결

<hr>

웬만하면 오버로딩을 하지 말자.

가변인수를 쓰는 메서드는 아예 시도조차 하지 말자.

가장 간단한 방법 = 메서드 네이밍을 다르게 지어주기

![image](https://github.com/been1118/CS-Study/assets/123082067/642214b1-1618-4141-96de-eb61c2a65894)

<br>

 `ObjectOutputStream`

장점
- write 메서드와 네이밍을 맞추기 좋다.

## 생성자 오버로딩은?

<hr>

여러 생성자가 필요한 상황이 분명 존재할 것이다.

파라미터 개수만 같은 생성자들끼리는 어떻게 처리할 것인가?

    파라미터 중 한 개 이상이 근본적으로 다른 타입이면 가능
    근본적으로 다른 타입이란, 서로 캐스팅 할 수 없다는 것을 의미함.

![image](https://github.com/been1118/CS-Study/assets/123082067/63f53ab6-2617-4437-97fa-897d158e83dd)

하지만 자바 5 이상부터 오토박싱이 도입되었다.

```Java
public class Main {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }

        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }

        System.out.println(set + " " + list);
    }
}
```

```Java
/*
예상결과
[-3, -2, -1] [-3, -2, -1]
        
실행결과
[-3, -2, -1] [-2, 0, 2]
 */
```

왜 그럴까?

[-3, -2, -1, 0, 1, 2] > 인덱스 0 원소 삭제 >

[-2, -1, 0, 1, 2] > 인덱스 1 원소 삭제 >

[-2, 0, 1, 2] > 인덱스 2 원소 삭제 >

[-2, 0, 2]

<br>

remove 메서드?

원래 원했던 것은 remove(Object) 였으나 for문 안의 타입이 int이기 때문에
Index를 기준으로 없애는 remove(int)가 컴파일 시점에 적용되었기 때문이다.

자바 4 이전에는 제네릭도 없었고 오토박싱도 없었으나, 
자바 5 이후로는 제네릭과 오토박싱이 도입되어 더 이상 Object와 int가 근본적으로 다른 타입이 아니기 때문이다.

## 메서드 참조

<hr>

![image](https://github.com/been1118/CS-Study/assets/123082067/69098415-2d33-4b15-b8c2-d0d51cfde296)

둘 다 runnable을 받는 생성자, 메서드이다.
그런데 밑 코드만 컴파일 오류가 발생한다.

![image](https://github.com/been1118/CS-Study/assets/123082067/1ab910be-2a73-4fe3-8a09-80ad1fb887a1)

Submit의 경우 Callable, Runnable로 오버로딩되어있는 것을 볼 수 있다.

System.out.println의 경우 void이기 때문에 Runnable 아닌가? 할 수 있겠지만,

System.out.println의 경우도 오버로딩되어있다.

Submit의 경우도 오버로딩 되었기 때문에 기대하는 동작이 이루어지지 않는 것이다.

    오버로딩 추론 규칙이 복잡해짐.

또, 비록 서로 다른 함수형 인터페이스라도 파라미터 위치가 같으면 혼란이 발생한다.
서로 다른 함수형 인터페이스라도 근본적으로 다르지 않다라는 뜻이다.

여기서 또한 근본적으로 다르지 않다는 것은 서로 캐스팅이 된다는 것이다.

그렇다면 System.out이 Runnable과 Callable 둘 다 Implements한다는 의미로 받아들여지는데,
내부 코드에서는 또 그렇지 않다.

결론적으로는 서로 다른 함수형 인터페이스더라도 같은 위치의 인수로 받아서는 안된다.

## In Java

<hr>

![image](https://github.com/been1118/CS-Study/assets/123082067/6aa698e5-9515-4003-981b-0d2abbcbe665)

![image](https://github.com/been1118/CS-Study/assets/123082067/fc992ff5-4f42-4b70-a8eb-559eac260853)

이번 아이템의 주제와 반대되지만, 
단순히 캐스팅을 한 후에 동일한 동작을 하고 있기 때문에 큰 문제는 없다고 할 수 있다.

## 결론

<hr>

무조건 다중정의를 이용하기 보다는 이번 내용에서 제시한 여러 조건들을 고려했을 때 

다중정의를 쓰지 않을 수 있다면 쓰지 않는 것이 좋다.