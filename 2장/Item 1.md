# 정적 팩터리 메서드
정적 팩터리 메서드는 **객체 생성 역할을 하는 클래스 메서드**다.

## 장점1. 이름을 가질 수 있다.
생성자 자체는 생성되는 객체의 특성을 직관적으로 설명하지는 않는다.

``` java
    public class Shape {
    private String type;

    private Shape(String type) {
        this.type = type;
    }

    public String getType() {
        return type;
    }

    public static Shape createCircle() {
        return new Shape("Circle");
    }

    public static Shape createRectangle() {
        return new Shape("Rectangle");
    }
}
```
이렇게 메서드 명을 지음으로서 어떤 객체가 생성되는지 더 구체적으로 알 수 있다.

같은 타입을 파라미터로 받는 생성자 두 개를 만들 수 없는데 정적 팩토리 메서드로 해결이 가능.

``` java

public class Shape {
    private String type;
    private String color;

    public Shape(String type) {
        this.type = type;
    }
    public Shape(String color) {
        this.color = color;
    }
}

```
위처럼 같은 타입의 파라미터로 받는 생성자를 받게되면 컴파일 오류 발생.
``` java

public class Shape {
    String type;
    String color;
    
    private Shape(){}

    private Shape(String type) {
        this.type = type;
    }

    public static Shape decideType(String type) {
        return new Shape(type);
    }

    public static Shape decideColor(String color) {
        Shape shape = new Shape();
        shape.color = color;
        return shape;
    }
}

```

## 장점2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

``` java

public class Singleton {
    private static Singleton instance;

    private Singleton() {
        // private 생성자
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton(); // 최초 호출 시에만 인스턴스 생성
        }
        return instance;
    }

    // 나머지 메서드들...
}
```

호출될 때마다 인스턴스를 새로 생성하지 않아도 된다는것의 대표적인 예시가 싱글톤 패턴이다.
싱글톤 패턴은 소프트웨어 디자인 패턴 중 하나로, 특정 클래스의 인스턴스가 오직 하나만 생성되도록 보장하는 패턴이다.
이를 통해 전역적으로 사용되는 인스턴스를 효율적으로 관리하고, 중복 생성을 방지하여 메모리와 자원을 절약할 수 있다.

# 장점3,4. 반환 타입의 하위 타입 객체 반환 가능 , 입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있다.
리턴 타입은 인터페이스로 지정해서 구현체는 노출시키지 않을 수 있다.

``` java

public interface TrashCan {
    void save(int[] trash);
    void clean();

    static TrashCan getTrashCan(int[] trash){
        if(trash.length>100){
            BigTrashCan bigTrashCan = new BigTrashCan();
            bigTrashCan.save(trash);
            return bigTrashCan;
        }else{
            SmallTrashCan smallTrashCan = new SmallTrashCan();
            smallTrashCan.save(trash);
            return smallTrashCan;
        }
    }

}
```

``` java
public class BigTrashCan implements TrashCan {
    int[] canVolume = new int[1000];
    int index = 0;
    @Override
    public void save(int[] trash) {
        for(int i=0;i<trash.length;i++){
            canVolume[index++] = trash[i];
        }
    }

    @Override
    public void clean() {
        canVolume = new int[1000];
    }
}
```

``` java
public class SmallTrashCan implements TrashCan{
    int[] canVolume = new int[100];
    int index = 0;
    @Override
    public void save(int[] trash) {
        for(int i=0;i<trash.length;i++){
            canVolume[index++] = trash[i];
        }
    }

    @Override
    public void clean() {
        canVolume = new int[100];
    }
}
```

API를 만들 떄 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다. (장점 3에 해당)
쓰레기통을 생성하는데 쓰레기의 사이즈를 보고 다른 쓰레기통을 반환해준다. (장점 4에 해당)


# 장점 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
인터페이스나 클래스가 만들어지는 시점에서 하위 타입의 클래스가 존재하지 않아도 나중에 만들 클래스가 기존의 인터페이스나 클래스를 상속 받으면 언제든지 의존성을 주입 받아서 사용가능하다. 반환값이 인터페이스가 되며 정적 팩터리 메서드이 변경없이 구현체를 바꿔 끼울 수 있다.

해당 장점은 OCP 원칙에 해당한다고 생각된다.
정적 팩터리 메서드가 반환할 객체의 클래스가 존재하지 않아도 되는 이점은 새로운 클래스가 추가될 때 기존 정적 팩터리 메서드를 수정하지 않고도 해당 클래스의 인스턴스를 반환할 수 있다는 것을 의미한다.
이는 OCP 원칙을 준수한 설계로 볼 수 있다. 즉, 코드를 수정하지 않고도 새로운 클래스를 추가하고 해당 클래스의 인스턴스를 반환함으로써 기능을 확장할 수 있다.



# 단점1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
그래서 `java.util.Collections`는 상속할 수 없다.

상속보다 컴포지션을 사용(아이템18) 도록 유도하고 불변 타입(아이템17)로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점이 될 수도 있기는 하다.

# 단점2. 프로그래머가 찾기 어렵다.
생성자는 Javadoc이 자동으로 상단에 모아서 보여준다. 정적 팩토리 메서드는 그렇지 않다.

# 컨벤션
- from : 하나의 매개 변수를 받아서 객체를 생성
    - ex. Date date = Date.from(instant);
- of : 여러개의 매개 변수를 받아서 객체를 생성
- valueOf: from과 of의 더 자세한 버전
    - BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
- getInstance | instance : 인스턴스를 생성. 이전에 반환했던 것과 같을 수 있음.
- newInstance | create : 새로운 인스턴스를 생성
- get[OtherType] : 다른 타입의 인스턴스를 생성. 이전에 반환했던 것과 같을 수 있음.
- new[OtherType] : 다른 타입의 새로운 인스턴스를 생성.
