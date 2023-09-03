# Comparable을 구현할지 고려하라

### Comparable 인터페이스에 유일무이한 메서드인 compareTo를 알아보자.

<hr>


Primitive 자료형은 쉽게 비교 및 정렬이 가능하다.
```Java
import java.util.Arrays;

public class Primitive {
    public static void main(String[] args) {
        int a = 2;
        int b = 3;

        // a > b, a < b...

        int[] intArr = {9, 6, 7, 4, 3, 8, 2, 5, 1};

        Arrays.sort(intArr);

        //[1, 2, 3, 4, 5, 6, 7, 8, 9]
    }
}
```


String의 정렬

```Java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ExampleString {
    public static void main(String[] args) {
        List<String> words = new ArrayList<>();

        words.add("book");
        words.add("apple");
        // ["book", "apple"]
        
        Collections.sort(words);
        // ["apple", "book"]
    }
}
```

## 객체를 비교할 수 있을까?

<hr>

```Java
import java.util.ArrayList;
import java.util.Collections;

public class Book {
    private final String title;
    private final int price;

    public Book(String title, int price) {
        this.title = title;
        this.price = price;
    }
}

public class Main {
    public static void main(String[] args) {
        List<Book> bookList = new ArrayList<>();
        bookList.add(new Book("첫 번째 책", 20000));
        bookList.add(new Book("두 번째 책", 25000));

        Collections.sort(bookList); 
        /**      
        * error :
        *        Required type : List<T>
        *        Provided : List<Car>
        *        reason: no instance(s) of type variable(s) T exist so that Car conforms to Comparable<? super T>
        */
    }
}
```
타입 변수 T에 대한 Car와 관련된 인스턴스가 없다는 내용의 컴파일 에러가 발생한다.
T와 관련된 타입과 호환되지 않는다는 것을 나타낸다.

    String은 Comparable을 구현했지만, Book은 Comparable을 구현하지 않았기 때문이다.

<br>

## Comparable이란?

<hr>

 - interface
 - 제네릭 <T> 자리에 비교할 타입이 들어간다.
 - 구현체는 반드시 compareTo메서드를 오버라이딩해서 사용한다.
 - 자바 라이브러리의 모든 값 클래스와 열거 객체는 Comparable을 구현해 놓았다.

```Java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

<br>

## compareTo메서드 구현

<hr>

 - compareTo를 호출하는 인스턴스와 매개변수로 주어지는 인스턴스의 순서를 비교한다.
 - 이 객체를 기준으로, 주어진 객체보다 작으면 음의 정수를, 주어진 객체와 같으면 0을, 주어진 객체보다 크면 양의 정수를 반환한다.
 - 일반적으로 -1, 0, 1을 반환하나 반드시 -1, 1일 필요는 없다.
 - 비교할 수 없는 타입의 인스턴스가 주어지면 ClassCastException을 던진다.

<br>

    첫번째 규약 (대칭성)
     - x.compareTo(y)와 y.compareTo(x)의 부호는 반대여야 한다.
     - x.compareTo(y)가 0이면 y.compareTo(x)도 0이어야 한다.
     - x.compareTo(y)가 예외를 던지면 y.compareTo(x)도 예외를 던져야 한다.
     
     -> 비교의 순서가 바뀌어도 반드시 예상한 결과가 나와야 한다.
<br>

    두 번째 규약 (추이성)
     - x.compareTo(y) > 0, y.compareTo(z) > 0이면, x.compareTo(z) > 0이다.

     -> 첫 번째가 두번째보다 크고 두 번째가 세번째보다 크면 첫번째는 세번째보다 커야한다.

<br>

    세 번째 규약 (반사성)
     - x.compareTo(y) == 0 이면 x.compareTo(z)와 y.compareTo(z)의 값이 같다.
        (x와 y가 같다면 모든 z에 대하여 x와 y의 비교 결과는 같아야 함)

     -> 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같다.

<br>

    네 번째 규약 (권고)
     - (x.compareTo(y) == 0) == (x.equals(y)여야 한다.
         - 즉, compareTo로 수행한 동치성 테스트의 결과가 equals의 결과와 같아야 한다.
         - 이 규약을 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 일관된다.
     - 지키지 않은 클래스는 그 사실을 명시해야 한다.
     - 이 규약을 지키지 않은 예시 : BigDecimal
         - new BigDecimal("1.0")과 new BigDecimal("1.0")
         - compareTo메서드로 비교하면 두 인스턴스가 같으나 equals메서드로 비교하면 서로 다르다는 결과가 나온다.
            따라서 TreeSet에서는 원소를 1개만 갖지만, HashSet에서는 원소를 2개 갖는다.

<br>

## 구현 시 주의사항

<hr>

원시 타입을 비교할 때 "<"나 ">" 대신 wrapper 클래스의 compare를 사용하자.
```Java
public class Book implements Comparable<Book> {
    private final int price;
    
