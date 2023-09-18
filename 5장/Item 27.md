# 아이템 27. 비검사 경고를 제거하라

### 비검사 경고가 나타나는 경우
~~~Java
Set<Lark> exaltation = new HashSet();
~~~
위는 비검사 경고가 발생하는 코드의 예이다.
위의 경우엔 앞에서 나왔던 로 타입을 사용해서 발생했다.
~~~Java
Venery.java:4: warning: [unchecked] unchecked conversion
Set<Lark> exaltation = new HashSet();
required: Set<Lark>
found: HashSet
~~~
자바 7부터는 <> 다이아몬드 연산자를 뒤에 적어주면 자동으로 타입 추론을 해준다.
할 수 있는 한 위와 같은 비검사 경고를 모두 제거해주어야 한다.
그래야 타입 안정성이 보장되고, 런타임에 ClassCastException이 뜰 확률이 줄어든다.
### 비검사 경고를 제거할 수 없는 경우
혹시 경고를 제거할 순 없지만 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기면 된다.
단 반드시 확신할 수 있는 경우에만 해야 한다.
- 가능한 좁은 범위에 해야 한다.
    - 변수 선언, 아주 짧은 메서드, 생성자 정도에 적용하는 것이 좋으며, 이 중에서도 가장 작은 범위가 좋다.
    - 의도치 않게 다른 비검사 경고가 숨겨지면 큰 문제가 된다.
### 비검사 경고를 제거할 수 없는 경우 - 코드 예제
~~~Java
public <T> T[] toArray(T[] a) {
if (a.length < size) {
return (T[]) Arrays.copyOf(elements, size, a.getClass());
}

System.arraycopy(elements, 0, a, 0, size);

if(a.length > size) {
a[size] = null;
}

return a;
}
~~~
~~~Java
ArrayList.java:305: warning: [unchecked] unchecked cast
return (T[]) Arrays.copyOf(elements, size, a.getClass());

required: T[]
found: Object[]
~~~
### 비검사 경고를 제거할 수 없는 경우 - @SuppressWarnings를 사용하여 코드 개선
~~~Java
public <T> T[] toArray(T[] a) {
if (a.length < size) {
// 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
@SuppressWarnings("unchecked")
T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
return result;
}

System.arraycopy(elements, 0, a, 0, size);

if(a.length > size) {
a[size] = null;
}

return a;
}
~~~
가능한 작은 범위인 변수 선언부에 @SuppressWarnings을 추가하여 적용되는 범위를 최대한 좁혔다.
경고를 숨기려면 경고를 숨기는 이유를 명확히 주석으로 작성하자.
### 정리
- 모든 비검사 경고는 잠재적으로 애플리케이션에 가장 위험한 런타임에러(ClassCastException)를 띄울 수 있으므로 최선을 다해 제거해야 한다.
- 경고를 없앨 방법을 찾지 못하겠다면, 그 코드가 타입 안전함을 증명하고 가능한 범위를 좁혀 @SuppressWarnings("unchecked") 애노테이션을 이용해 경고를 숨기자.
  - 그 다음 경고를 숨기기로 한 근거를 주석으로 남기자.