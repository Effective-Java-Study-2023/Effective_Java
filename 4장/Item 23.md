# 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라. 

> 태그란 객체나 클래스의 "형태"나 "종류" 구분하기 위한 목적으로 사용되는 변수나 필드를 말한다.  


<details><summary>태그 달린 클래스의 예제 코드 </summary>
<div>

~~~java
    // 태그 달린 클래스
    public class Figure {
        enum Shape { RECTANGLE, CIRCLE };
		
		// 태그 필드 : 현재 모양을 나타낸다. Circle or Rectangle 
        final Shape shape;
    
        double length;
        double width;
    
        double radius;
    
        Figure(double radius) {
            shape = Shape.CIRCLE;
            this.radius = radius;
        }
    
        Figure(double length, double width) {
            shape = Shape.RECTANGLE;
            this.length = length;
            this.width = width;
        }
    
        double area() {
            switch (shape) {
                case RECTANGLE :
                    return length * width;
                case CIRCLE :
                    return Math.PI * (radius * radius);
                default:
                    throw new AssertionError(shape);
            }
        }
    }
~~~
</div>
</details>


## 태그 달린 클래스의 단점 
1. 쓸데없는 코드가 많다.
   - 가독성이 안 좋다. 
2. 필드들을 final 로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야 한다.
   - 필드를 잘못 초기화해도 컴파일시에 알 수 없다. 
3. 다른 의미를 추가하려면 코드를 수정해야 한다. (예. 삼각형 추가 -> switch 문 수정)
4. 인스턴스의 타입만으로는 현재 나타내는 의미를 알 수 없다. 

### 이것을 보완하기 위해... 

## 클래스 계층 구조를 활용하라 
1. 루트(root)가 될 추상 클래스 정의 
    - 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언
    - 모든 하위 클래스에서 공통으로 사용하는 데이터 필드 전부 루트 클래스에 작성
    - 태그 값에 관계없이 동작이 일정한 메서드들은 클래스 계층 구조에서는 루트 클래스의 일반 메서드로 추가 가능
2. 구체 클래스 정의
    - 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다. 
   
<details><summary>계층 구조로 변경</summary>
<div>

~~~java
    public abstract class NewFigure {
    
        abstract double area();
    
    }
~~~

~~~java
    public class Circle extends NewFigure {
        final double radius;
    
        Circle(double radius) {
            this.radius = radius;
        }
    
        @Override
        double area() {
            return Math.PI * (radius * radius);
        }
    }
~~~

~~~java
    public class Rectangle extends NewFigure{
    
        final double length;
        final double width;
        Rectangle(double length, double width) {
            this.length = length;
            this.width = width;
        }
    
        @Override
        double area() {
            return length * width;
        }
    }
~~~

</div></details>

## 계층 구조의 장점 
1. 각 클래스의 생성자가 모든 필드를 초기화하고, 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다.
2. 다른 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용할 수 있다.
3. 변수의 의미를 명시하거나 제한할 수 있고, 특정 의미만 매개변수로 받을 수 있다. 

## 결론
> 태그 필드가 필요하다면 태그를 없애고 계층구조로 리팩터링하는 것을 고려하자.
    
