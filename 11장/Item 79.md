# 아이템 79. 과도한 동기화는 피하라.

> 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳을 수 있다.  
> 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에게 양도하면 안 된다. 

- 동기화된 코드 블럭 안에서는 재정의 가능한 메서드를 호출해서는 안 된다. 
- 클라이언트가 넘겨준 함수 객체를 호출해서도 안 된다. 
- 이런 메서드는 동기화된 클래스 관점에서 외계인 메서드라고 부른다. 
  - 이 메서드가 하는 일에 따라 예외를 발생시키거나, 교착상태를 만들거나, 데이터를 훼손시킬 수 있다. 

### 잘못된 코드 - 동기화 블록 안에서 외계인 메서드를 호출
~~~java
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }
    
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for(SetObserver<E> observer : observers) {
                observer.added(this, element); // 외계인 메서드 호출
            }
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); //notifyElementAdded를 호출
        }
        return result;
    }

    @FunctionalInterface // 인터페이스가 함수형 인터페이스로 선언됨을 알림
    public interface SetObserver<E> {
      //ObservableSet 에 원소가 더해지면 호출된다.
      void added(ObservableSet<E> set, E element);
    }

}
~~~
- SetObserver 인터페이스는 ObservableSet 클래스에 의해 호출
- 이 메서드가 어떻게 구현되어 있는지는 ObservableSet 을 사용하는 외부 코드에서 정의 
- 따라서 added 메서드의 내용은 ObservableSet 이 아닌 외부에서 제공되므로 외계인 메서드라 할 수 있다. 

### 문제를 일으킬 수 있는 익명함수 콜백 : 예외 발생
집합에 추가된 정수값을 출력하다가 그 값이 23이면 자기 자신을 제거하는 관찰자
~~~java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if(e == 23) {
                s.removeObserver(this);
            }
        }
    });

    for(int i = 0; i < 100; i++) {
        set.add(i);
    }
}
~~~

<details><summary>ConcurrentModificationException</summary> 

- 동시 수정이 감지될 때 발생하는 예외

1. main 에서 set.add()가 호출되면 ObservableSet 재정의된 add()가 호출된다.
2. 재정의된 add 는 notifyElementAdded() 를 호출
3. notifyElementAdded()에서는 List<SetObserver<E>> 를 순회하며 added()를 호출한다.
4. main 에서 익명 함수로 정의한 added 가 호출된다. 해당 added 는 특정 조건에서 removeObserver 메서드를 호출한다.
5. 이때, removeObserver()가 호출되면 콜백으로 되돌아와 자신을 수정하는 것을 막지 못하므로 원소가 삭제된다.
6. notifyElementAdded()에서 순회 중인 동기화 블록에서 ConcurrentModificationException 이 발생한다.(동기화가 걸려있음에도 원소가 삭제됨 → 동기화 오류)

</details>

### 문제를 일으키는 스레드 : 교착 상태 
~~~java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

    set.addObserver(new SetObserver<Integer>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23) {
                ExecutorService exec = Executors.newSingleThreadExecutor();
                try {
                    // 여기서 lock이 발생하며, 메인 스레드가 작업을 기다리고 있다. 
                    // submit 의 get 메서드가 완료되기를 기다리는 중
                    exec.submit(() -> s.removeObserver(this)).get();
                } catch (ExecutionException | InterruptedException ex) {
                    throw new AssertionError(ex);
                } finally {
                    exec.shutdown();
                }
            }
        }
    });

    for (int i = 0; i < 100; i++)
        set.add(i);
}
~~~
- ExecutorService 를 통해 만들어진 백그라운드 스레드가 s.removeObserver 를 호출하면 Observer 를 잠그려 시도하지만 락을 얻을 수 없다. 
- 왜냐하면 메인 스레드가 락을 쥐고 있다.
- 메인 스레드는 백그라운드 스레드가 Observer 를 제거하기를 기다린다. (== 교착 상태)

### 교착 상태 해결 방법 : 열린 호출 
- 외계인 메서드 호출을 동기화 블록 바깥으로 옮기기 
<details><summary>원래 코드</summary>

~~~java
// 원래
private void notifyElementAdded(E element) {  // 요소가 추가될 때 등록된 모든 관찰자에게 알린다.
    synchronized (observers) {
        for(SetObserver<E> observer : observers) {
            observer.added(this, element); // 외계인 메서드(added) 호출
        }
    }
}
~~~
</details>

~~~java
// 문제 해결
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element); //외계인 메서드를 동기화 블록 바깥으로 옮겼다.
    }
}
~~~
- 외계인 메서드 호출 전에 관찰자 리스트를 복사하면 복사본을 통해 안전하게 순회 가능하다. 
  - 락을 사용하여 리스트 복사 후에는 원본 리스트의 상태와 상관없이 복사본을 사용하여 외계인 메서드 호출 
