# 아이템 82. 스레드 안전성 수준을 문서화하라.
> - 하나의 메서드를 여러 스레드가 동시 호출 시 그 메서드가 어떻게 동작하는지는 해당 클래스와 클라이언트 사이의 중요한 계약이다. 
> - 자바독 기본 옵션에서 생성한 API 문서에는 synchronized 한정자가 포함되지 않는다. 

### 클래스가 지원하는 스레드 안전성 수준
<안전성이 높다>
1. 불변(immutable)
   - 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화도 필요 없다. 
   - String, Long, BigInteger 등 
2. 무조건적 스레드 안전(unconditionally thread-safe)
   - 인스턴스는 수정될 수 있지만, 별도의 외부 동기화 없이 동시에 사용해도 안전하다. 
   - AtomicLong, ConcurrentHashMap 등 
3. 조건부 스레드 안전(conditionally thread-safe)
   - 무조건적 스레드 안전과 같지만, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다. 
   - Collections.synchronized 래퍼 메서드가 반환한 컬렉션들이 여기 속한다. 
   - 이 컬렉션들이 반환하는 반복자는 외부에서 동기화해야 한다. 
4. 스레드 안전하지 않음(not thread-safe)
   - 이 클래스의 인스턴스는 수정될 수 있다.
   - 동시에 사용하려면 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야 한다. 
   - ArrayList, HashMap 같은 기본 컬렉션 
5. 스레드 적대적(thread-hostile)
   - 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티 스레드 환경에서 안전하지 않다.
   - 고의로 만드는 사람은 없지만 동시성을 고려하지 않고 작성하면 우연히 만들어질 수도.. 
   - 아이템 78의 generateSerialNumber 에서 내부 동기화를 생략하는 경우 

### 스레드 안전성 애너테이션 
애너테이션을 사용하므로써 스레드 안전성 수준을 명시적으로 표현하고 문서화하는데 도움을 줄 수 있다.
1. @Immutable
   - 해당 클래스가 불변임을 나타낸다. 
   - 불변
2. @ThreadSafe
   - 클래스나 메서드가 스레드 안전하다는 것을 말한다. 
   - 무조건적 스레드 안전, 조건부 스레드 안전
     - 조건부 스레드 안전한 클래스는 주의해서 문서화해야 한다. 
  
<details><summary>조건부 스레드 안전의 문서화</summary><div>

- 어떤 순서로 호출시 외부 동기화가 필요한지  
- 그 순서로 호출시 어떤 락을 얻어야 하는지 알려줘야 한다.
- 예시 : Collections.synchronizedMap
   ~~~java
   /*
   Returns a synchronized (thread-safe) map backed by the specified
   map.  In order to guarantee serial access, it is critical that
   <strong>all</strong> access to the backing map is accomplished
   through the returned map.<p>
   
   It is imperative that the user manually synchronize on the returned
   map when traversing any of its collection views via {@link Iterator},
   {@link Spliterator} or {@link Stream}:
   <pre>
   Map m = Collections.synchronizedMap(new HashMap());
      ...
   Set s = m.keySet();  // Needn't be in synchronized block
      ...
   synchronized (m) {  // Synchronizing on m, not s!
      Iterator i = s.iterator(); // Must be in synchronized block
      while (i.hasNext())
          foo(i.next());
   }
    
  Failure to follow this advice may result in non-deterministic behavior.
   */
   ~~~
   synchronizedMap 이 반환한 맵의 컬렉션 뷰를 순회하려면 반드시 그 맵을 락으로 사용해 수동으로 동기화하자.  
   코드대로 따르지 않으면 동작을 예측할 수 없다. 
</div></details>

3. @NotThreadSafe
   - 클래스나 메서드가 스레드 안전하지 않다는 것을 나타낸다. 
   - 스레드 안전하지 않음


### 외부에 Lock 제공
- 클래스가 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 일련의 메서드 호출을 원자적으로 수행 가능 
- 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용할 수 없다. 
- 클라이언트가 서비스 거부 공격(denial-of-service attack)을 수행할 수도 있다. 
  - 서비스 거부 공격이란 클라이언트가 공개된 락을 쥐고 놓지 않는 것을 말한다. 
  - 서비스 거부 공격을 막으려면 비공개 락 객체를 사용해야 한다.
    - 비공개 락 객체란 특정 객체에 대한 동기화를 담당하는 락을 의미
  ~~~java
  // 비공개 락 객체 관용구 - 서비스 거부 공격을 막아준다. 
   public class Counter {
      private int counter = 0;
      private final Object lock = new Object();
   
      public void increment() {
         synchronized (lock) {
               counter++;
         }
      }
   }  
   ~~~
  - lock 객체는 Counter 클래스의 비공개 락 
  - 락 필드는 항상 final 로 선언하여, 우연히라도 락 객체가 교체되는 일을 예방한다. 
  - 비공개 락 객체 관용구는 클래스 내부에서만 사용되는 락을 외부로 노출하지 않는다. 

### 비공개 락 객체 관용구
- 무조건적 스레드 안전 클래스에서만 사용 가능하다. 
- 조건부 스레드 안전 클래스에서는 사용할 수 없다. 
  - 특정 호출 순서에 필요한 락이 무엇인지 알려주어야 하기 때문에
- 상속용으로 설계한 클래스에 특히 유용하다.
  - 주의할 점
  - 상속용 클래스에서 자신의 인스턴스를 락으로 사용할 경우 
  - 하위 클래스는 쉽게, 의도치 않게 기반클래스의 동작을 방해할 수 있다. (예. 오버라이드)
  - 같은 락을 다른 목적으로 사용하는 것으로 서로가 서로를 훼방 놓을 수 있다. 

## 결론
- 모든 클래스가 자신의 스레드 안전성 정보를 명확히 문서화해야 한다. 
  - 정확한 언어로 명확히 설명하거나 애너테이션을 사용
- synchronized 한정자는 문서화와 관련이 없다. 
