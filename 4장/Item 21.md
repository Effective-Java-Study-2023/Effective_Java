# 인터페이스는 구현하는 쪽을 생각해 설계하라 

## In Java 8

<hr>

Java 8이 등장하면서 여러 가지가 바뀌게 되었고,
대표적으로 람다, 스트림 등이 있지만
인터페이스 입장에서는 default 메소드가 추가되었다.

## Default Method?

<hr>

젭근 제어자는 public이며 메소드 body를 가질 수 있는 메소드.

Java 8 이전 버전에서 인터페이스는 계속 쓰여왔고,
그 당시의 인터페이스는 추상 클래스 말고는 어떠한 클래스도 가질 수 없었다.

Java 8 이전에서 인터페이스는 무조건 추상 메소드만 가질 것이라는 전제 하에 인터페이스를 상속받는 클래스는 사실상 default method의 존재를 모른 채 삽입된다.

## Java 8에서 추가된 Collection의 removeIf (default method)

<hr>

```Java
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```

아파치의 SynchronizedCollection 클래스는 Collection의 구현체와 잘 어우러지지 않는 예시

아파치 버전은 컬렉션 대신 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다.
즉, 모든 메소드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스이다.

SynchronizedCollection은 책이 쓰여진 시점에서는 removeIf를 재정의하고 있지 않다.

이 클래스를 자바 8과 사용한다면 모든 메소드 호출을 동기화해주지 못하게 되는 것이다.
따라서 removeIf의 구현은 동기화에 관해 아무것도 모르므로 락 객체를 사용할 수 없다.

## Multi-Thread 라면?

<hr>

removeIf에서 Lock을 걸어주지 않기 때문에 에러가 나거나 원하지 않는 결과가 발생할 가능성이 매우 높다.

## 해결책?

<hr>

removeIf를 재정의하자.

removeIf 호출 이전에 Lock

```Java
@Override 
public boolean removeIf(Predicate<? super E> filter) {
    synchronized (mutex) {
        return c.removeIf(filter);
    }
}
```

자바 플랫폼에 속하지 않는 제 3의 기존 컬렉션 구현체들은 이런 언어 차원의 인터페이스 변화에 발맞춰 수정될 기회가 없었으며,
그 중 일부는 여전히 수정되지 않고 있다.

## 결론

<hr>

디폴트 메소드는 런타임 오류를 일으킬 가능성이 있다. 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다.

기존 인터페이스에 디폴트 메소드로 새 메소드를 추가하는 것은 꼭 필요한 일이 아니라면 피해야 한다.
또, 인터페이스를 릴리즈 한 이후에도 결함을 수정하는게 가능한 경우도 있지만, 절대 그 가능성에 기대서는 안된다.
릴리즈 전에 반드시 테스트를 거쳐야 한다는 것이다.

## 번외

<hr>

```Java
static class SynchronizedCollection<E> implements Collection<E>, Serializable {
        @java.io.Serial
        private static final long serialVersionUID = 3053995032091335093L;

        @SuppressWarnings("serial") // Conditionally serializable
        final Collection<E> c;  // Backing Collection
        @SuppressWarnings("serial") // Conditionally serializable
        final Object mutex;     // Object on which to synchronize

        SynchronizedCollection(Collection<E> c) {
            this.c = Objects.requireNonNull(c);
            mutex = this;
        }

        SynchronizedCollection(Collection<E> c, Object mutex) {
            this.c = Objects.requireNonNull(c);
            this.mutex = Objects.requireNonNull(mutex);
        }

        public int size() {
            synchronized (mutex) {return c.size();}
        }
        public boolean isEmpty() {
            synchronized (mutex) {return c.isEmpty();}
        }
        public boolean contains(Object o) {
            synchronized (mutex) {return c.contains(o);}
        }
        public Object[] toArray() {
            synchronized (mutex) {return c.toArray();}
        }
        public <T> T[] toArray(T[] a) {
            synchronized (mutex) {return c.toArray(a);}
        }
        public <T> T[] toArray(IntFunction<T[]> f) {
            synchronized (mutex) {return c.toArray(f);}
        }

        public Iterator<E> iterator() {
            return c.iterator(); // Must be manually synched by user!
        }

        public boolean add(E e) {
            synchronized (mutex) {return c.add(e);}
        }
        public boolean remove(Object o) {
            synchronized (mutex) {return c.remove(o);}
        }

        public boolean containsAll(Collection<?> coll) {
            synchronized (mutex) {return c.containsAll(coll);}
        }
        public boolean addAll(Collection<? extends E> coll) {
            synchronized (mutex) {return c.addAll(coll);}
        }
        public boolean removeAll(Collection<?> coll) {
            synchronized (mutex) {return c.removeAll(coll);}
        }
        public boolean retainAll(Collection<?> coll) {
            synchronized (mutex) {return c.retainAll(coll);}
        }
        public void clear() {
            synchronized (mutex) {c.clear();}
        }
        public String toString() {
            synchronized (mutex) {return c.toString();}
        }
        // Override default methods in Collection
        @Override
        public void forEach(Consumer<? super E> consumer) {
            synchronized (mutex) {c.forEach(consumer);}
        }
        @Override
        public boolean removeIf(Predicate<? super E> filter) {
            synchronized (mutex) {return c.removeIf(filter);}
        }
        @Override
        public Spliterator<E> spliterator() {
            return c.spliterator(); // Must be manually synched by user!
        }
        @Override
        public Stream<E> stream() {
            return c.stream(); // Must be manually synched by user!
        }
        @Override
        public Stream<E> parallelStream() {
            return c.parallelStream(); // Must be manually synched by user!
        }
        @java.io.Serial
        private void writeObject(ObjectOutputStream s) throws IOException {
            synchronized (mutex) {s.defaultWriteObject();}
        }
    }
```

> // Override default methods in Collection

이 주석 밑 부분부터 디폴트 메소드를 재정의하는 코드