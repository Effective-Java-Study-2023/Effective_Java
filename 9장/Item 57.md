# Item 57 - 지역변수의 범위를 최소화하라

<hr>

## 지역변수의 범위를 최소화해야 하는 이유

<hr>

코드의 가독성과 유지 보수성 때문.

지역변수의 범위를 사용하는 지점으로 한정한다면 불필요한 오류의 발생 가능성을 감소시켜줄 것이다.

## 가장 처음 쓰일 때 선언하기

<hr>

자바에서는 문장을 선언할 수 있는 곳이면 어디서든 변수를 선언할 수도 있다.

사용하는 시점과 동떨어진 곳에서 지역변수를 선언하게 되면 가독성이 떨어질 수 있을 뿐더러
변수를 실제 사용하는 시점엔 타입과 초깃값이 기억나지 않을 수도 있다.

지역변수의 범위를 선언된 지점부터 해당 블록이 끝날 때까지로 한정 짓는다면
사용하지 않아야 할 지점에 사용하게 되는 이슈는 없을 것이다.

핵심은 지역변수의 범위를 줄이는 가장 좋은 방법이 가장 처음 쓰일 때 선언하기라는 것이다.

## 거의 모든 지역변수는 선언과 동시에 초기화해야 한다.

<hr>

초기화에 필요한 정보가 충분하지 않다면 충분해질 때까지 선언을 미뤄둬야 한다.

특정 상황(try-catch)을 제외하고는 선언과 동시에 초기화 해야한다.

```Java
// try 구문의 변수가 try블록 외부에서 사용될 수 있기 때문에 이 부분은 예외적임
Class<? extends Set<String>> cl = null;
try {
    cl = (Class<? extends Set<String>>) Class.forName(args[0]);
} catch(ClassNotFoundException e) {
    //...
}
cl = ~~~;
```

## for문을 사용하라

<hr>

for문은 독특한 방식으로 변수 범위를 최소화해준다. for문이든 for-each문이든 반복문에서는 반복 변수의 범위가 반복문의 몸체, 
그리고 for 키워드와 몸체 사이의 괄호로 제한된다. 

따라서 이번 아이템의 주제에 따라서 반복문이 종료된 이후에도 변수의 값을 써야하는 경우가 아니라면 while문보다 for문을 사용하는 것이 좋다.

```Java
// 컬렉션이나 배열을 순회하는 권장 관용구
for (Element e : c) {
    //...
}
```

반복자를 사용해야 하는 상황이라면 for-each문보단 for문을 사용하는 것이 낫다.

또, 아래와 같은 이유때문에 while 보다는 for문을 사용하는 것을 추천한다.

```Java
Iterator<Element> i = c.iterator();
while(i.hasNext()) {
    doSomething(i.next);    
}

Iterator<Element> i2 = c2.iterator();
while(i.hasNext()) {
    doSomething(i2.next);    
}
```
i가 첫번째 반복문에서 쓰임이 끝나고 나서도 사용할 수 있는 상태로 남아있기 때문에
그 다음 while문에서도 i를 쓰는 실수를 하고 있다.

그러나 위 코드는 컴파일시 문제도 되지 않을 뿐더러 실행 시에 예외도 던지지 않는다.

두 번째 반복문은 c2를 순회하지 않고 곧장 끝나 c2가 비어있다고 착각하게 만든다.

이러한 실수를 방지할 수 있도록 for문을 사용할 것을 추천한다.

```Java
for (Iterator<Element> i = c.iterator(); i.hasNext;) {
    Element e = i.next();
    //...
}

for (Iterator<Element> i = c2.iterator(); i.hasNext;) {
    Element e = i.next();
    //...
}
```
위처럼 for문을 사용하면 while문을 사용했을 때의 발생할 수 있는 문제를 미연에 방지할 수 있다.

반복문이 사용한 원소와 반복자의 유효 범위가 반복문 종료와 함께 끝나기 때문이다.

또, 위처럼 변수의 유효범위가 for문의 범위와 일치하기 때문에 똑같은 이름의 변수를 여러 반복문에서 써도 서로 아무런 영향을 주지 않는다.

## 메서드를 작게 유지하고 한 가지 기능에 집중하라

<hr>

만약 한 메서드에서 여러가지 일을 수행한다면, 메서드의 길이도 늘어지고 여러 지역변수가 사용될 가능성이 높아진다.
이렇게 되면 명확한 변수 이름을 짓지 않는 한 문제가 발생할 수 있다.

간단하게 메서드를 기능별로 쪼개는 것으로 이 문제를 해결할 수 있다.

## 정리

<hr>

이번 아이템은 클린코드와 관련한 아이템이라 크게 소개할 내용은 없지만 while보다 for문을 사용해야 하는 이유, 
메서드를 작게 유지해야 하는 이유 등의 주장에 힘을 실어줄 수 있는 아이템이라고 생각하면 좋을 것 같다.
