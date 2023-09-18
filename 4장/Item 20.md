# 아이템 20. 추상 클래스 보다는 인터페이스를 우선하라
자바에는 인터페이스와 추상 클래스를 제공한다. 
또한 자바 8 부터는 인터페이스에 default method를 제공하게 되어 인퍼테이스와 추상 클래스 모두 인스턴스 메소드를 구현 형태로 제공할 수 있게되었다.
그렇다면 둘의 차이는 무엇일까? 추상 클래스를 상속받아 구현하는 클래스는 반드시 추상 클래스의 하위 타입이 되어야한다는 점이다. 
즉, 자바에서는 단일 상속만 지원 하기 때문에 한 추상 클래스를 상속받은 클래스는 다른 클래스를 상속받을 수 없게되는 것이다. 
이와 반대로 인터페이스는 구현해야할 메소드만 올바르게 구현한다면 어떤 다른 클래스를 상속했던 간에 같은 타입으로 취급된다.

### 인터페이스 장점
- 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.
- 인터페이스는 믹스인 정의에 안성맞춤이다. 
- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
- 래퍼 클래스 관용구와 함께 사용한다면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.

### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.

기존 클래스에 새로운 인터페이스를 구현하려면, 간단하게 해당 인터페이스를 클래스 선언문에 implements 구문으로 추가하고 인터페이스가 제공하는 메소드들을 구현하기만 하면 끝이다.
반면에 기존 클래스에 새로운 추상 클래스를 상속하기에는 어려움이 따른다. 두 클래스가 같은 추상 클래스를 상속하길 원한다면 계층적으로 두 클래스는 공통인 조상을 가지게 된다. 만약 두 클래스가 어떠한 연관도 없는 클래스라면 클래스 계층에 혼란을 줄 수 있다.

### 인터페이스는 믹스인 정의에 안성맞춤이다.

믹스인이란 어떤 클래스의 주 기능에 추가적인 기능을 혼합한다 하여서 믹스인이라고 한다. 그러므로 믹스인 인터페이스는 어떤 클래스의 
주 기능이외에 믹스인 인터페이스의 기능을 추가적으로 제공하게 해주는 효과를 준다.
추상 클래스가 믹스인 정의에 맞지않은 이유는 기존 클래스에 덧씌울 수 없기 때문이다. 
자바는 단일 상속을 지원하기 때문에 한 클래스가 두 부모를 가질 수 없고 부모와 자식이라는 클래스 계층에서 믹스인이 들어갈 합리적인 위치가 없다.

믹스인 인터페이스엔 대표적으로 Comparable, Cloneable, Serializable 이 존재한다.
~~~java
public class Mixin implements Comparable {
    @Override
    public int compareTo(Object o) {
    return 0;
    }
}
~~~
위의 코드와 같이 Comparable을 구현한 클래스는 같은 클래스 인스턴스 끼리는 순서를 정할 수 있는 것을 알 수 있다.
### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

현실의 개념 중에는 동물과 포유류, 파충류, 조류 와 같이 타입을 계층적으로 정의하여 구조적으로 잘 표현할 수 있는 개념이 있는가 하면 가수와 작곡가, 그리고 가수겸 작곡가(SingerSongWriter) 같이 계층적으로 표현하기 어려운 개념이 존재한다.
이런 계층구조가 없는 개념들은 인터페이스로 만들기 편하다.

가수, 작곡가 인터페이스가 존재한다. 또한 가수겸 작곡가도 존재한다. 가수와 작곡가를 인터페이스로 정의하였으니 사람이라는 클래스가 두 인터페이스를 구현해도 전혀 문제가 되지 않는다.
~~~Java
public class People implements Singer, SongWriter {
    
    @Override
    public void 노래부르기() {

    }
    
    @Override
    public void 작곡하기() {

    }
}
~~~
또한 두 인터페이스를 확장하고 새로운 메소드까지 추가한 인터페이스 또한 정의할 수 있다.
~~~Java
public interface SingerSongWriter extends Singer, SongWriter {
    void 춤도추자();
    void 작사도하자();
}
~~~
만약 이러한 구조를 추상 클래스로 만들면 어떻게 될까?
~~~Java
public abstract class Singer {
    abstract void 노래부르자();
}

public abstract class SongWriter {
    abstract void 작곡하자();
}

public abstract class SingerSongWriter {
    abstract void 춤도추자();
    abstract void 작사도하자();
    abstract void 작곡하자();
    abstract void 노래부르자();
}
~~~
추상 클래스로 만들었기 때문에 Singer 클래스와 SongWriter 클래스를 둘다 상속할 수 없어 SingerSongWriter라는 또 다른 추상 클래스를 만들어서 클래스 계층을 표현할 수 밖에 없다.
만약 이런 Singer 와 SongWriter와 같은 속성들이 많이 있다면, 그러한 클래스 계층구조를 만들기 위해 많은 조합이 필요하고 
결국엔 고도비만 계층구조가 만들어질 것이다. 이러한 현상을 조합 폭발이라고 한다.

