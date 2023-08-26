# 아이템 10. equals 는 일반 규약을 지켜 재정의하라.
<br>

    equals 란 객체의 동등성을 확인하기 위한 메서드이며, 최대한 재정의하지 않는 것이 좋다.
<br>

## equals 재정의 하지 않는 것이 좋은 경우  


### 1. 각 인스턴스가 본질적으로 고유할 때
- 값을 표현하는것이 아니라 동작하는 개체를 표현하는 클래스

### 2. 인스턴스의 "논리적 동치"를 검사할 필요가 없는 경우
- 예를 들어 두개의 random 또는 pattern 의 인스턴스가 같은지 검사하는 것은 의미가 없다. (결과를 예측할 수 없으므로)

### 3. 상위 클래스에서 재정의한 equals 를 하위 클래스에서도 문제없이 사용 가능할 때

### 4. 클래스가 private 이거나 package-private 인 경우 
- private or package-private 클래스는 외부에서 접근할 수 없으므로, equals 메서드를 재정의하더라도 외부에서 그 차이를 알 수 없다. 


## equals 재정의 해야하는 경우

<br>

    논리적 동치성을 확인해야하 하는 경우, 주로 값 클래스(Integer, String 등) 들이 해당된다
<br>

<details><summary>논리적 동치</summary>
두 객체가 논리적으로 동일한 값을 가지고 있는지를 알려주는 것이다. 
내부 상태나 데이터를 기반으로 두 객체가 같은지 다른지를 결정한다. 

예) String a = "Hello", String b = "Hello" 
객체 a, b 는 물리적으로 다른 곳에 있지만, 내용은 같으므로 논리적으로 동치이다.

논리적 동치와 반대 되는 개념은 "객체 식별" 이다. 
실제로 동일한 객체를 가리키는지 확인하는 것인데, 두 참조가 메모리 상의 동일한 위치를 가리키면, 객체 식별에 의해 동일하다고 판단한다. 
</details>

---


## equals 메서드의 일반 규약 

### 1. 반사성(reflexivity)


- null 이 아닌 모든 참조값 x 에 대해 x.equals(x) 는 true
- 즉, 객체는 자기 자신과 같아야 한다 라는 의미 

### 2. 대칭성(symmetry)


- null 이 아닌 모든 참조값 x,y 에 대해 x.equals(y) 가 true 면, y.equals(x) 도 true 
- 두 객체는 서로에 대한 동치 여부에 똑같이 답해야한다.
~~~java
public final class Item10 {
	private final String s;

	public Item10(String s) { //생성자
		this.s = Objects.requireNonNull(s);
	}

	@Override
	public boolean equals(Object o) {
		if (o instanceof Item10) {  // Item10 이 o로 형 변환 된다면,
			// s는 item 10 클래스의 멤버변수 이고, 이 메서드는 string 클래스의 equalsIgnoreCase 호출
			// equals 의 파라미터 o 를 item10 으로 형 변환
			// 형변환된 Item10 객체의 s 멤버 변수에 접근
			return s.equalsIgnoreCase(((Item10) o).s);
		}
		if (o instanceof String) {
			return s.equalsIgnoreCase((String) o);
		}
		return false;
	}


	public static void main(String[] args) {
		Item10 item10 = new Item10("Polish");
		String s = "polish";


		System.out.println(item10.equals(s)); // (1)
		System.out.println(s.equals(s));
		System.out.println(s.equals(item10)); // (2)
	}
}
~~~
- 객체는 문자열과 비교 가능하지만(1), 문자열은 객체와의 비교를 지원하지 않는다(2). 
- 따라서, equals 비교 시 웬만하면 다른 타입의 객체와 비교하지 않도록 하자. 

### 3. 추이성(transitivity)

---
- null 이 아닌 모든 참조 값 x, y, z 에 대해, x.equals(y) 가 true 이고, y.equals(z) 도 true 이면, x.equals(z) 도 true

~~~java

public class Point {
	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override
	public boolean equals(Object o) {
		if (!(o instanceof Point)) {
			return false;
		}
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
}
~~~

~~~java
// 색상 추가
public class ColorPoint extends Point{
	private final Color color;

