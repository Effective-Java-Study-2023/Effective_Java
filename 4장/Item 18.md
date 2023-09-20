# 아이템 18 상속보다는 컴포지션을 사용하라 

> "상속"은 코드를 재사용하는 가장 좋은 방법으로, 상위 클래스와 하위 클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서 사용하면 안전한다.
> 또한, 확장할 목적으로 설계되었고 문서화도 잘 된 클래스도 안전하다.
* 상속 : 클래스가 다른 클래스를 확장하는 구현 상속

### 하지만..  구체 클래스를 상속하는 것은 메서드 호출과 달리 캡슐화를 깨트릴 수 있다. 

### 잘못된 상속의 예
~~~java
    // 책에 나온 예시 
    public class InstrumentedHashSet<E> extends HashSet<E> {
        private int addCount = 0;
    
        public InstrumentedHashSet() {
        }
    
        public InstrumentedHashSet(int initCap, float loadFactor) {
            super(initCap, loadFactor);
        }
    
        @Override
        public boolean add(E e) {
            addCount++;
            return super.add(e);
        }
        
        @Override
        public boolean addAll (Collection< ? extends E > c){
            addCount += c.size();
            return super.addAll(c);
    
        }
        public int getAddCount() {
            return addCount;
        }
    }

~~~

~~~Java
    public class Example18 {
        public static void main(String[] args) {
            InstrumentedHashSet<String> hs = new InstrumentedHashSet<>();
            hs.addAll(List.of("틱", "탁탁", "펑"));
            System.out.println(hs.getAddCount());  // 6 반환
        }
    }
~~~
- HashSet 의 addAll 메서드는 add 메서드를 사용하여 구현 되어 있다. 
- 재정의 된 add 메서드가 불려져서 addCount++ 가 된 후 다시 addAll 을 수행하여 ++ 된다.
- 따라서 3이 아닌 6을 반환하게 된다. 
~~~java
  // hashSet 의 addAll 메서드 
  public boolean addAll(Collection<? extends E> c) {
      boolean modified = false;  // 실제 변화가 있는지 확인
      for (E e : c)
          if (add(e))  // 만약 원소가 이미 Hashset에 있다면, add 메서드는 false 반환
              modified = true;
      return modified;
  }
~~~

<details><summary>위 코드를 컴포지션만 사용한 경우</summary>
<div>

~~~java
  public class InstrumentedSet<E> {
      private final HashSet<E> set; // 내부에서 사용할 HashSet
      private int addCount = 0;
  
      public InstrumentedSet() {
          this.set = new HashSet<>();
      }
  
      public InstrumentedSet(int initCap, float loadFactor) {
          this.set = new HashSet<>(initCap, loadFactor);
      }
  
      public boolean add(E e) {
          addCount++;
          return set.add(e);
      }
  
      public boolean addAll(Collection<? extends E> c) {
          addCount += c.size();
          return set.addAll(c);
      }
  
      public int getAddCount() {
          return addCount;
      }
  }
~~~
- 전달 없이 컴포지션만 사용하는 경우
  - 내부 객체의 메서드를 직접 호출하는 새로운 메서드를 매번 작성해야하 한다.
  -


</div></details> 

### 상속을 잘못한다면...
- 다음 릴리스에서도 유지할 수 있을지 알 수 없다. 
- 상위 클래스의 메서드 동작을 다시 구현하기 어렵다. 
- 하위 클래스에서는 접근할 수 없는 Private 필드를 써야한다면 구현할 수 없다. 
- 상위 클래스에 새로운 메서드를 추가한다면, 고려해야할 것이 많아진다. 
  - 하위 클래스에서 재정의하지 못한 새로운 메서드를 사용해 '허용되지 않은' 원소 추가를 막아야한다. 
  - 상위 클래스의 새로운 메서드가 하필 하위 클래스에 추가한 메서드와 시그니처가 같고 반환타입이 다르다면 컴파일 되지 않는다. 
- 상속을 잘못한 클래스 : 스택은 벡터가 아니지만 벡터를 상속 받음 
  - public class Stack<E> extends Vector<E>

