# 아이템 63. 문자열 연결은 느리니 주의하라.

> 문자열 연결 연산자 (+)는 한 줄짜리 출력값 혹은 작고 크기가 고정된 객체의 문자열 표현할 때만 사용하자.
> 문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2 에 비례한다.

문자열은 불변이라 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사하기 때문에 성능 저하는 피할 수 없다.
성능을 포기하지 않으러면 StringBuilder 를 사용하자.

### 예시 코드
~~~ java
// 문자열 연결 연산자를 사용 : 느리다. 
String test = "test";
for (int i = 0; i < 100000; i++) {
    test += "a";
}
~~~

~~~ java
// StringBuilder 를 사용하여 성능 개선
StringBuilder b = new StringBuilder();
b.setLength(0);
for (int i = 0; i < 100000; i++) {
    b.append("a");
}
~~~

## 결론
성능에 신경 써야할 경우, 문자열 연결 연산자(+)를 피하는 대신, StringBuilder 의 append 메서드를 사용하자. 