	public ColorPoint(int x, int y, Color color) {
		// 부모 클래스 point 의 생성자 호출하여 x,y 는 Point 의 필드에 저장
		super(x, y);
		this.color = color;
	}
	
	// 추이성 위배
	@Override
	public boolean equals(Object o) {
		if ((!(o instanceof Point))) return false;

		// o가 Point 면 색상을 무시하고 비교한다.
		if (!(o instanceof ColorPoint)) {
			return o.equals(this);
		}

		// o가 ColorPoint 면 색상까지 비교
		return super.equals(o) && ((ColorPoint) o).color == color;
	}

	public static void main(String[] args) {
		ColorPoint colorPoint = new ColorPoint(1, 2, Color.RED);
		Point p = new Point(1, 2);
		ColorPoint colorPoint1 = new ColorPoint(1, 2, Color.BLUE);

		// 색상 무시 비교
		System.out.println(colorPoint1.equals(p)); // true
		// 색상 무시 비교
		System.out.println(p.equals(colorPoint1)); // true
		// 색상까지 비교
		System.out.println(colorPoint.equals(colorPoint1)); // fasle
	}


}
~~~

<추이성 : 무한재귀 (StackOverflowError)>
~~~java
public class LocationPoint extends Point {

	private final Location location;

	public LocationPoint(int x, int y, Location location) {
		super(x, y);
		this.location = location;
	}

	@Override
	public boolean equals(Object o) {
		if (!(o instanceof Point)) {
			return false;
		}

		// o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교
		if (!(o instanceof LocationPoint)) {
			return o.equals(this);
		}

		// o가 ColorPoint이면 색상까지 비교
		return super.equals(o) && this.location == ((LocationPoint) o).location;
	}

	// 스택 오버플로우
	public static void main(String[] args) {
		Point cp = new ColorPoint(2, 3, Color.RED);
		Point sp = new LocationPoint(2, 3, Location.LEFT);

		System.out.println(cp.equals(sp));
	}
}

~~~
- cp.equals(sp) 는 	
  - if (!(o instanceof ColorPoint)) return o.equals(this); 에서 o->sp : return sp.equals(cp)
  - if (!(o instanceof LocationPoint)) return o.equals(this); 에서 o->cp : return cp.equals(sp)
  - 따라서 무한히 서로를 호출하게 된다.
> 객체 지향적 추상화의 이점을 포기 하지 않는 한, 구체 클래스를 확장해 새로운 값을 추가하면서 equals() 규약을 만족시킬 방법은 존재하지 않는다.  

<추이성 : 리스코프 치환 원칙 위배>
- equals 안의 instanceof 검사를 getClass 로 바꾸면 구체 클래스를 상속할 수 있는가? NOPE

~~~java
@Override
public boolean equals(Object o) {
    // getClass
    if (o == null || o.getClass() != this.getClass()) {
        return false;
    }

    Point p = (Point) o;
    return this.x == p.x && this.y = p.y;
}
~~~
- 같은 구현 클래스의 객체와 비교할 때만 true 를 만환한다. 
- 그러나, Point 의 하위 클래스는 Point 로써 활용될 수 있어야하지만, 여기서는 불가능하다

  => 리스코프 치환 원칙에 위배 : 리스코프치환이란 타입의 모든 메서드가 하위타입에서도 똑같이 작동해야한다는 것이다. 

<추이성 : 상속 대신 컴포지션 사용>
- 상속 대신 컴포지션을 사용하라 (아이템 18)
- equals 규약을 지키면서 값 추가하기 
~~~java
public class ColorPoint {

    private Point point;
    private Color color;

