# 아이템 17. 변경 가능성을 최소화하라


### 불변 클래스 만드는 방법 
> 1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다. 
>    - setter 사용하지 않는다. 
> 2. 클래스를 확장할 수 없도록 한다. (상속 금지) 
>    * 하위 클래스에서 객체의 상태를 변화하는 것을 막는다. 
> 3. 모든 필드를 final 로 선언한다. 
>    * 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러낸다. 
> 4. 모든 필드를 private 으로 선언한다. 
>    * 필드가 참조하는 가변 객체를 직접 접근해 수정하는 일을 막아준다.
> 5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다. 
>    * 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야한다. 
>    * 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행하라. 
>      * 원시 타입의 필드는 private final 이면 불변이지만, 참조 타입의 필드는 private final 이어도 가변일 수 있다.  


### 불변 객체의 장점
  - 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다. 
    - 여러 스레드가 동시에 사용해도 훼손되지 않는다. 
  - 불변 객체는 자유롭게 공유할 수 있으며, 불변 객체끼리는적 내부 데이터를 공유할 수 있다. 
    - 정적 팩토리를 사용하면 쉽다. (생성자를 private 으로 두어 상속을 막는다)
    - 방어적 복사조차 필요 없다. 
    - 불변 클래스는 clone 메서드나 복사 생성자를 제공하지 않는 것이 좋다. 
  - 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다. 
  - 불변 객체는 그 자체로 실패 원자성을 제공한다. 

### 불변 객체의 단점
  - 값이 다르면 반드시 독립된 객체로 만들어야 한다. 
    - 이를 해결하기 위해 가변 동반 클래스를 제공한다. 예를 들어 String 클래스의 가변 동반클래스로 StringBuilder

## 결론
- 게터가 있다고 무조건 세터를 만들지 말자. 
- 클래스는 꼭 필요한 경우가 아니면 불변이어야 한다. 
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
- 합당한 이유가 없다면 모든 필드는 private final 이어야 한다. 
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다. 
---

<details><summary>얕은 복사, 방어적 복사, 깊은 복사</summary><div>

### 얕은 복사 
- 얕은 복사는 원 본 객체의 멤버 변수들의 값만을 복사하는 방식이므로, 원본 객체와 복사본 객체의 기본타입의 멤버 변수는 독립적이다. 
- 하지만, 참조 타입의 멤버 변수는 객체의 주소값을 복사하여 복사본이 원본과 같은 객체를 참조한다. 
- 복사본에서 객체 내부의 값을 변경하면 원본 객체도 영향을 받는다. 
- 자바에서 객체의 얕은 복사는 = 연산자 사용
  - 그럼 언제 사용하는게 좋을까?
    - 깊은 복사보다 속도가 빨라야 할 때
    - 특정 상황에서 내부 데이터나 상태를 공유해야할 때 
    - 어떤 객체에 일시적인 작업을 수행할 때
  ~~~Java
  class Person {
    String name;
    Address address;
  }
  
  class Address {
    String city;
    String street;
  }
  
  public static void main(String[] args) {
    Person person1 = new Person();
    person1.name = "John";
    person1.address = new Address();
    person1.address.city = "New York";

    Person person2 = person1; // 얕은 복사

    System.out.println(person1.address.city); // 출력: New York
    System.out.println(person2.address.city); // 출력: New York

    person2.address.city = "Los Angeles";

    System.out.println(person1.address.city); // 출력: Los Angeles
    System.out.println(person2.address.city); // 출력: Los Angeles
  }

  ~~~
  Person 객체를 얕은 복사한다면...
  - name 은 기본 타입으로 복사본 Person의 name 은 독립적인 값을 갖는다.
  - 복사본 Person 의 address 는 참조 타입이므로 원본의 address 동일한 객체를 참조한다. 
  - 복사본 Person 의 address 의 city or street 변경하면, 원본도 영향을 받는다.  

### 방어적 복사
- 복사본이 원본 주소를 그대로 참조하지 않지만, 복사본 객체 내부에 있는 객체들은 원본과 동일한 주소를 참조한다. 
- List 는 같은 주소를 참조하지 않지만, List 내 각 요소들은 원본과 동일한 주소를 참조한다. 
- 요소를 변경할 수 있다는 의미이다. 
  - 왜 사용할까?
    - 객체의 내부 상태를 외부에 직접 노출시키지 않고, 복사본을 제공하여 객체의 불변성을 유지할 때 주로 쓴다.
  ~~~java
  class Person {
    String name;
    Address address;

    // 생성자
    public Person(String name, Address address) {
        this.name = name;
        this.address = new Address(address); // 방어적 복사
    }

    // Person 생성자 복사 
    public Person(Person original) {
        this.name = original.name;
        this.address = new Address(original.address); // 방어적 복사
    }
  }

  class Address {
  String city;

    // 생성자 
    public Address(String city) {
        this.city = city;
    }

    // Address 생성자 복사 
    public Address(Address original) {
        this.city = original.city;
    }
  }

  public static void main(String[] args) {
  Person person1 = new Person("John", new Address("New York"));

    // 방어적 복사 
    Person person2 = new Person(person1);

    System.out.println(person1.address.city); // 출력: New York
    System.out.println(person2.address.city); // 출력: New York

    person2.address.city = "Los Angeles";

    System.out.println(person1.address.city); // 출력: New York
    System.out.println(person2.address.city); // 출력: Los Angeles
  }

  ~~~

### 깊은 복사 
- 객체의 모든 내부 상태를 완전히 복사하여 새로운 객체를 만드는 방식이다. 
- 원본 객체 내부의 모든 객체들도 재귀적으로 복사한다. 
- 복사된 객체와 원본 객체는 서로 다른 객체를 참조하고, 하나의 객체를 수정하더라도 다른 객체에 영향을 미치치 않는다. 

</div></details>
<details><summary>불변 클래스</summary><div>

불변클래스란 그 클래스 정보로 생성된 인스턴스의 내부 값들은 생성된 후에는 수정할 수 없어야한다. 불변 인스턴스의 정보는 객체가 소멸되기 전까지 달라져서는 안된다.
예를 들어 String, BigInteger, 박싱된 클래스들이다.  
불변 클래스는 가변 클래스보다 설계가 구현하고 사용하기 쉽고, 오류가 생길 여지도 적어 훨씬 안전하다. 
~~~java
// 코드 일부
public class BigInteger extends Number implements Comparable<BigInteger> {
    final int signum;
    final int[] mag;
    
    // ...코드 생략 
    
    public BigInteger negate() {
        return new BigInteger(this.mag, -this.signum);
    }
}
~~~
</div>
</details>