### 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상키는 안전하고 강력한 수단이 된다.
타입을 추상 클래스로 정의해두면 해당 타입에 기능을 추가하는 방법은 상속 뿐이다. 
상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 쉽게 깨진다. (아이템18 참고)


Java8 부터는 인터페이스에서 디폴트 메서드 기능을 제공해 개발자들이 중복되는 메서드를 구현하는 수고를 덜어줄 수 있게 되었다. 
하지만 디폴트 메서드에도 단점은 존재한다. Object의 equals, hashcode 같은 메서드는 디폴트 메서드로 제공해서는 안 된다. 
또한 public이 아닌 정적 멤버도 가질 수 없다. 또한 본인이 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.
한편, 인터페이스와 추상 골격 구현 클래스를 함께 제공하면 인터페이스와 추상 클래스의 장점을 모두 가져갈 수 있다.
인터페이스로는 타입을 정의하고 필요한 일부 디폴트 메서드를 구현한다, 추상 골격 구현 클래스는 나머지 메서드들 까지 구현한다.
이렇게 하면 추상 골격 구현 클래스를 확장하는 것만으로 인터페이스를 구현하는데 대부분 일이 완료된다. 
이는 템플릿 메서드 패턴과 같다.
이런 추상 골격 구현 클래스를 보여주는 좋은 예로는 컬렉션 프레임워크의 AbstractList, AbstractSet 클래스이다.
다음은 예시이다.
~~~java
public interface Champion {
    void move();
    void stop();
    void attack();
    void process();
}
~~~
~~~java
public class Anivia implements Champion{
    @Override
    public void move() {
        System.out.println("움직임");
    }

    @Override
    public void stop() {
        System.out.println("멈춤");
    }

    @Override
    public void attack() {
        System.out.println("얼음 날리기");
    }

    @Override
    public void process() {
        move();
        attack();
        stop();
    }
}
~~~
~~~java
public class Yasuo implements Champion{
    @Override
    public void move() {
        System.out.println("움직임");
    }

    @Override
    public void stop() {
        System.out.println("멈춤");
    }

    @Override
    public void attack() {
        System.out.println("칼로 베기");
    }

    @Override
    public void process() {
        move();
        attack();
        stop();
    }
}
~~~
야스오 , 애니비아 모두 attack 메서드를 제외하고 같은 동작을 한다. 중복 코드를 제거하기 위해 
인터페이스를 추상클래스로 대체하지 않고 추상 골격 구현을 이용 중복 코드를 제거한다.
~~~java
public abstract class AbstractChampion implements Champion{
    @Override
    public void move() {
        System.out.println("움직임");
    }

    @Override
    public void stop() {
        System.out.println("멈춤");
    }
    @Override
    public void process() {
        move();
        attack();
        stop();
    }
}
~~~
~~~java
public class Anivia extends AbstractChampion implements Champion{
    @Override
    public void attack() {
        System.out.println("얼음 날리기");
    }
}
~~~
~~~java
public class Yasuo extends AbstractChampion implements Champion{
    @Override
    public void attack() {
        System.out.println("칼로 베기");
    }
}
~~~
만약 프렐요드 , 아이오니아와 같은 속성도 추가해주어야 한다면 어떻게 해야할까?
~~~Java
public class Freljord{
    public void passive(){
        System.out.println("얼음 폭풍");
    }
}
public class Anivia extends Freljord implements Champion{
    @Override
    public void move() {
        InnerAbstractChampion.move();
    }

    @Override
    public void stop() {
        InnerAbstractChampion.stop();
    }
    @Override
    public void process() {
        passive();
        InnerAbstractChampion.process();
    }
    
    private class InnerAbstractChampion extends AbstractChampion{
        @Override
        public void attack() {
            System.out.println("얼음 날리기");
        } 
    }
}
~~~
위와 같이 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고 각 메소드 호출을 내부 클래스의 인스턴스에 전달하여 골격 구현 클래스를 우회적으로 이용하는 방식을
시뮬레이트한 다중 상속(simulated multiple inheritance) 이라고 한다.

마지막으로 단순 구현(simple implementation)을 알아보자. 
단순 구현이란 골격 구현의 작은 변종으로 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니라는 점에서 차이점을 가지고 있다.
단순 구현은 추상 클래스와 다르게 그대로 써도 되거나 필요에 맞게 확장해도 된다. 단순 구현의 좋은 예로는 AbstractMap.SimpleEntry 가 있다.
~~~Java
public abstract class AbstractMap<K,V> implements Map<K,V> {
    public static class SimpleEntry<K, V>
            implements Entry<K, V>, java.io.Serializable {
        @java.io.Serial
        private static final long serialVersionUID = -8499721149061103585L;

        @SuppressWarnings("serial") // Conditionally serializable
        private final K key;
        @SuppressWarnings("serial") // Conditionally serializable
        private V value;

        /**
         * Creates an entry representing a mapping from the specified
         * key to the specified value.
         *
         * @param key the key represented by this entry
         * @param value the value represented by this entry
         */
        public SimpleEntry(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
}
~~~