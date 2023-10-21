# 아이템 44. 표준 함수형 인터페이스를 사용하라.

> 함수 객체를 받는 정적 팩터리나 생성자를 이용하여 서브 클래스 재정의 없이도 유연한 동작 변경이 가능하다. 이 때, 함수형 매개변수 타입을 올바르게 선택해야 한다.
- 표준 함수형 인터페이스 : java.util.function 패키지에 도입된, 주요 함수형 프로그래밍 작업을 위한 인터페이스들의 집합

### LinkedHashMap 예시 
~~~java
public class Example {
	public static void main(String[] args) {
		LinkedHashMap<String, Integer> map = new LinkedHashMap<String, Integer>() {
			@Override
			protected boolean removeEldestEntry(Map.Entry eldest) {
				return size() > 2;
			}
		};

		map.put("a", 1); 
		map.put("b", 2);
		map.put("c", 3); 
		map.put("d", 4);

		// 결과는 {c=3, d=4}
		System.out.println(map);
	}
}
~~~
- 잘 동작하지만 람다를 사용하여 디시 구현한다면, 함수 객체를 받는 정적 팩터리나 생성자를 제공할 수 있다. 
- 재정의한 removeEldestEntry 는 size 메서드를 호출하는데, 인스턴스 메서드라 가능한 방식이다. 
- 팩터리나 생성자를 호출할 때는 Map 의 인스턴스가 존재하지 않아 Map 자신도 함수 객체에 넘겨주어야 한다. 

~~~java
// 불필요한 함수형 인터페이스 (표준 함수형 인터페이스가 따로 있다)
@FunctionalInterface interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K,V> map, Map.Entry<K, V> eldest);
}
~~~
- 함수형 인터페이스는 하나의 추상 메서드만 포함하는 인터페이스를 말한다.

### 표준 함수형 인터페이스 
- 필요한 용도에 맞는게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 사용하는 것이 좋다. 
  - API 가 다루는 개념의 수가 줄어들어 익히기 좋다. 
  - 유용한 디폴트 메서드를 많이 제공하여, 다른 코드와의 상호운용성도 좋아진다.
- java.util.function 패키지 하위에 동일한 역할을 하는 것들이 43가 있다. 
- 그 중에서 6개의 기본 인터페이스를 기억하자.
- 
  | 인터페이스                | 함수 시그니쳐 | 의미 | 예 |
  |----------------------|---------|----|---|
  | UnaryOperator< T >	  | T apply(T t)    | 반환값과 인수의 타입이 같은 함수, 인수는 1개	 | String::toLowerCase
  |
  | BinaryOperator< T >	 | T apply(T t1, T t2)    | 반환값과 인수의 타입이 같은 함수, 인수는 2개	 | BigInteger::add
  |
  | Predicate< T >	      | boolean test(T t)    | 한 개의 인수를 받아서 boolean을 반환하는 함수	 |  Collection::isEmpty
  |
  | Function< T, R >	    | R apply(T t)    | 인수와 반환 타입이 다른 함수	 | Arrays::asList
  |
  | Supplier< T >	       | T get()    | 인수를 받지 않고 값을 반환, 제공하는 함수	 | Instant::now
  |
  | Consumer< T >        | void accept(T t)    | 한 개의 인수를 받고 반환값이 없는 함수	 | System.out::println
  |
  - 표준 함수형 인터페이스 대부분은 기본타입만 지원한다. 
    - 박싱된 기본 타입을 넣어서 사용하지 말자. (아이템 61)

### 직접 작성해야 하는 경우 (고려 사항)
- 필요한 용도에 맞는게 없는 경우
- 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다. 
- 반드시 따라야하는 규약이 있다. 
- 유용한 디폴트 메서드를 제공할 수 있다. 

### @FunctionalInterface 를 사용하는 이유 
- 해당 클래스의 토그나 설명 문서를 읽는 사람에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다. 
- 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일된다. 
- 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.
- 그러므로 직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용하자

### 함수형 인터페이스를 API 에서 사용할 때의 주의점
- 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중으로 정의하면 안 된다. (아이템 52)
  - 다중정의 : 같은 이름의 메서드가 다른 매개변수 타입이나 매개변수의 수를 가지고 여러번 정의 되는 것
~~~java 
public interface ExecutorService extends Executor {
    <T> Future<T> submit(Callback<T> task);
    Future<?> submit(Runnable task);
}
~~~

## 결론
- API 를 설계할 때 람다를 염두에 두자.
- 입력값과 반환값에 함수형 인터페이스 타입을 활용하자. 
- 흔치않지만, 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을때도 있다.
