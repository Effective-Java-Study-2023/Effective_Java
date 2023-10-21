# 아이템 43. 람다보다는 메서드 참조를 사용하라
람다가 익명 클래스보다 나은 점 중에서 가장 큰 특징은 간결함이다.
더 간결한 방법은 메서드 참조이다.
### 예제코드
~~~Java
class Example {
    public static void main(String[] args) {
        // 람다 사용
        map.merge(key, 1, (count, incr) -> count + incr);

        // 메서드 레퍼런스 사용
        map.merge(key, 1, Integer::sum);
    }
}
~~~
* merge() : 키, 값, 함수를 인자로 받으며 주어진 키가 맵 안에 아직 없다면 주어진 {키,값} 쌍을 그대로 저장하고, 키가 이미 있다면 {키, 인자로 받은 함수의 결과} 쌍을 저장

람다는 두 인수의 합을 단순히 반환할 뿐이다. 자바 8이 되면서 Integer 클래스는 이 람다와 기능이 같은 정적 메서드 sum을 제공하기 시작했다.
~~~Java
map.merge(key, 1, Integer::sum);
~~~
매개변수 수가 늘어날 수록 메서드 참조로 제거할 수 있는 코드 양도 늘어난다.

만약 매개변수 이름이 드러나는게 가독성이 더 좋다면 람다를 써도 좋다.
또는 람다가 메서드 참조보다 간결할 때가 있는데, 메서드와 람다가 같은 클래스에 있을 때 그렇다.
~~~Java
// 만약 GoshThisClassNameIsHumongous 클래스에 execute메서드와 action 람다가 같이 있을경우
service.execute(GoshThisClassNameIsHumongous::action); // 메서드 참조
service.execute(() -> action()); // 람다
~~~
기본적으로 람다로 할 수 없는거는 메서드 참조로도 할 수 없다.
## 정리
메서드 참조는 람다의 간단명료한 대안이 될 수 있다.
메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고 , 그렇지 않을때만 람다를 사용

