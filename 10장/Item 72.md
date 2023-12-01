# 아이템 72.표준 예외를 사용하라.

표준 예외를 재사용하라. 
- API 가 다른 사람이 익히고 사용하기 쉬워진다. 
- 읽기 쉽게 된다. 
- 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다.

### 자주 재사용되는 예외들 
- IllegalArgumentException
    호출자가 인수로 부적절한 값을 넘길 때 던지는 예외
~~~java
public class Item72 {
    List<Integer> list = new ArrayList<>();
    
    public void positiveAdd(Integer number) {
        if (number < 0) {
            throw new IllegalArgumentException("음수값은 허용하지 않음");
        }
        list.add(number);
    }
    
    public static void main(String[] args) {
        Item72 i = new Item72();
        i.positiveAdd(-1);
    }
}
~~~
- IllegalStateException
    대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 
~~~java
public class Item72 {
    List<Integer> list = new ArrayList<>();

    public void evenIndexAdd(Integer number){
        if(list.size()%2 != 0){
            throw new IllegalStateException("짝수 인덱스에 들어갈 수 없음");
        }
        list.add(number);
    }

    public static void main(String[] args) {
        Item72 i = new Item72();
        i.evenIndexAdd(0);
        i.evenIndexAdd(1);
    }
}
~~~
- NullPointerException 
    NULL 값을 허용하지 않는 메서드에 NULL 을 건네는 경우
- IndexOutOfBoundsException
    인덱스가 범위를 넘어 섰을 때
- ConcurrentModificationException
    단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때 
- UnsupportedOperationException
    클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 

### 예외 재사용 시 주의 사항 
- Exception, RuntimeException, Throwable, Error 는 직접 재사용하지 말자.
  - 추상 클래스라고 생각해야 한다. 
  - 다른 예외들의 상위 클래스이므로, 안정적으로 테스트할 수 없고, 정확한 예외를 넘기기 어렵다. 
- 예외의 이름 뿐만 아니라 예외가 던져지는 맥락이 부합할 때 예외를 재사용 하자. 
  - 상황에 부합한다면 항상 표준 예외를 재사용하는 거싱 좋다. 
  - 예를 들어, 숫자를 입력받아 해당 숫자만큼 카드를 나누어주는 메서드에서 인자로 전달된 숫자가 너무 큰 경우(IllegalArgumentException)와
  - 카드 수가 너무 적어 처리할 수 없는 경우(IllegalStateException) 어떤 것을 예외 처리할지 생각해보아야 한다. 
- 예외는 직렬화할 수 있다. 
  - 예외에서 많은 정보를 제공하길 원한다면 표준 예외를 확장해도 되지만, 예외는 직렬화할 수 있으니 커스텀한 예외를 사용해야할 이유가 사라진다. 
  - 직렬화 
    - 객체를 데이터 스트림으로 변환하는 과정 
    - 객체를 저장하거나 전송할 때 유용하다. 
    - 직렬화된 데이터에는 객체의 타입 정보와 같은 클래스 메타 정보가 포함되어 데이터의 크기가 커진다.  
    - 직렬화된 데이터를 역직렬화할 때 해당 클래스의 모든 타입의 객체를 생성할 수 있기 때문에 보안에 취약할 수 있다. 
## 결론 
- 상황에 부합한다면 항상 표준 예외를 사용하자. 
- 예외 재사용 시 Exception, RuntimeException, Throwable, Error 는 직접 재사용하지 말자.
- 웬만하면 커스텀한 예외의 사용을 피하자. 