    public ColorPoint(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

	// ColorPoint 의 Point 뷰를 반환
    public Point asPoint() {
        return this.point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return this.point.equals(cp) && this.color.equals(cp.color);
    }
}
~~~
- 추상 클래스의 하위 클래스에서라면 equals 규약을 지키면서 값을 추가할 수 있다. 
- 태그 달린 클래스보다는 클래스 계층구조를 활용하라 (아이템 23) 

### 4. 일관성(consistency)

---
- null 이 아닌 모든 참조 값 x, y 에 대해 x.equals(y) 를 반복해서 호출하면 항상 true 반환하거나 항상 false 반환
- 두 객체가 같다면 수정되지 않는 한 같다는 의미이다.
- 클래스가 불변이든 가변이든 euqals 의 판단에 신뢰할 수 없는 자원이 끼어들면 안된다.
  - 예를 들어, java.net.URL 클래스는 URL과 매핑된 host의 IP주소를 이용해 비교하기 때문에 같은 도메인 주소라도 나오는 IP정보가 달라질 수 있기 때문에 반복적으로 호출할 경우 결과가 달라질 수 있다.
- 이러한 문제를 피하려면, equals 는 항상 메모리에 존재하는 객체만 사용한 결정적(deterministic)계산 만 수행해야 한다. 
  - 결정적 계산 : 항상 동일한 입력에 대해 동일한 결과를 반환하는 계산 

### 5. null 아님

---
- null 이 아닌 모든 참조 값 x 에 대해, x.equals(null) 은 false
- 즉, 모든 객체가 null 과 같지 않아야 한다. 

~~~java

@Override 
public boolean equals(Object o) {
	// 명시적 null 검사 : 필요 없음 
    if (o == null) {
        return false;
    }
    
    // instanceof 를 이용하여 올바른 타입인지 먼저 거르기
    // 묵시적 null 검사 : 추천
    if (!(o instanceof Point)) {
        return false;
    }
}
~~~


## 올바른 equals 메서드 구현 방법 


1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인하낟. 
3. 입력을 올바른 타입으로 형변환한다. 
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일지하는지 하나씩 검사한다.

~~~java
//올바른 equals  구현
@Override
public boolean equals(Object o) {

    // 1
    if (this == o) {
        return true;
    }

    // 2
    if (!(o instanceof Point)) {
        return false;
    }

    // 3
    Point p = (Point) o;

    // 4
    
    // floate && double 을 제외한 기본 타입 필드는 == 연산자로 비교
	return this.x == p.x && this.y == p.y;

	// 참조 타입 필드는 equals 메서드로 비교한다. 
	return str1.equals(str2);

	// null 도 정상 값으로 취급하는 경우, 아래를 이용하여 NPE 발생을 예방하자! 
    return Object.equals(Object, Object);
}
~~~
다 구현 했다면, 대칭적인지, 추이성이 있는지, 일관적인지 생각해보고, 단위 테스트로 확인하자 

---
## 결론 


- 꼭 필요한 경우가 아니면 재정의 하지 않는다. 
- equals 를 재정의할 경우 다섯가지 규약을 지킨다. 
- equals 의 성능을 위해 다를 가능성이 더 크거나 비교하는 비용이 싼 필드부터 비교한다. 
- equals 를 재정의할 땐 hashCode 도 반드시 재정의하자 (아이템 11)
- Object 외의 타입은 매개변수로 받는 equals 메서드는 선언하지 말자 
  - @Override 애너테이션을 일관되게 사용하라 (아이템 40)

---

내가 몰라서 정리하는 내용 



<details><summary>동치 관계</summary>
<div>

집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 것
이 부분집합을 동치류, 동치 클래스 라고 한다. 
equals 를 쓰기 위해서는 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환 가능해야 한다.

</div></details>

---
값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스

: 주로 상태나 값을 표현하는 것이 아니라 특정 작업을 수행하거나 행동하는 것에 중점을 둔 클래스

값 클래스 
- 주로 데이터나 값을 나타낸다 
- Integer, Date, String 등 
- 객체들은 상태에 의해 주로 정의된다. 

동작 클래스 
- 주로 행동이나 동작을 나타낸다 
- Thread, FileWriter, Timer 등 
- 어떤 작업을 수행하는 메서드에 의해 주로 정의된다. 
- 행동은 상태에 독립적일 수 있기 때문에 equals 메서드를 재정의하여 객체의 상태를 기반으로 동등성 판단은 의미 없을 수도 있다. 

