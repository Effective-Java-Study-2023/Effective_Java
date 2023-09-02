# 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 흔하게 실수하는 클래스
이따금 인스턴스 필드들을 모아놓는 일 외에는 아무 목적도 없는 퇴보한 클래스를 작성하려 할 때가 있다.

```Java
class Point {

    public double x;
    public double y;

}
```
- API의 필드를 수정하지 않고는 내부 표현을 바꿀 수 없다. (외부 코드가 사용중이라면 영향을 끼침)
  - getter/setter 메서드가 존재한다면 얼마든지 표현 변경이 가능하다.
- 불변식을 보장할 수 없다.
  - 클라이언트가 직접 데이터를 변경할 수 있다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.
  - 1차원적인 접근만 가능하고, 추가 로직을 삽입할 수 없다.

해당 패턴을 가진 클래스의 문제점을 대표적으로 3가지 짚어주고 있다. 모두 캡슐화의 이점을 포기한 내용들이다. (정보은닉 , 모듈화 , 독립성)

## 대표적인 해결방법: 클래스 캡슐화
철저한 객체지향 프로그래머는 필드를 모두 private 로 바꾸고 접근자(getter) 메서드를 추가한다.

```Java
class Point {

    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }

}
```
- getter/setter 메서드를 통해 언제든지 내부 표현을 바꿀 수 있다.
- 불변식이 보장된다. (Setter 메서드가 없다면)
  - 클라이언트는 메서드를 통해서만 필드에 접근 가능하다.
- 외부에서 필드에 접근할 때 부수 작업을 수행 시킬 수 있다.
  - getter/setter 메서드에 얼마든지 추가 가능하다.

## 또 다른 해결방법: private 중첩 클래스
하지만 package-private 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.

```Java
public class TopPoint {

    private static class Point {
        public double x;
        public double y;
    }

    public Point getPoint() {
        Point point = new Point();
        point.x = 3.5;              
        point.y = 4.5;              
        return point;
    }
}
```
위와 같이 private 클래스를 중첩시키면 TopPoint 클래스에서는 얼마든지 Point 클래스의 필드를 조작할 수 있지만, 외부 클래스에서는 Point 클래스의 필드에 직접 접근 할 수 없다.

package-private 클래스 역시 해당 클래스가 포함되는 패키지 내에서만 조작이 가능하고 패키지 외부에서는 접근이 불가능 하므로, 처음 제시된 3가지 문제점을 모두 매꿀 수 있다.

클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다. 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다.

즉, 클래스를 통해 표현하려는 추상 개념만 상세하고 올바르게 표현하면 클래스와 필드를 선언하는 입장에서나 클라이언트 입장에서나 훨씬 깔끔한 코드가 작성 가능하다는 것이다.



## public 클래스의 필드를 직접 노출시킨 사례
자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 존재한다.
대표적으로 java.awt.package 패키지의 Point와 Dimension 클래스가 있다.

### java.awt.Component 클래스 내부

getSize() 메서드를 호출 할 때마다 Dimension 인스턴스를 생성하고 있다.
Dimension 클래스란 그래픽 사용자 인터페이스(GUI) 컴포넌트의 크기를 나타내는 데 사용된다고 한다...
```Java
    public Dimension getSize() {
        return size();
    }

    @Deprecated
    public Dimension size() {
        return new Dimension(width, height);
    }
```
```Java
public class Dimension extends ... {

    public int width;
    public int height;
}
```
Dimesion 클래스의 필드는 가변으로 설계되어 getSize를 호출하는 모든 곳에서 방어적 복사를 위해 인스턴스를 새로 생성해야만 한다. 단순히 작은 객체로 몇 개 생성하는 것은 요즘 VM, HW 사양으로 큰 문제가 없겠지만 수 만, 수 억개가 된다면 이야기가 달라진다.

### final 키워드를 추가한 예시
```Java
public final class Time {

  public final int hour;
  public final int minute;
}
```
공개된 필드 public에 final 키워드를 추가하여 불변을 보장하는 것은 당장 불변식은 보장할 수 있지만 API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 2가지 문제점을 여전히 매꿀 수 없다. 
(게다가 해당 필드가 배열이라면 이야기가 다르다. 배열 원소의 불변식을 보장하지 못한다.)

