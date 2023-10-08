# [Item 33] - 타입 안전 이종 컨테이너를 고려하라

## 타입 안전 이종 컨테이너 패턴

<hr>

컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해준다.

즉, 각 타입의 `Class` 객체를 매개변수화한 키 역할로 사용하는 것이다. `class` 리터럴의 타입은 `Class`가 아닌 `Class<T>`이다.
예를 들어, `String.class`의 타입은 `Class<String>`이고 `Integer.class`의 타입은 `Class<Integer>`이다.

    제네릭은 Set<E>, Map<K, V> 등의 컬렉션과 ThreadLocal<T>, AtomicReference<T> 등의 단일원소 컨테이너에 등에 흔히 쓰이는데, 
    이런 모든 쓰임에서 매개변수화되는 대상은 (원소가 아닌) 컨테이너 자신입니다. 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다.

이러한 설계 방식을 타입 안전 이종 컨테이너 패턴이라 한다.

## 타입 안전 이종 컨테이너 구현

<hr>

[타입 안전 이종 컨테이너 - 인스턴스 저장, 검색기능의 Favorites 클래스]

```Java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();
    
    public <T> void putFavorite(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type), instance);
    } // 클래스의 리터럴 타입은 Class가 아니라 Class<T>이다.

    public <T> T getFavorite(Class<T> type){
        return type.cast(favorites.get(type));
        // Object 타입의 객체(favorites.get(type)를 꺼내 T로 바꿔 반환해야 한다.
        // cast메서드로 이 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환 한다.
    } 
    
}
public class Main {
    public static void main(String[] args) {
        Favorites favorites = new Favorites();
        favorites.putFavorite(String.class, "morning");
        // String의 클래스 타입은 Class<String>이다.
        favorites.putFavorite(Integer.class, 0xcafebabe);
        // Integer의 클래스 타입은 Class<Integer>이다.
        favorites.putFavorite(Class.class, Favorites.class);

        String favoriteString = favorites.getFavorite(String.class);
        Integer favoriteInteger = favorites.getFavorite(Integer.class);
        Class<?> favoriteClass = favorites.getFavorite(Class.class);

        System.out.printf("%s %x %s", favoriteString, favoriteInteger, favoriteClass.getName());
        // 실행결과: morning cafebabe Favorites
    }
}
```

Favorites 인스턴스는 타입 안전하다. String 타입을 요청했는데 Integer를 반환하는일은 절대 없기 때문이다.

모든 키의 타입이 제각각이라, 일반적인 맵과 달리 여러가지 타입의 원소를 담을 수 있다.

따라서 Favorites는 타입 안전 이종 컨테이너라 할 만하다.

    컴파일타임 타입정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 타입토큰(Type Token) 이라고 한다.

<br>

Favorite 클래스에서 타입 안전을 보장하는 비결은 cast 메서드에 있다.

그 이유는 cast 메서드의 반환 타입은 Class 객체의 타입 매개변수와 같다. 즉, cast 메서드는 Class 클래스가 제네릭이라는 이점을 잘 활용한다.

```Java
public class Class<T> {
	T cast(Object object);
}
```

이 기능은 getFavorite 메서드에 필요한 기능으로 T로 비검사 형변환하는 과정 없이도 Favorites를 타입 안전하게 만들어준다.

## 동적 형변환 > 타입 안정성 확보

<hr>

만약, 클라이언트가 Class 객체를 (제네릭이 아닌) 로(Raw) 타입으로 넘기면 타입안정성이 깨지게 된다.

하지만, 이렇게 로 타입을 넘길경우 컴파일시 비검사 경고가 뜰 것이다.

만약, 타입 안전성을 확보하고 싶다면 값(value) 인수로 주어진 타입이 키(key)로 명시한 타입과 같은지 확인하면 된다.

