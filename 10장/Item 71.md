# 아이템 71. 필요 없는 검사 예외 사용은 피하라 
> JAVA 의 예외 처리는 모두 Throwable 을 상속하고 있고, 크게 Exception 과 Error 로 나뉜다. 
> Exception 은 검사 예외이고, Error 는 비검사 예외이다. 


### 검사 예외를 제대로 사용한다면
- 검사 예외를 제대로 활용한다면 API 와 프로그램의 질을 높일 수 있다. 
- 발생한 문제를 프로그래머가 처리해 안전성을 높이게 해준다.

### 검사 예외를 과하게 사용한다면 
- 검사 예외를 던질 수 있다고 선언 되면, 이를 호출하는 코드에서 catch 블록을 두어 그 예외를 붙잡아 처리하거나 바깥으로 던져 문제를 전파해야 한다. 
- 검사 예외를 던지는 메서드는 스트림 안에서 직접 사용할 수 없어 자바 8부터 부담이 커졌다. 

### 검사 예외 코드 
~~~java
// throw 이용
} catch (TheCheckedException e) {
    throw new AssertionError();
}
~~~

~~~java
// 로직으로 처리
} catch (TheCheckedException e) {
    e.printStackTrace();
    System.exit(1);
}
~~~

- 검사 예외라 catch 문으로 처리하는데 throw 나 시스템 종료라는 로직이 비검사 예외가 큰 차이가 없다. 
- 별 다른 방법이 없으면 비검사 예외를 선택해야 한다. 
- 검사 예외는 하나의 예외라도 try-catch 를 사용해야하므로 불편하다는 단점이 있다. 

### 검사 예외 회피 방법 
- 적절한 결과 타입을 담은 옵셔널을 반환한다. 
  - 검사 예외를 던지는 대신 단순히 빈 옵셔널을 반환하면 된다. 
  - 단점 : 예외가 발생한 이유를 알려주는 부가 정보를 담을 수 없다. 
~~~java
try {
    // 예외가 발생하지 않는 경우 결과를 옵셔널로 감싸서 반환
    String result = someOperationThatMayThrow();
    return Optional.of(result);
} catch (Exception e) {
    // 예외가 발생하면 빈 옵셔널 반환
    return Optional.empty();
}
~~~
- 기존 메서드를 2개로 쪼개 비검사 예외로 바꾸기 
  - 리팩터링 후의 API 가 더 깔끔하지는 않지만, 더 유연하다. 
  - 단점 : 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나, 
  - 외부 요인에 의해 상태가 변할 수 있다며 적절하지 않은 방식이다. 
~~~java
try {
	obj.action(args);
} catch (TheCheckedException e) {
	... // 예외 처리
}
~~~
~~~java
if(obj.actionPermitted(args)){
	obj.action(args);
} else {
	... // 예외 처리
}
~~~

## 결론
API 호출자가 예외 상황에서 복구할 방법이 없는가?
 - YES(복구불가) : 비검사 예외를 던지자 
 - NO(복구가능)
   - 예외를 호출자가 처리해주어야 하는가?
     - YES : OPTIONAL 반환 고려 
       - BUT, 예외 이유에 대한 정보를 전달하고 싶은 경우 검사 예외를 던져라.
     - NO(예외를 API 에서 던진다) : 검사 예외를 던져라.

---
![image](https://github.com/silversheep26/Effective_Java/assets/122955367/c77a6e74-fccd-4851-841b-2e4d50f269b3)
### [출처](https://velog.io/@chullll/JAVA-%EA%B2%80%EC%82%AC%EC%98%88%EC%99%B8%EC%99%80-%EB%B9%84%EA%B2%80%EC%82%AC%EC%98%88%EC%99%B8)

### 검사 예외 
개발자가 명시해야 하는 부분은 검사 예외인 Exception 으로 어플리케이션 수행 중 일어날 법한 예외를 검사하고 대비하는 목적으로 사용한다. 
- 컴파일 단계에서 체크하며, 처리하지 않으면 컴파일 에러가 발생한다.
- 과도한 예외 검출은 시스템의 성능을 저하시킬 수 있다. 

### 비검사 예외 
Error 는 시스템적인 예외를 말한다. 개발자가 예외를 try-catch 로 잡지 않았을 때 발생한다. 
- 런타임에 발생하며, 코드 상의 문제이다. 
- NullPointerException 등이 있다.

