# 아이템 39. 명명 패턴보다 애너테이션을 사용하라. 

## 명명 패턴
- 일정한 규칙 또는 형식에 따라 이름을 지정하는 방법이다.

## 명명 패턴의 단점 
1. 오타가 나면 안 된다. 
2. 올바른 프로그램 요소에서만 사용된다는 보증이 없다. 
3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.


## 애너테이션
- 마커 애너테이션 : 아무 매개변수 없이 단순히 대상에 마킹한다.

### 마커 애너테이션 타입 선언 
~~~java
/**
 * 테스트 메서드임을 선언하는 에너테이션
 * 매개변수 없는 정적 메서드 전용
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
~~~
- @Retention 과 @Target 은 애너테이션 선언에 다는 애너테이션으로 메타애너테이션이라 한다. 
- @Retention(RetentionPolicy.RUNTIME) 은 런타임에도 유지되어야 한다는 표시이다. 
- @Target(ElementType.METHOD) 메타애너테이션은 @Test 가 반드시 메서드 선언에서만 사용돼야 한다고 알려준다. 클래스 선언, 필드 선언 등 다른 프로그램 요소에는 달 수 없다.

### 마커 애너테이션을 사용한 프로그램 예 

~~~java
public class Sample {
    @Test
    public static void m1() { // 성공해야 한다.
    }

    public static void m2() {
    }

    @Test
    public static void m3() { // 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m4() {
    }

    @Test
    public void m5() { // 잘못 사용한 예: 정적 메서드가 아니다.
    }

    public static void m6() {
    }

    @Test
    public static void m7() { // 실패해야 한다.
        throw new RuntimeException("실패");
    }

    public static void m8() {
    }
}
~~~
- 1개 성공, 2개 실패, 1개 잘못 사용, 나머지 4개 메서드(@Test 없는)는 테스트 도구가 무시
- @Test 애너테이션이 Sample 클래스의 의미에 직접적인 영향을 주지 않고, 추가 정보를 제공한다. 

### 마커 애너테이션을 처리하는 프로그램 
~~~java
public class RunTests {
    public static void main(String[] args) throws ClassNotFoundException, InvocationTargetException, IllegalAccessException {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + "실패: " + exc);
                } catch (Exception exception) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
~~~
- 명령줄로부터 완전 정규화된 클래스 이름을 받아 @Test 애너테이션이 달린 메서드를 차례로 호출한다. 
- 애너테이션을 잘못 사용해 예외가 발생하면 오류 메세지를 출력한다. 


### 매개변수 하나를 받는 애너테이션 타입 
~~~java
/**
 * 명시한 예외를 던저야만 성공하는 테스트 메서드 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
~~~
- 이 애너테이션의 매개변수 타입은 Class<? extends Throwable> 로써 모든 예외(와 오류) 타입을 다 수용한다.

## 다수의 예외를 명시하는 애너테이션 : 배열 매개변수
- 예외를 여러개 명시하고 그중 하나가 발생하며 성공하게 만들 수 있다. 
- 매개변수 타입을 Class 객체의 배열로

### 배열 매개변수를 받는 애너테이션 타입
~~~java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}
~~~

### 배열 매개변수를 받는 애너테이션을 사용하는 코드 
~~~java
public class RunTests {
    @ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
    public static void doublyBad() { // 성공해야 한다.
        List<String> list = new ArrayList<>();
        
        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나 
        // NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
}
~~~
- 원소가 여러 개인 배열을 지정할 때는 원소들을 중괄호로 감싸고, 쉼표로 구분해주면 된다.

## 다수의 예외를 명시하는 애너테이션 : @Repeatable
- 자바 8에서는 여러 개의 값을 받는 애너테이션을 @Repeatable 이용하여 만들 수 있다. 
- 배열 매개변수 사용하는 대신 애너테이션에 @Repeatable 메타애너테이션 사용하여 코드 가독성을 높일 수 있다. 
- @Repeatable 를 단 애너테이션은 하나의 프로그램 요소에 여러번 달 수 있다.

> ### 주의 사항 
> 1. @Repeatable 을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고, @Repeatable 에 이 컨테이너 애너테이션의 Class 객체를 매개변수로 전달해야 한다. 
> 2. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의한다. 
> 3. 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention) 과 적용 대상(@Target)을 명시해야 한다. (그렇지 않으면 컴파일되지 않음)

### 반복 가능한 애너테이션 타입 
~~~java
// 반복 가능한 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}

// 반복 가능 애너테이션 다루기 
if (m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)) {
        tests++;
        try {
            m.invoke(null);
                System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
            } catch (Throwable wrappedExc) {
                Throwable exc = wrappedExc.getCause();
                int oldPassed = passed;
                ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class);
                for (ExceptionTest excTest : excTests) {
                    if (excTest.value().isInstance(exc)) {
                        passed++;
                        break;
                    }
                }
            if (passed == oldPassed)
                System.out.printf("테스트 %s 실패: %s %n", m, exc);  
        }
    }
}
~~~
### 반복가능 애너테이션 처리시 주의할 점 
- 반복 가능 애너테이션을 여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다.
  - getAnnotationsByType 메서드는 구분하지 않아서 반복 가능 애너테이션과 그 컨테이너 애너테이션을 모두 가져오지만, isAnnotationPresent 메서드는 둘을 명확히 구분한다. 
  - 반복 가능 애너테이션을 여러 번 단 다음 isAnnotationPresent로 반복 가능 애너테이션이 달렸는지 검사한다면 "그렇지 않다"고 알려준다(컨테이너가 달렸기 때문에).
  - 그 결과 애너테이션을 여러 번 단 메서드들을 모두 무시하고 지나친다. 
    - 같은 이유로, isAnnotationPresent로 컨테이너 애너테이션이 달렸는지 검사한다면 반복 가능 애너테이션을 한 번만 단 메서드를 무시하고 지나친다. 
    - 결론적으로, 애너테이션이 달려 있는 수와 상관없이 모두 검사하려면 둘을 따로따로 확인해야 한다.

## 결론 
애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유가 없다.  
자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다. (@Override, @SuppressWarnings)