- 예외 발생과 교착 상태 증상 해결 가능 
- 동시성 효율을 높여준다.
  - 외계인 메서드는 얼마나 오래 실행될지 알 수 없다. 
  - 동기화 영역 안에서 호출 시 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야 하기 때문에

### (더 나은) 교착 상태 해결 방법 : CopyOnWriteArrayList 
- 자바의 동시성 컬렉션 라이브러리 CopyOnWriteArrayList 사용 
- ArrayList 를 구현한 클래스로 내부 변경 시 항상 깨끗한 복사본을 만들어 수행하도록 구현되어 있다. 
- 내부의 배열은 절대 수정되지 않으니 순회시 락이 필요 없어 빠르다.
- 수정할 일이 적고 순회만 빈번하게 일어나는 관찰자 리스트 용도로 사용하기 좋다. 

<details><summary>원래 코드</summary>

~~~java
private final List<SetObserver<E>> observers = new ArrayList<>(); 

public void addObserver(SetObserver<E> observer) {  
  synchronized (observers) {
    observers.add(observer);
  }
}

public boolean removeObserver(SetObserver<E> observer) { 
  synchronized (observers) {
    return observers.remove(observer);
  }
}

private void notifyElementAdded(E element) {  
    synchronized (observers) {
        for(SetObserver<E> observer : observers) {
            observer.added(this, element); // 외계인 메서드(added) 호출
        }
    }
}
~~~
</details>

~~~java
// 명시적으로 동기화한 곳이 사라졌다. 
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
~~~

### 동기화의 성능 
과도한 동기화가 초래하는 진짜 비용은 서로 스레드끼리 경쟁하느라 낭비하는 시간이다.  

- 병렬로 실행할 기회를 잃는다. 
- 모든 코어가 메모리를 일관되게 보기 위한 지연시간(캐시 일관성)이 진짜 비용
  - 각 코어는 자체 캐시를 갖고, 메모리의 일부를 캐시에 복제
  - 캐시 일관성을 유지하기 위해 캐시 간의 데이터를 동기화 
  - 일관되게 보기 위해 추가적인 지연시간이 필요
- 가상머신의 코드 최적화를 제한하는 점도 숨은 비용이다. 

### 가변 클래스를 작성하는 경우 동기화에 대해 고려할 점 
1. 동기화를 전혀 하지 말고, 가변 클래스를 동시에 사용하는 클래스가 외부에서 동기화 하자.
2. 동기화를 내부에서 수행하여 스레드 안전한 클래스로 만들자.
   - 단 클라이언트가 외부에서 전체에 락을 거는 것보다 효율성이 좋을때만 

### 클래스 내부에서 동기화하기로 했다면... 
다양한 기법으로 동시성을 높이자. 
- 락 분할 
  - 하나의 클래스에서 기능적으로 Lock 을 분리하여 사용하는 것 
- 락 스트라이핑
  - 자료구조 관점에서 한 자료구조 전체가 아닌 일부분에 락을 적용
- 비차단 동시성 제어 
  - 락 사용하는 대신, 원자적 연산과 비차단 알고리즘을 사용하여 동시성을 만드는 방법

## 결론
- 교착 상태와 데이터 훼손을 피하기 위해 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자. 
- 동기화 영역 안에서의 작업은 최소한으로 하자. 
- 가변 클래스 설계 시 스스로 동기화가 필요한지 고민하자. 
- 멀티코어 세상에서 과도한 동기화는 피하자. 
- 합당한 이유가 있을 때만 내부에서 동기화 하고, 동기화 여부를 문서에 명확히 작성하자. 

-----
<details><summary>Observer Pattern</summary>
<div>
관찰자 패턴은 관찰자들이 관찰하고 있는 대상의 상태에 변화가 있을 때마다 대상자는 직접 목록의 각 관찰자들에게 통지하고, 관찰자들은 알림을 받아 조치를 취하는 행동 패턴을 말한다.  

관찰자 패턴은 일대다 종속성을 정의한다. 주로 분산 이벤트 처리 시스템에서 사용되고, 'observer' 인터페이스와 'observable' 클래스를 통해 구현한다. 
</div></details>


<details><summary>synchronized</summary>
<div>

- synchronized 키워드는 멀티스레드 환경에서 동기화를 제공하는 기능이다. 
- 특정 코드 블록이나 메서드를 synchronized 로 감싸면 해당 코드 블록이나 메서드는 하나의 스레드에 의해서만 동시에 실행될 수 있다. 
- 이것이 락을 실행한다는 의미이다. 
- synchronized 는 동기화를 통해 스레드 간의 안전한 작업을 보장합니다. 
</div></details>

---
참조  
https://jjingho.tistory.com/129