    //...
    
    @Override
    public int compareTo(Book book) {
        return Integer.compare(price, book.price);
        // Integer 클래스가 제공하는 정적 메서드 compare 사용
    }
}
```
박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드 compare를 대신 이용한다.
관계연산자(<,>)는 오류를 유발할 가능성이 있기에 추천하지 않는다.

정렬 기준인 필드가 여러 개일 때

```Java
public class Book implements Comparable<Book> {
    private final String title;
    private final int price;

    //...

    @Override
    public int compareTo(Book book) {
        // Book 클래스를 가격순으로, 가격이 같다면 제목 순으로 정렬
        int result = Integer.compare(price, book.price);
        if (result == 0) result = String.CASE_INSENSITIVE_ORDER.compare(title, book.title);
        return result;
        // price값이 같을 경우 title에 대한 비교를 수행, 만약 비교하려는 필드가 추가될 경우 if문을 통한 또다른 필드 비교 추가.
    }
}
```

자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 함께 연쇄 방식으로 비교자를 생성할 수 있게 되었다.
이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는데에 활용할 수 있다.

```Java 
import java.util.Comparator;

public class Book implements Comparable<Book> {
    private final String title;
    private final int price;

    //...

    private static final Comparator<book> COMPARATOR = 
            Comparator.comparingInt((Book book) -> book.price)
                    .thenComparing(book -> book.title);
    
    @Override
    public int compareTo(Book book) {
        return COMPARATOR.compare(this, book);
    }
    // 간결하지만 약간의 성능 저하가 있다.
}
```
이 코드는 클래스를 초기화할 때 비교자 생성 메서드 2개를 이용해서 비교자를 생성한다.
comparingInt는 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아서 
그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드이다.

thenComparing은 Comparator의 인스턴스 메서드로써 키 추출자 함수를 인수로 받아 다시 비교자를 반환한다.
<br>
(이 비교자는 첫 번째 비교자를 적용한 다음 새로 추출한 키로 추가 비교를 수행함, 원하는 만큼 연달아 호출 가능)

Comparator는 수많은 보조 생성 메서드가 있다.

<br>

## Comparable? Comparator?

<hr>

| Comparable                       | Comparator            |
|:---------------------------------|:----------------------|
| 자기 자신과 매개변수 객체를 비교               | 두 매개변수 객체를 비교         |
| 주로 정렬해야 하는 클래스를 <br>구현체로 만들어서 사용 | 주로 익명 객체로 구현해서 사용     |
| lang 패키지 -> import 필요 x          | util 패키지 -> import 필요 |

<br>

## 안티패턴
<hr>
해시코드 값의 차를 기준으로  하는 비교자 (추이성 위배)

```Java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```
이 방식은 정수형 오버플로우를 일으킬 수도 있고, 부동소수점 계산 방식에 따른 오류를 발생시킬 수 있다.
그렇다고 위에 있는 다른 방법들보다 속도가 월등히 빠르지도 않다.


이 대신 다음 두 방법을 사용하자.

정적 compare 메서드를 활용한 비교자
```Java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(final Object o1, final Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
};
```

정적 compare 메서드를 활용한 비교자

```Java
static Comparator<Object> hashCodeOrder = 
Comparator.comparingInt(o -> o.hashCode());
```

<br>

## 정리

<hr>

순서를 고려해야하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야한다.
compareTo 메서드에서 필드의 값을 비교할 때 "<"와 ">" 연산자는 지양해야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator가 제공하는 비교자 생성 메서드를 이용하자.