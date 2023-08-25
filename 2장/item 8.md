# 아이템8. finalizer와 cleaner 사용을 피하라
아이템 제목부터 사용을 피하라고 한다.

자바는 finalizer , cleaner 이렇게 두 가지 객체 소멸자를 제공한다.
하지만 finalizer는 예측할 수 없고 , 상황에 따라 위험할 수 있어 일반적으로 불필요하다.

### Finalize 동작원리
GC가 어떤 객체를 memory 해제하려고 하는데 그 객체가
finalize 메소드를 재정의한 경우 즉각적으로 회수 되지 않는다.
대신 Finalization Queue에 들어간 후 Finalizer에 의해 정리가 된다.
Finalizer는 객체의 finalize 메소드를 실행한 후 메모리 정리 작업을 수행한다.

자바 9에서는 finalizer가 deprecated 하였고 그 대안으로 cleaner를 소개하였다.
![스크린샷 2023-08-24 오후 1.21.33.png](..%2F..%2F%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-08-24%20%EC%98%A4%ED%9B%84%201.21.33.png)
cleaner는 finalizer보다는 덜 위험하지만 , 여전히 예측할 수 없고 , 느리고 , 일반적으로 불필요하다.

### 사용을 피해야하는 이유1
언제 수행될지 모른다. finalizer와 cleaner로 제때 실행되어야하는 작업은 절대 할 수 없다. 타이밍이 중요한 작업은 절대 사용하면 안된다. 예를 들어, 파일 닫기를 처리한다면 시스템이 finalizer나 cleaner실행을 게을리 해서 동시에 열 수 있는 파일 개수를 초과해 중대한 오류를 일으킬 수 있다.

### 사용을 피해야하는 이유2
인스턴스의 자원 회수가 지연될 수 있다. Finalizer스레드는 다른 애플리케이션 스레드보다 우선순위가 낮아서 실행될 기회를 얻지 못할 수도 있다.

cleaner는 자신을 수행할 스레드를 제어할 수 있지만 백그라운드에서 동작하고 GC통제하에 있어서 언제 수행될지 모른다.

### 사용을 피해야하는 이유3
수행시점 뿐만 아니라 수행여부도 보장하지 않는다. Finalizer나 Cleaner로 상태를 영구적으로 수정하는 작업은 하면 안된다. 예를 들어 데이터베이스 같은 공유자원의 영구락 해제를 저걸로 한다면 분산 시스템 전체가 멈출 수도 있다.

System.gc나 System.runFinalization 역시 실행될 가능성을 높어주긴 하지만 보장하지 않는다.

System.runFinalizersOnExit, Runtime.runFinalizersOnExit는 심각한 결함으로 deprecated된 상태다.

### 사용을 피해야하는 이유4
심각한 성능 문제를 동반한다. AutoCloseable객체를 만들고, try-with-resources로 수거하면 12ns가 걸린 반면 finalizer는 550ns가 걸렸다. Cleaner는 66ns로 5배 정도 느렸다.

### 사용을 피해야하는 이유5
공격에 노출되어 심각한 보안 문제를 일으킬 수 있다. A클래스를 공격하려는 B클래스가 A클래스를 상속받고 B클래스 인스턴스를 생성하는 도중 예외가 발생하거나, 직렬화할때 예외가 생기면 죽었어야할 객체의 finalizer가 실행될 수 있다.
그럼 그 안에서 인스턴스의 레퍼런스를 기록할 수 있고 GC가 되지 못할 수 있다. 그 안에서 인스턴스의 메서드를 호출할 수 있다.
예외발생시 없어져야할 인스턴스인데 Finalizer때문에 죽지 않고 살아있는 것이다.
Final클래스는 하위클래스를 만들 수 없기 때문에 finalizer공격으로부터 방어하려면 finalize메서드를 만들고 final로 선언해야 한다. 

### 자원 반납 방법
AutoCloseable 인터페이스를 구현하고 close메서드를 구현하면된다. 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야 한다.
추가로 각 인스턴스가 자신이 닫혔는지를 추적하기 위해 close메서드는 이미 종료된 상태인지 확인 후 반납이 끝난 상태에서 close가 호출되면 IllegalStateException을 던진다.

### 안전망으로 사용
Finalizer나 Cloeaner는 자원의 소유자가 close메서드를 호출하지 않을 것에 대비한 안전망 역할이다.
실제로 FileInputStream, FileOutputStream, ThreadPoolExcutor이 대표적으로 안전망으로 동작하는 Finalizer가 있다.

다음은 안전망으로 사용된 Cleaner 예시이다.
``` java
public class Room implements AutoCloseable {

    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안된다.
    private static class State implements Runnable {
        int numJunkPiles; // 방안의 쓰레기 수

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // Close메서드나 cleaner가 호출된다.
        @Override
        public void run() {
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }

    // 방상태 cleanable과 공유
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```
State는 cleaner가 청소할 때 수거할 자원을 담고 있다.

cleanable객체는 Room생성자에서 cleaner에 Room과 State를 등록할 때 얻는다.

Room의 close가 호출될 때 run메서드가 호출된다. 

혹은 GC가 Room을 회수할 때까지 close가 호출되지 않으면 cleaner가 run메서드를 호출해줄 것이다.

State인스턴스는 절대 Room인스턴스를 참조해서는 안된다.( 순환 참조로 인해 GC가 Room인스턴스를 회수하지 못함)
``` java
public static void main(String[] args) throws Exception {
    try (Room myRoom = new Room(1)) {
        System.out.println("청소 하자이");
    }  // try문의 경우 close가 호출되면서 방청소가 된다.
    new Room(2); // 방청소가 될지 안될지 예측할 수 없다.
    System.out.println("청소 안하냐");
}
```

### 네이티브피어 정리할 때 사용
네이티브 객체는 일반객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다. 자바 객체가 아니라서 GC는 그 존재를 모른다.
성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 갖고 있지 않을 때 finalizer나 cleaner를 통해 자원을 회수 할 수 있다. 
즉시 회수해야할 경우는 close메서드를 사용한다. (사실 그냥 close 메서드를 쓰면 되지 않나...)

