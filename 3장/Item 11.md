# Item 11 - equals를 재정의하려거든 hashCode도 재정의하라

## hashCode란?

<hr>

해시코드를 간단하게 말하면 해시 알고리즘에 의해 생성된 정수 값이다.

책에서 나온 내용을 보면 

> "hashCode 일반 규약을 어기게 되어 해당 클래스의 HashMap 이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다."

라고 나오는데, 이때 나오는 HashTable이 hashCode를 활용해서 만들어지게 된다.

HashTable은 hashCode() 메서드를 사용해서 주어진 키에 대한 해시 값을 계산하고
내부적으로 이 값을 저장하여 데이터를 저장한다.

### hashCode 일반 규약

- equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode 도 변하면 안 된다.
(애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없음)
- equals가 두 객체가 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환한다.
→ 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
- equals가 두 객체를 다르다고 판단했더라도, hashCode는 꼭 다를 필요는 없다.
하지만, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

<br>
<br>

```Java
public class Car {
    
    private String name;
    private String brand;
    
    public Car(String name, String brand) {
        this.name = name;
        this.brand = brand;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Car car = (Car) o;
        return Objects.equals(name, car.name) && Objects.equals(brand, car.brand);
    }
}
```

```Java
public class Main {
    public static void main(String[] args) {

        Car bus = new Car("bus", "cityBus");
        Car taxi = new Car("bus", "cityBus");
        
        System.out.println(bus.hashCode());   // 1747585824
        System.out.println(taxi.hashCode());  // 1023892928
        System.out.println(bus.equals(taxi)); // true
    }
}
```
equals만을 오버라이드 했을 때 hashCode가 다르지만 equals() 값은 true가 나온다.

```Java
public class Main {
    public static void main(String[] args) {

        Car bus = new Car("bus", "cityBus");
        Car taxi = new Car("bus", "cityBus");

        Map<Car, Integer> map = new HashMap<>();
        map.put(bus, 2);

        System.out.println(map.get(bus));  // 2
        System.out.println(map.get(taxi)); // null
    }
}
```
그러나 HashMap에서는 같은 키라고 인정받지 못한다.

## hashCode() 메서드 구현

<hr>

```Java
public class Car {

    private String name;
    private String brand;

    public Car(String name, String brand) {
        this.name = name;
        this.brand = brand;
    }
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Car car = (Car) o;
        return Objects.equals(name, car.name) && Objects.equals(brand, car.brand);
    }
    
    @Override
    public int hashCode() {
        return 40;
    }
}
```
최악이지만 적합한 방법의 hashCode 구현이다.(사용금지)

```Java
public class Main {
    public static void main(String[] args) {

        Car bus = new Car("bus", "cityBus");
        Car taxi = new Car("bus", "cityBus");

        System.out.println(bus.hashCode());   // 40
        System.out.println(taxi.hashCode());  // 40
        System.out.println(bus.equals(taxi)); // true

        Map<Car, Integer> map = new HashMap<>();
        map.put(bus, 2);

        System.out.println(map.get(bus));    // 2
        System.out.println(map.get(taxi));   // 2
    }
}
```
모두 정상적으로 동작한다.

HashMap이 동일한 key인지 판단할 때, equals()와 hashCode(), 이 두 메서드에 의존하기 때문이다.
HashMap은 해시코드가 서로 다른 엔트리끼리는 동치성 비교를 시도조차 않도록 최적화 되어있다.

HashTable에서는 객체를 꺼내올 때 해당 객체의 hashCode() 메서드를 호출하여 객체의 해시 코드를 얻고,
이 해시 코드를 기반으로 어떤 버킷에 해당 객체가 저장되어 있는지 결정된다.
선택된 버킷 내부에서 객체들은 동치성 비교를 위해서 순차적으로 equals() 메서드를 호출하여 객체들 간의 동치성을 판단하게 된다.

위 코드처럼 계속 같은 hashCode를 반환해도 동작은 하지만, 원소가 늘어날수록 해시 테이블의 성능이 떨어진다.
O(1)의 시간복잡도가 점차 LinkedList처럼 O(n)이 되어갈 것이다.

모든 객체가 해시 테이블의 버킷 하나에 담기기 때문이다.

## 좋은 hashCode() 메서드 구현

<hr>

좋은 hashCode를 작성하는 요령

1. int 변수인 result를 선언한 후 값을 c로 초기화한다.
    - 이 때, c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드이다.
    - 여기서 핵심 필드는 equals 비교에 사용되는 필드를 말한다.
2. 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
   1. 해당 필드의 해시코드 c 를 계산한다.
        - 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본타입의 박싱 클래스다.
        - 참조 타입 필드면서, 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출하여 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
        - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.
        - 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
   2. 단계 2.1에서 계산한 해시코드 c로 result를 갱신한다.
        - result = 31 * result + c;
