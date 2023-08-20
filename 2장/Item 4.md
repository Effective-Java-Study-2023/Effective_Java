# 아이템4. 인스턴스화를 막으려거든 private 생성자를 사용하라

객체 지향적으로 쓰일 수는 없지만, 정적 메소드, 정적 필드만 담은 클래스를 유용하게 쓰일 때가 있다.
(보통 이런 클래스를 유틸리티 클래스라고 한다.)
<details><summary>위반사항</summary>

1. 정적 메소드와 정적 필드를 담은 클래스는 상태를 가지지 않으므로, 상태와 동작을 함께 캡슐화하지 못한다.
2. 보통 final 로 선언 되므로 상속할 수 없다. 
3. 인스턴스를 생성하지 않기 때문에 다형성 원칙에 부합하지 않는다. (오버라이드 불가)
</details>

1. 기본 타입 값이나 배열 관련 메서드들을 모아 놓을 수 있다. 
   * java.lang.Math, java.util.Arrays 등이 있다. 
   * Math 를 자세히 들여다 보자 

    ```java
    public final class Math {
        private Math(){}
   
        public static final double PI = 3.14159265358979323846;
   
        public static int max(int a, int b) {
            return (a >= b) ? a : b;
        }
    }
    ```
   * 내부 메서드를 보면 전부 static 으로 선언 되어 있으며 생성자가 private 으로 막혀 있다. 
   생성자가 private 이니 클래스의 인스턴스화를 막을 수 있다. 
   
2. 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드(또는 팩터리)를 모아 놓을 수 있다.  
   * java.util.Collections가 대표적인 예이다.
   * 아래의 예시는 인스턴스 Comparable 을 구현하는 객체를 생성해주는 정적 메서드이다. 
   ```java
   public class Collections {
     private Collections() {}
   
     public static <T extends Comparable<? super T>> void sort(List<T> list) { list.sort(null); }
    
   }
   ```
   * 자바 8부터 이런 메서드를 인터페이스에 넣을 수도 있다. 
   * <details> <summary>예시 코드</summary>
        <div>
       
        ```java
        // static 메서드를 담은 인터페이스 
        public interface MyInterface {
            void myMeThod();
            static String staticMethod() {
                return "인터페이스~!";
            }    
        }
       
       "-----------------"
     
        public class MyCalss implements MyInterface {
        
            @Override
            public void myMethod() {
       		    System.out.println("myMethod");
            }
            public static void main(String[] args) {
		        MyClass myClass = new MyClass();
		        myClass.myMethod();

        		// 인터페이스의 정적 메서드 호출
	        	String result = MyInterface.staticMethod();
		        System.out.println("result = " + result);
            }
        }
        ```
        </div>
     </details>
    
3. final 클래스와 관련한 메서드들을 모아 놓을 수 있다. 
   * final 클래스를 상속할 수 없다.
   * <details><summary>예시 코드 </summary>
     final 클래스의 대표적인 예는 String 과 Math 이다. 
     
     ```java
     public final class String implements java.io.Serializable, Comparable<String>, CharSequence,Constable, ConstantDesc { ... }
     ```
     
     </details>

---
보통 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 것이 아니다.   

*의도치 않은 인스턴스 화를 막기 위해서, private 생성자를 추가 하여 클래스의 인스턴스화를 막는다.*
* 생성자를 명시하지 않으면, 컴파일러가 자동으로 기본 생성자로 만들고 이 생성자는 public 이다. 
* 추상 클래스로 만든다 하더라도, 하위 클래스를 만들어 인스턴스화 할 수 있다. 
* 이렇게 private 생성자를 만들면 상속을 불가능하게 할 수 있다. 
* 생성자가 있으나 호출이 불가능하므로, 오해하지 않도록 적절한 주석을 달아 설명해주자.

---
<details>
<summary>static "정적" 은 무엇이고, 언제 붙일까?</summary>
<div>
static 은 '클래스의' 또는 '공통적인'의 의미를 가지고 있다. 인스턴스 변수는 하나의 클래스로부터 생성되었더라도, 다른 값을 유지하지만, 
클래스 변수 즉, static 멤버 변수는 인스턴스에 관계 없이 같은 값을 갖는다. 그 이유는 하나의 변수를 모든 인스턴스가 공유하기 때문이다. 

* 그렇다면 언제 붙이는 것이 좋을까?
  * 클래스의 멤버변수 중 모든 인스턴스에 공통된 값을 유지해야하는 것이 있다면 붙여준다.
  * static 이 붙은 변수(클래스 변수)는 클래스가 메모리에 올라갈 때 이미 자동적으로 생성되어 인스턴스 생성하지 않아도 사용 가능하다.
  * 작성한 메서드 중에서 인스턴스 변수나 인스턴스 메서드를 사용하지 않는 메서드에 고려해본다. 
    * 클레스 메서드는 인스턴스 변수를 사용할 수 없다.
    * 이렇게 되면, 메서드 호출 시간이 짧아지므로 성능이 향상된다. 
    * 인스턴스 메서드는 실행 시 호출되어야할 메서드를 찾는 과정이 추가적으로 든다. 
* static 이 사용될 수 있는 곳 : 멤버변수, 메서드, 초기화 블럭
</div>
</details>

<details>
<summary>그렇다면 어떻게 불러올 수 있는가?</summary>
<details>
Math 나 Arrays 는 인스턴스화가 불가능한데, 어떻게 사용할 수가 있는가?  

* 같은 패키지 안에 있다면, 그냥 클래스 이름을 통해 직접 접근 가능하다. 
  * 만약 다른 패키지에 있다면, import 문을 사용하여 클래스를 가져와야 한다. :) 
<summary>예시 코드</summary>
<div>

```java
public class ExampleStatic {

	private ExampleStatic(){}

	public static int e = 1;

	public static int plus(int a, int b, int c) {
		return a+b+c;
	}

}
```

```java
// 위의 클래스를 사용하고 싶은 경우 
public class ExampleUtil {
	public static void main(String[] args) {

		int value = ExampleStatic.e;
		int sum = ExampleStatic.plus(1, 2, 3);

		System.out.println("sum = " + sum);
	}
}

```
</div>
</details>
</details>

<details> <summary>final 은 무엇일까?</summary>
<div>
final 은 '마지막의' 또는 '변경될 수 없는'의  의미를 가지고 있고, 거의 모든 대상에 사용할 수 있다. 
변수에 붙이면 값을 변경할 수 없는 상수가 되고, 메서드에 사용하면 오버라이딩을 할 수 없으며, 클래스에 사용하면 자손클래스를 만들 수 없다.

* final 이 사용될 수 있는 곳 : 클래스, 메서드, 멤버변수, 지역변수
</div>
</details>
