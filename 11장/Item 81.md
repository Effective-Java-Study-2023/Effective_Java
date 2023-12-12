# Item 81 - wait와 notify보다는 동시성 유틸리티를 애용하라

<hr>

wait와 notifiy는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자

## wait와 notify?

<hr>

스레드의 상태 제어를 위한 메소드

wait()는 가지고 있던 고유 락을 해제하고, 스레드를 잠들게 하는 역할을 하는 메서드이고,

notify()는 잠들어 있던 스레드 중 임의로 하나를 골라 깨우는 역할을 하는 메서드이다.

## 고수준 동시성 유틸리티

<hr>

java.util.concurrent 패키지에는 고수준의 동시성 유틸리티를 제공한다. 
크게 세가지로 분류해보면 실행자 프레임워크, 동시성 컬렉션 그리고 동기화 장치로 나눌 수 있다.

## 동시성 컬렉션

<hr>

List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 추가한 것이다. 
높은 동시성을 위해 동기화를 내부에서 수행한다. 동시성을 무력화하는 것이 불가능하며, 
외부에서 락(Lock)을 걸면 오히려 속도가 더 느려진다.

## 상태 의존적 메서드

<hr>

동시성 컬렉션의 동시성을 무력화하지 못하기 때문에 여러 메서드를 원자적으로 묶어 호출하는 것도 못한다.
그래서 여러 동작을 하나의 원자적 동작으로 묶는 상태 의존적 메서드가 추가되었다. 
몇몇 메서드는 매우 유용하여 일반 컬렉션 인터페이스의 디폴트 메서드 형태로 추가되었다.

예를 들면 putIfAbsent는 Map의 디폴트 메서드인데 인자로 넘겨진 key가 없을 때 value를 추가한다. 
기존 값이 있으면 그 값을 반환하고 없는 경우에는 null을 반환한다. 

String의 intern 메서드를 아래와 같이 흉내를 낼 수 있다.

> intern 메서드? <br>
> String 객체의 intern 메서드가 호출되면 이 풀에 해당 문자열과 같은(String.equals() 반환값이 true인) String 객체가 존재하는 경우 해당 객체를 반환하고, 
> 존재하지 않는 경우 해당 객체를 풀에 추가하고 해당 객체를 반환하게 됩니다.

```Java
private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

public static String intern(Stirng s) {
    String previousValue = map.putIfAbsent(s, s);
    return previousValue == null ? s : previousValue;
}
```

ConcurrentHashMap은 get 같은 검색 기능에 최적화되었다. 
따라서 get을 먼저 호출하여 필요할 때에만 putIfAbsent를 호출하면 더 빠르다.

```Java
public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null) {
            result = s;
        }
    }
    return result;
}
```

동기화한 컬렉션보다 동시성 컬렉션을 사용해야 한다. 
예를 들어 Collections의 synchronizedMap보다는 ConcurrentHashMap을 사용하는 것이 훨씬 좋다. 
동기화된 맵을 동시성 맵으로 교체하는 것 하나만으로 성능이 개선될 수 있다.

## 동기화 장치

<hr>

스레드가 다른 스레드를 기다릴 수 있게 하여 서로의 작업을 조율할 수 있도록 해준다. 
대표적인 동기화 장치로는 CountDownLatch와 Semaphore가 있으며 CyclicBarrier와 Exchanger도 있다.
가장 강력한 동기화 장치로는 Phaser가 있다.

CountDownLatch는 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다린다. 
생성자 인자로 받는 정수값은 래치의 countdown 메서드를 몇 번 호출해야 대기하고 있는 스레드들을 깨우는지 결정한다.

예를 들어 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 코드를 아래와 같이 작성할 수 있다.

```Java
public class CountDownLatchTest {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        try {
            long result = time(executorService, 3,
                    () -> System.out.println("hello"));
            System.out.println(result);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            executorService.shutdown();
        }
    }

    public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비가 됐음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // 타이머에게 작업을 마쳤음을 알린다.
                    done.countDown();
                }
            });
        }

        ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
}
```

위 코드에서 executor는 concurrency 매개변수로 지정한 값만큼의 스레드를 생성할 수 있어야 한다. 
그렇지 않으면 메서드 수행이 끝나지 않는데 이를 스레드 기아 교착 상태라고 한다. 
또, 시간을 잴 때는 시스템 시간과 무관한 System.nanoTime을 사용하는 것이 더 정확하다.

## wait와 notify 메서드를 사용해야하는 상황에는?

<hr>

어쩔 수 없이 wait()와 notify()를 사용해야하는 상황에는?

반드시 동기화 영역 안에서 사용해야 하며,
항상 반복문 안에서 사용해야 한다.

```Java
synchronized (obj) {
    while (조건이 충족되지 않았다) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
    }

    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다. 
대기 전에 조건을 검사하여 조건이 충족되었다면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치다.

만약 조건이 이미 충족되었는데 스레드가 notify 또는 notifyAll 메서드로 먼저 호출한 후 대기 상태로 빠지면,
그 스레드를 다시 깨우지 못할 수 있다.

한편, 대기 후에 조건을 검사하여 조건을 충족하지 않았을 때 다시 대기하게 하는 것은 잘못된 값을 계산하는 안전 실패를 막기 위한 조치다. 
그런데 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 있다.

- notify를 호출하여 대기 중인 스레드가 깨어나는 사이에 다른 스레드가 락을 거는 경우
- 조건이 만족되지 않았지만 실수 혹은 악의적으로 notify를 호출하는 경우
- 대기 중인 스레드 중 일부만 조건을 충족해도 notifyAll로 모든 스레드를 깨우는 경우
- 대기 중인 스레드가 드물게 notify 없이 깨어나는 경우. 허위 각성(spurious wakeup)이라고 한다.

일반적으로 notify보다는 notifyAll을 사용하는 것이 안전하며, 
wait는 항상 while문 내부에서 호출하도록 해야 한다.

## 정리

<hr>

- 코드를 새로 작성한다면 wait와 notify를 쓸 이유가 거의(어쩌면 전혀) 없다.
- 만약 사용해야한다면 wait는 while문 안에서 호출해야한다.
- 일반적으로 notify()보다는 notifyAll()을 사용하자
- 혹시라도 notify를 사용하다면 응답 불가 상태에 빠지지 않도록 조심하자