3. result를 반환한다.

<br>

주의할 점

 - equals비교에 사용되는 필드에 대해서만 해시코드를 계산한다.
 - 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
 - 만약 hash로 바꾸려는 필드가 기본 타입이 아니면 해당 필드의 hashCode를 불러 구현한다.
 - 계산이 복잡한 경우는 표준형을 만들어 구현한다.
 - 참조 타입 필드가 null일 경우 0을 사용한다.
 - 31을 곱하는 이유는 비슷한 필드가 여러개일 때 해시효과를 크게 높여주기 위해서다.(홀수이면서 소수, 31 * i = (i << 5) - i 이기 때문에 시프트 연산과 뺄셈으로 대체해서 최적화할 수 있음)

```Java
@Override
public int hashCode() {
    int result = Integer.hashCode(areaCode);
    result = 31 * result + Integer.hashCode(prefix);
    result = 31 * result + Integer.hashCode(lineNum);
    return result;
}
```
책에 나오는 전형적인 hashCode 메서드이다.

PhoneNumber 인스턴스의 핵심 필드 3개를 사용해 간단한 계산을 수행하고, 이 과정에 비결정적 요소는 없다.
동치인 PhoneNumber 인스턴스는 서로 같은 해시코드를 가질 것이 확실하다.

```Java
    public class Car {

    private String name;
    private String brand;

    public Car(String name, String brand) {
        this.name = name;
        this.brand = brand;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Car car = (Car) o;
        return Objects.equals(name, car.name) && Objects.equals(brand, car.brand);
    }

    @Override
    public int hashCode() {
        int result = name.hashCode();
        result = 31 * result + brand.hashCode();

        return result;
    }
}
```
마찬가지로 Car 인스턴스의 핵심 필드 2개를 이용해서 계산을 수행하고, 이 과정에서 비결정적 요소는 없다.

```Java
public class Main {
    public static void main(String[] args) throws IOException {

        Car bus = new Car("bus", "cityBus");
        Car taxi = new Car("bus", "cityBus");

        System.out.println(bus.hashCode());   // 3133440
        System.out.println(taxi.hashCode());  // 3133440
        System.out.println(bus.equals(taxi)); // true

        Map<Car, Integer> map = new HashMap<>();
        map.put(bus, 2);

        System.out.println(map.get(bus));   // 2
        System.out.println(map.get(taxi));  // 2
    }
}
```
hashCode가 일치하고 equals 메서드에서도 true를 반환하기 때문에 map.get(taxi) 메서드를 호출했을 때 
null이 아닌 map.get(bus)의 값과 같은 2를 반환하고 있다.

<br>

Objects 클래스의 hashCode 메서드 - 성능이 살짝 아쉽다. 
입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본타입이 있다면 박싱과 언박싱도 거쳐야 하기 때문이다.

```Java
@Override
public int hashCode() {
    return Objects.hash(name);
}
```

## 캐싱과 지연 초기화

<hr>

- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다 캐싱을 고려해야 한다. 
    - 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해 둔다.
```Java
// 캐싱 예시 코드
public class Car {

    private String name;
    private String brand;
    private int cachedHashCode;

    public Car(String name, String brand) {
        this.name = name;
        this.brand = brand;
        this.cachedHashCode = computeHashCode();
    }

    private int computeHashCode() {
        int result = name.hashCode();
        result = 31 * result + brand.hashCode();
        return result;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Car car = (Car) o;
        return Objects.equals(name, car.name) && Objects.equals(brand, car.brand);
    }

    @Override
    public int hashCode() {
        return cachedHashCode;
    }
}

```

- 해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화를하면 좋다.
    - 필드를 지연 초기화 하려면 그 클래스가 thread-safe가 되도록 동기화에 신경 쓰는 것이 좋다.

```Java
// 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안전성까지 고려해야 한다.
private int hashCode;

@Override
public int hashCode() {
      	int result = hashCode; // 초기값 0을 가진다.
        if(result == 0) {
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        hashCode = result;
        }
        return result;
}
```
동시에 여러 쓰레드가 hashCode를 호출하면 여러 쓰레드가 동시에 계산하여 동시에 여러번 계산하는 상황이 발생할 수 있다.
그래서 지연 초기화를 하려면 동기화를 신경써주는 것이 좋다.

### 주의할 점

<hr>

 - 성능을 위해 해시코드 계산할 때 핵심 필드를 생략해서는 안된다.
   - 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수 있다.
 - hashcode 값의 생성 규칙을 API 사용자에게 자세히 공표해서는 안된다.
   - 이렇게 해야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.