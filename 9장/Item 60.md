# 아이템 60. 정확한 답이 필요하다면 float 와 double 은 피하라.

> float 와 double 타입은 이진 부동소수점 연산에 쓰이고, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 설계되었다.
> 따라서, 정확한 결과가 필요할 때는 사용하면 안 된다. 금융 관련 계산에 취약하다.

### 부동 소수점 방식
- 부동 소수점(floating point) 방식은 소수점(point)이 둥둥 떠다닌다(floating) 라는 의미이다.
- 표현할 수 있는 값의 범위를 최대한 넓혀 오차를 줄이고자 한다.
- 실수를 1.XXX * 2^n 이고, 부호부(1bit), 지수부(8bit), 가수부(23bit)로 나눈다.

### 부동 소수점 계산 방법
~~~java
±(1.가수부) × 2^지수부-127
~~~

-118.625 f 를 부동소수점으로 변환해보자.
- 음수이므로 부호부 1
- 절대값 118.625 이진법 변환
    - ~~~java
    // 정수부
    118 = 1110110(2)
    // 소수부
    0.625
    = 0.625 x 2 = 1.250 → 정수부 1
    = 0.250 x 2 = 0.500 → 정수부 0
    = 0.500 x 2 = 1.000 → 정수부 1
    = 101(2)
    // 결과
    1110110.101(2)
    ~~~
- 소수점을 이동시켜 정수부가 한자리가 되도록 변환한다.
  ~~~Java
  1110110.101(2) => 1.110110101 * 2^6
  ~~~
- 가수부에 11011010100000000000000
- 지수부에 6 + 127(바이어스값)을 더하고 2진수로 변경한 값 10000101
- 따라서 11000010111011010100000000000000(2)

### float, double 의 소수 계산 오차가 발생하는 이유
~~~java
System.out.println(1.03-0.42);
// 결과 : 0.6100000000000001

System.out.println(1.1 == 1.1f);
// 결과 : false
~~~
- 무한 소수
    - 0.1을 2진수로 변환할 경우 나누어 떨어지지 않고 0.000110011... 로 무한히 반복된다.
    - 따라서 끝자리가 5로 끝나지 않는다면, 무한 소수가 된다.

~~~java
double funds = 1.00;
int itemsBought = 0;
for (double price = 0; funds >= price; price += 0.10) {
    funds -= price;
    itemsBought++;
}
System.out.println(itemsBought + "개 구입"); // 4
System.out.println("잔돈(달러) : " + funds);  // 0.3999999...
~~~
## 금융 계산에는 BigDecimal, int or long 을 사용
- 소수의 크기가 9자리를 넘지 않으면 int 타입
- 18자리를 넘지 않으면 long 타입
- 18자리 초과 BigDecimal 클래스

### BigDecimal 의 단점
- 객체를 생성하고, 메서드를 호출해야 하므로, 기본 타입보다 사용하기 불편하다.
- 훨씬 느리다.

~~~java
final BigDecimal TEN_CENTS = new BigDecimal(".10");
int itemsBought = 0;
BigDecimal funds = new BigDecimal("1.00");
for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
    funds = funds.subtract(price);
    itemsBought++;
}
System.out.println(itemsBought + "개 구입"); // 4
System.out.println("잔돈(달러) : " + funds); // 0.00
~~~
### int, long 타입을 쓸 경우 단점
- 다룰 수 있는 값의 크기가 제한된다.
- 소수점을 직접 관리해야 한다. (정수 부분과 소수 부분을 따로 처리해야 한다.)
~~~java
double a = 1000.0;
double b = 999.9;
System.out.println(a - b); // 0.10000000000002274


// 각 숫자에 10을 곱해서 소수부를 없애주고 정수로 형변환
long a2 = (int)(a * 10);
long b2 = (int)(b * 10);
double result = (a2 - b2) / 10.0; // 정수끼리 연산, 다시 10.0을 나누기 하여 실수로 변환하여 저장
System.out.println(result); // 0.1
~~~
~~~java
int itemsBought = 0;
int funds = 100;
for (int price = 10; funds >= price; price += 10) {
    funds -= price;
    itemsBought++;
}
System.out.println(itemsBought + "개 구입");  // 4
System.out.println("잔돈(달러) : " + funds);  //0
~~~


## 결론
- 정확한 답이 필요한 경우 float 나 double 을 피해라.
- 성능이 중요하고, 소수점을 직접 추적할 수 있으며, 숫자가 너무 크지 않다면 int 나 long 을 사용해라.
- 코딩 시의 불편함이나 성능 저하를 신경 쓰지 않는다면, BigDecimal 을 사용해라.
- 소수의 크기가 9자리를 넘지 않으면 int 타입, 18자리를 넘지 않으면 long 타입, 18자리 초과 BigDecimal 클래스

<details><summary>실수의 2진수 표현 방식</summary>
<div>

- 11.765625 를 2진수로 변환하는 경우
- 정수부 11 과 소수부 0.765625 를 각각 2진수로 변환한다.
- 정수부 : 11 -> 1011(2)
- 소수부 : 0.765625 -> 0.110001(2)
    - 절대값이 1보다 작은 10진수 소수에 2를 곱한다.
    - 2를 곱한 결과가 1을 넘을 경우 1을 떼어내고, 아니면 0으로 처리해 계산을 이어나간다.
    - 소수점 이하에 아무것도 남지 않을 때까지 반복한다.
    - 이 과정에서 만들어진 0 또는 1들을 순서대로 쓰면 된다.

</div></details>