```Java
public <T> void putFavorite(Class<T> type, T instance){
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

java.util.Collections의 checkedSet, checkedList, checkedMap 등이 있는데,
이들은 모두 제네릭이라 Class 객체와 컬렉션의 컴파일타임 타입이 같음을 보장하고 이 래퍼틀은 내부 컬렉션들을 실체화한다.

예컨대, 런타임에 Coin을 Collections<Stamp>에 넣으려하면 ClassCastException을 던진다.

즉, 이 래퍼들은 제네릭과 로 타입을 섞어서 사용하는 애플리케이션에서 클라이언트 코드가 컬렉션에 잘못된 타입의 원소를 넣지 못하게 추적하는데 도움을 준다.

## 실체화 불가 타입에는 사용할 수 없다.

<hr>

String이나 String[]은 사용할 수 있지만 `List<String>`은 사용할 수 없다.

`List<String>`을 사용하려는 코드는 컴파일 되지 않는다. 그 이유는 `List<String>`용 Class 객체를 얻을 수 없기 때문이다.

`List<String>.class`라고 쓰면 문법 오류가 발생한다. `List<String>`과 `List<Integer>`는 `List.class`라는 같은 Class 객체를 공유하므로 같은 타입의 객체 참조를 반환한다면 객체 내부에서 이들을 구분할 방법이 없어진다.

이 제약에 대한 완벽히 만족스런 우회로는 없다.

    옮긴이에 의하면 이 제약을 슈퍼 타입 토큰으로 해결하려는 시도가 있다고 나온다. 실제로 아주 유용한 방법이라 스프링 프레임워크에서는 
    ParameterizedTypeReference라는 클래스로 구현해놓았다고 한다. 그러나 이 또한 완벽한 방법이 아니라 그런지 책의 저자는 이 방법에 대해서 언급하지 않았다고 한다.
    

참고
<br>
[슈퍼 타입 토큰 방식과 원리](https://gafter.blogspot.com/2006/12/super-type-tokens.html) <br>
[이 방식의 한계](https://gafter.blogspot.com/2007/05/limitation-of-super-type-tokens.html)

## 한정적 타입 토큰

<hr>

```Java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();
    
    public <T> void putFavorite(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type), instance);
    } 

    public <T> T getFavorite(Class<T> type){
        return type.cast(favorites.get(type));
    } 
    
}
```

Favorites가 사용하는 타입 토큰은 비한정적이라 getFavorite과 putFavorite 어떤 Class 객체도 받아들인다.

이 메서드들이 허용하는 타입을 제한하고 싶다면 한정적 타입 토큰을 활용하면 된다.

    한정적 타입 토큰: 단순히 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입 토큰이다.


애너테이션 API는 한정적 타입 토큰을 적극적으로 사용한다.

[AnnotatedElement메서드 - 애너테이션을 런타임에 읽는 기능]

```Java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```

annotationType인수는 애너테이션 타입을 뜻하는 한정적 타입 토큰이다.

이 메서드는 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려있다면 그 애너테이션을 반환하고 없다면 null을 반환한다.

즉, 애너테이션된 요소는 그 키가 애너테이션 타입인 타입 안전 이종 컨테이너이다.

[asSubClass - 한정적 타입토큰을 안전하게 형변환]

```Java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName){
        Class<?> annotationType = null; // 비한정적 타입 토큰
        try {
            annotationType = Class.forName(annotationTypeName);
        }catch (Exception exception){   
            throw new IllegalArgumentException(exception);
        }
        return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```
형변환을 안전하게 그리고 동적으로 수행해주는 asSubclass메서드를 이용해 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다. 실패시 ClassCastException을 던진다.

(형변환 된다는 것은 이 클래스가 인수로 명시한 클래스의 하위 클래스라는 의미이다.)

## 정리

<hr>

컬렉션 API로 대표되는 일반적인 제네릭형태는 한 컨테이너가 다룰수 있는 매개변수의 수가 고정적이다.

하지만 컨테이너자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다.

타입 이종 컨테이너는 Class를 키로 사용하며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다.

또한 직접 구현한 키 타입도 쓸 수 있다. 예컨데 DB의 행(컨테이너)을 표현한 DatabaseRow 타입에는 제네릭 타입인 Column<T>를 사용할 수 있다.

하지만 타입이종 컨테이너를 사용하는데 제약이 있으니 이런 제약들을 주의해서 사용하자.
