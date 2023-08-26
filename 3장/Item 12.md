# 아이템 12. toString 을 항상 재정의하라

>toString 의 규약에 따르면 '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야 하며, "모든 하위 클래스에서 이 메서드를 재정의하라"고 한다.

### toString 이 자동 호출 되는 경우

---
1. 객체를 println, printf, 문자열 연결 연산자(+), assert구문에 넘길 때
2. 디버거가 객체를 출력할 때
3. 우리가 만든 객체를 참조하는 컴포넌트가 오류 메시지를 로깅할 때


### 어떻게 재정의하는 것이 좋은가?

---
- 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 담는다.
- 객체가 가진 주요 정보를 모두 반환하는 것이 좋다.
    - 하지만 객체가 너무 거대하거나, 문자열로 표현하기 적합하지 않은 경우는 요약 정보 추천

### toString 을 구현할 때 반환값의 포맷을 문서화할지 정해야 한다

---
- 값 클래스라면 문서화를 권장
    - 포맷을 명시하면 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.
    - 포맷을 명시한다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공하면 좋다.
- 포맷을 명시하면 그 포맷에 얽메이게 되어 수정이 어렵다.
- 포맷 명시 여부와는 상관 없이 의도를 명확히 밝혀야 하고, toString 이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공해야 한다.

~~~java
// 출처: 이펙티브자바 p.75코드
/**
* 이 전화번호의 문자열 표현을 반환한다.
* 이 문자열은 "XXX-YYY-ZZZZ"형태의 12자로 구성된다.
* XXX는 지역코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
* 각각의 대문자는 10진수 숫자 하나를 나타낸다.
*
* 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
* 앞에서부터 0으로 채워나간다.
* ex) 가입자 번호가 123이면 전화번호의 마지막 네 문자는 0123
*/
@Override
public String toString() {
  return String.format("%03d-%03d-%04d", ..., ..., ...);
}
~~~

~~~java

public class Coordinate {
    private int x;
    private int y;

    public Coordinate(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public String toString() {
        return "(" + x + ", " + y + ")";
    }

    // x와 y 값을 가져오는 getter 메서드
    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    public static void main(String[] args) {
        Coordinate coordinate = new Coordinate(3, 4);
        System.out.println(coordinate);  // "(3, 4)"

        // toString 결과로부터 직접 정보를 추출하는 대신, getter 메서드를 사용
        int xValue = coordinate.getX();
        int yValue = coordinate.getY();
        System.out.println("X: " + xValue + ", Y: " + yValue);
    }
}

~~~


### toString 을 재정의하지 않아도 되는 경우

---
- 정적 유틸리티 클래스
- 열거 타입

## 결론

---
> 상위 클래스에서 잘 재정의한 경우를 제외하고, 모든 구체 클래스에서 Object 의 toString 을 재정의하자.
> 해당 객체에 관해 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.