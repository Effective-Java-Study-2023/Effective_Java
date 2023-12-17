# Item 83 - 지연 초기화는 신중히 사용하라

<hr>

## 지연초기화(lazy initialization)

<hr>

용도 및 효과
- 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법
- 주로 최적화 용도로 사용
- 클래스와 인스턴스 초기화 때 발생 하는 위험한 순환문제를 해결하는 효과도 있음

적용 대상
- 정적 필드
- 인스턴스 필드

## 지연초기화는 양날의 검이다.

<hr>

지연초기화를 사용한다면 인스턴스 생성 시의 초기화 비용은 줄지만 그 대신 필드에 접근하는 비용은 커진다.

즉 지연 초기화 필드중 초기화가 이뤄지는 비율에 따라, 실제 초기화에 드는 비용에따라, 초기화된 각 필드를 얼마나 빈번히 호출하느냐에 따라
지연 초기화가 실제로는 성능을 느려지게 할 수 있다.

따라서 지연초기화는
해당 클래스의 인스터스 중 그 필드를 사용하는 인스턴스의 비율이 낮고
그 필드를 초기화하는 비용이 크다면
지연초기화를 사용 하자.

하지만 이러한 경우를 알기 위한 방법은 지연초기화 적용 전후의 성능을 측정해보는 것 이 외에는 없다.

또한 멀티스레드 환경에서는 지연 초기화 하는 필드를 둘 이상의 스레드가 공유한다면
어떤 형태로든 반드시 동기화 해야한다.

## 초기화 방법들 

<hr>

### 일반적 초기화 예제

```Java
class Example {
    // final 한정자를 통한 인스턴스 필드 생성
    private final FieldType field = computeFieldValue();
}
```

대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.

### 인스턴스 필드의 지연초기화

지연 초기화가 초기화 순환성(initialization circularity) 을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용한다.
> 초기화 순환성이란? <br>
> 예를들어
> A클래스의 생성자가 B의 인스턴스를 생성하고 -> B클래스의 생성자가 C의 인스턴스를 생성하고 -> C클래스의 생성자가 A의 인스턴스를 생성하는 경우를 말한다.
```Java
class Example {
    private final FieldType field;
    
    private synchronized FieldType getField() {
        if (field == null) 
            field = computeFieldValue();
        return field;
    }
}
```

두 초기화 방식은 정적 필드에서도 똑같이 적용된다.

이 떄 필드와 접근자 메서드 선언에 static 한정자를 추가해야 한다.

### 정적 필드의 지연 초기화

성능면에서 정적 필드의 지연 초기화가 필요한 경우 지연 초기화 홀더 클래스 관용구(lazy initialization holder class) 를 사용한다.

클래스는 클래스가 처음 쓰일 때 비로소 초기화된다는 특성을 이용한 관용구이다.

```Java
class Example {
    private static class FieldHolder {
        static final FieldType field = computeFieldValue();
    }

    private static FieldType getField() { return FieldHolder.field; }
}
```

- getField가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서, 비로소 FieldHolder 클래스 초기화를 촉발한다.
- getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질 이유가 전혀 없다.

> 일반적인 VM은 오직 클래스를 초기화할 때만 필드 접근을 동기화할 것이다.
> 클래스 초기화가 끝난 후에는 VM이 동기화 코드를 제거하여, 그 다음부터는 아무런 검사나 동기화 없이 필드에 접근하게 된다.

## 성능적인 측면에서의 인스턴스 필드 지연 초기화를 위한 이중검사 관용구(double-check)

<hr>
필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile 로 선언해야 한다.

> Volatile 키워드를 사용하면 해당 변수를 메인 메로리에 저장하고 읽기 때문에 항상 최근에 기록된 값을 읽을 수 있다. 
> [Item 78](https://obtainable-poppyseed-72e.notion.site/item-78-e5c999b8ceea4ff0b9d3fc288eacb95b)
```Java
class Example {
    private volatile FieldType field;
    
    private FieldType getField() {
        FieldType result = field; // 초기화 시 한 번만 읽도록 하기 위함
        if(result != null) return result;
        
        synchronized (this) {
            if(field == null)  // 두 번째 검사 (락 사용)
                field = computeFieldValue();
            return field;
        }
    }
}
```

반드시 필요하지는 않지만 성능을 높여주고, 저수준 동시성 프로그래밍에 표준적으로 적용되는 더 우아한 방법이다.

정적 필드에 대한 이중검사 관용구 적용?
- 정적 필드를 지연 초기화하기 위해서는 이중검사보다 지연 초기화 홀더 클래스 방식이 더 낫다.

## 이중 검사의 특이 유형 2가지

<hr>

반복해서 초기화해도 상관없는 인스턴스 필드를 지연초기화해야 하는 경우, 이중검사에서 두 번째 검사를 생략할 수 있다.

### 단일 검사(single-check, 이중 검사에서 두 번째 검사 생략) 관용구

- 필드는 volatile로 선언
- 초기화가 중복해서 일어날 수 있다.

```Java
class Example {
    private volatile FieldType field;
    private FieldType getField() {
        FieldType result = field;
        if(result == null) 
            field = result = computeFieldValue();
        return result;
    }
}
```

### 짜릿한 단일 검사(racy single-check) 관용구

사용 조건

- 모든 스레드가 필드의 값을 다시 계산해도 되고, 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일검사의 필드 선언에서 volatile 한정자를 없애도 된다.

효과

- 어떤 환경에서는 필드 접근 속도를 높여주지만, 초기화가 스레드당 최대 한 번 더 이루어질 수 있다.

아주 이례적인 기법으로, 보통은 쓰이지 않는다.

## 정리

<hr>

|                **종류**                |                       **적용 관용구**                        |
|:------------------------------------:|:-------------------------------------------------------:|
|             **인스턴스 필드**              |              이중검사 관용구 <br> (double-check)               |
|              **정적 필드**               |지연 초기화 홀더 클래스 관용구 <br> (lazy initialization holder class)|
| 반복적인 초기화가 <br> 상관없는 <br> **인스턴스 필드** |              단일 검사 관용구 <br> (single-check)              |

- 여기서 다룬 모든 초기화 기법은 기본 타입 필드와 객체 참조 필드에 모두 적용할 수 있다.
- 대부분의 필드는 지연시키지말고 곧바로 초기화해야 한다.
- 성능 때문에 혹은 위험한 초기화 순환을 막기 위해 꼭 지연 초기화를 써야 하는 경우 올바른 지연 초기화 기법을 사용한다.
- 인스턴스 필드에는 이중검사 관용구, 정적 필드에는 지연 초기화 홀더 클래스 관용구를 사용한다.
- 반복해 초기화해도 괜찮은 인스턴스 필드에는 단일검사 관용구도 고려대상이다.