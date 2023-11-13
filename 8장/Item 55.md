# 아이템 55. Optional 반환은 신중히 하라

## 자바8 이전에는 메서드에서 반환할 값이 없는 경우 두가지 선택지가 있었다.
1. 예외를 던진다.
    단점 1 : 예외는 반드시 예외적인 상황에서만 사용해야 한다.
    단점 2 : 예외는 실행 스택의 추적을 캡쳐하기 때문에 비용이 비싸다.
2. 반환타입이 객체인 경우 null 반환
    단점 1 : 클라이언트 코드에서는 항상 null 체크 로직을 추가해야 하고 그렇지 않다면
NullPointerException이 발생한다.

## Optional
- Optional은 원소 하나를 가지는 불변 컬렉션.
- 자바 8이전의 코드보다 null-safe한 로직을 처리할 수 있게끔 해준다.
- Optional을 반환하여 조금 더 유연한 로직을 작성할 수 있게끔 해준다.

### Optional 주요 메서드

#### Optional.empty()
내부 값이 비어있는 Optional 객체를 반환
~~~Java
public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
~~~
#### Optional.of(T value)
내부 값이 value인 Optional 객체를 반환 , 
만약 value가 null이면 NullPointerException 발생
~~~Java
public final class Optional<T> {

    private Optional(T value) {
        this.value = Objects.requireNonNull(value); // null인 경우 NPE 발생
    }
    
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
}
~~~
#### Optional.ofNullable(T value)
value가 null이면 empty Optional 객체를 반환하고, null이 아닌 경우 Optional.of 메서드로 객체 생성 후 반환.
#### T get()
Optional 내의 값을 반환. 만약 Optional 내부 값이 null인 경우 NoSuchElementException이 발생.
#### boolean isPresent()
Optional 내부의 값이 있으면 true, null인 경우 false를 반환.
#### boolean isEmpty()
Optional 내부의 값이 null이라면 true, null이 아닌 경우 false를 반환.
#### Optional.filter
Optional에 filter 조건을 걸어 조건에 맞을 때만 Optional 내부 값을 사용.
조건이 맞지 않으면 Optional.empty를 반환.
~~~Java
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent()) {
        return this;
    } else {
        return predicate.test(value) ? this : empty();
    }
}
~~~
#### Optional.map
Optional 내부의 값을 Function을 통해 가공.
~~~Java
public <U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent()) {
        return empty();
    } else {
        return Optional.ofNullable(mapper.apply(value));
    }
}
~~~

### 반환타입을 Optional로 사용하면 안되는 경우
- 반환값으로 옵셔널을 사용한다고해서 무조건 득이 되는것은 아님. 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다. 
빈 Optional<List<T>>를 반환하기 보다는 List<T>를 반환하는게 좋다.
- 빈 List를 반환하는게 좋은 이유는 Optional로 반환하게 된다면 클라이언트에서 Optional 처리 코드를 넣어야 한다.
- 박싱된 기본 타입을 담은 Optional은 기본 타입 자체보다 무거울 수밖에 없다. 값을 두 겹이나 감싸기 때문.
- 그래서 자바 API 설계자들은 int, long, double 전용 Optional 클래스를 준비해놓았다. 이 Optional들도 Optional<T>가 제공하는 메서드를 거의 다 제공한다.
- 이렇게 대체제까지 있으니 박싱된 기본 타입을 담은Optional을 반환하는 일은 없도록 해야함. 다만 Boolean, Byte, Char 등은 예외일 수 있다.
### 반환타입을 Optional로 해야하는 경우
- 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T>를 반환.
- 하지만 Optional<T>를 반환하는 데는 댓가가 따른다. Optional도 엄연히 새로 할당하고 초기화해야하는 객체이고, 그 안에서 값을 꺼내려면 메서드를 호츨해야 하니, 한 단계를 더 거치는 셈. 그래서 성능이 중요한 상황에서는 적절히 고려.

## 정리
- Optional을 사용하여 반환하는 경우 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 반환하기보다는 컨테이너 자체를 반환하는게 좋다.
- 박싱된 기본 타입(int, long, double)을 담은 옵셔널을 반환하지 않도록 하자. 다만 Boolean, Byte, Char등은 예외
