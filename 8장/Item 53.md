# 아이템 53. 가변인수는 신중히 사용하라
## 가변인수 메서드
~~~Java
static int sum(int... args) {
    int sum = 0;
    for (int arg : args)
        sum += arg;
    return sum;
}
~~~
- 가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.
- 가변인수 메서드를 호출하, 가장 먼저 인수의 개수가 같은 길이의 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다.

아래 코드는 문제점이 존재한다.
~~~Java
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
~~~
- 인수를 0개만 넣어 호출하면 런타임에 실패한다.
- args 유효성 검사를 명시적으로 해야한다.

해결책
~~~Java
static int min(int firstArgs, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
~~~
- 첫 번째로는 평범한 매개변수를 받고, 가변인수는 두 번째로 받으면 앞의 문제를 해결할 수 있다.
### 성능에 민감한 상황에 사용할 수 있는 패턴
~~~Java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int… rest) { }
~~~
- 가변 인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다. 따라서 성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다
- 다중정의를 활용하여 최적화할 수 있다. 호출 시 인수가 0개 ~ 3개인 경우엔 배열을 할당-초기화하는 가정이 없기 때문이다.
- EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다.

## 정리
- 인수 개수가 일정하지 않은 메서드를 정의해야 하는 경우엔 가변인수를 사용한다.
- 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 둔다.
- 가변인수를 사용할 때는 성능 문제까지 고려.