### 해결책으로...
> 확장 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조 하자.  
> 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 '컴포지션'이라고 한다.

> 새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 결과를 반환한다. 
> 이 방식을 전달(forwarding)이라고 한다. 
> 새 클래스의 메서드들을 전달 메서드(forwarding method) 라고 부른다. 

새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나고, 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향 받지 않는다. 

~~~Java
  // 이 InstrumentedSet 클래스가 Set 인스턴스를 감싸고 있어서 래퍼 클래스
  public class InstrumentedSet<E> extends ForwardingSet<E>{
    private int addCount = 0;
  
    public InstrumentedSet(Set<E> s) {
      super(s);
    }
  
    @Override
    public boolean add(E e) {
      addCount++;
      return super.add(e);
    }
  
    @Override
    public boolean addAll(Collection<? extends E> c) {
      addCount += c.size();
      return super.addAll(c);
    }
  
    public int getAddCount() {
      return addCount;
    }
  }
~~~

~~~java 
  public class ForwardingSet<E> implements Set<E> {
  
      private final Set<E> s;
  
      public ForwardingSet(Set<E> s) {
          this.s = s;
      }
  
      public void clear() {
          s.clear();
      }
  
      public boolean contains(Object o) {
          return s.contains(o);
      }
  
      public boolean isEmpty() {
          return s.isEmpty();
      }
  
      public int size() {
          return s.size();
      }
  
      public Iterator<E> iterator() {
          return s.iterator();
      }
  
      public boolean add(E e) {
          return s.add(e);
      }
  
      public boolean remove(Object o) {
          return s.remove(o);
      }
  
      public boolean removeAll(Collection<?> c) {
          return s.removeAll(c);
      }
  
      public boolean containsAll(Collection<?> c) {
          return s.contains(c);
      }
  
      public boolean addAll(Collection<? extends E> c) {
          return s.addAll(c);
      }
  
      public boolean retainAll(Collection<?> c) {
          return s.retainAll(c);
      }
  
      public Object[] toArray() {
          return s.toArray();
      }
  
      public <T> T[] toArray(T[] a) {
          return s.toArray(a);
      }
  
      @Override
      public boolean equals(Object o) {
          return s.equals(o);
      }
  
      @Override
      public int hashCode() {
          return s.hashCode();
      }
  
      @Override
      public String toString() {
          return s.toString();
      }
  }
~~~

### 래퍼 클래스 - 상속 대신 컴포지션 사용
래퍼 클래스란 다른 인스턴스를 감싸고 있는 클래스이다. 

### 래퍼 클래스의 단점 
- 콜백 프레임워크와는 어울리지 않는다. (SELF 문제)
  - 콜백 프레임 워크 : 특정 동작을 완료한 후 실행되어야 할 메서드(콜백 메서드)를 정의하는 방식
  - SELF 문제 : 래퍼 객체가 내부 객체 대신 자신을 콜백으로 전달해야하는 상황에서 내부 객체를 전달하는 경우 발생 

### 컴포지션을 사용해야하는데 상속을 했다면...
- 내부 구현을 불필요하게 노출시킨다.
- 클라이언트가 내부에 직접 접근이 가능하다.
- 클라이언트가 상위 클래스를 직접 수정하여 하위 클래스의 불변식을 해칠 수 있다.

## 결론
- 상속은 캡슐화를 해칠 수 있다. 
- 명확하게 상위 클래스와 하위 클래스가 Is-a 관계일 때 상속을 쓴다.
  - 확장하려는 클래스의 API에 아무런 결함이 없는지 확인 할 것
  - 결함이 API까지 전파돼도 괜찮은지 생각해 볼 것
- 상속의 취약점을 피하기 위해서 컴포지션과 전달을 사용하자. 
  - (전달 메서드 작성이 귀찮더라도) 전달 클래스를 인터페이스당 하나씩 만드는 것을 추천
- 특히, 래퍼 클래스는 하위 클래스보다 견고하고 강력하므로 래퍼 클래스로 구현할 인터페이스가 있는 경우 컴포지션과 전달을 사용하자. 