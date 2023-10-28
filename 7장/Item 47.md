# 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다. 

자바 7 이전 Array 형태의 선형 자료구조를 반환하는 메서드의 반환 타입으로 아래와 같은 타입을 사용했다. 
- Collection, Set, List 와 같은 컬렉션 인터페이스 
- 배열
- Iterable 인터페이스 

 기본적으로 컬랙션 인터페이스를 사용한다.
for-each 에서만 사용하거나, 일부 Collection 메서드를 구현할 수 없는 경우 Iterable 인터페이스를 사용한다. 
반환 원소들이 기본 타입이거나 성능에 민감하면 배열을 쓴다.

그러나 자바 8 이후 스트림이 생기면서 선택이 복잡해졌다. 

## API 에서 Stream 만 반환하는 경우
- for-each 문을 사용하기 어렵다. 
- 스트림파이프 라인에서만 쓰인다면, 스트림을 반환하자. 
### Stream 은 반복(loop)을 지원하지 않는다. 
스트림과 반복을 알맞게 조합해야 한다.  
Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하며, Iterable 인터페이스가 정의한 방식대로 동작한다.  
하지만, Iterator 를 확장(extends) 하지 않아 for-each 문으로 반복하지 못한다. 

~~~java
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
}
// for-each 문은 iterable 인터페이스를 구현한 객체를 대상으로만 사용 가능하다.(iterator 인터페이스 사용 안 됨)

->

// 형 변환 : 난잡하고, 직관성이 떨어진다. 
for (ProcessHandle ph : (Iterable<ProcessHandle>)ProcessHandle.allProcesses()::iterator) {
}
~~~

### 중개 어탭터 
- 자바의 타입 추론이 문맥을 잘 파악하여 어댑터 메서드 안에서 따로 형변환하지 않아도 된다.
- 어댑터를 사용하면 어떤 스트림도 for-each 문으로 반복할 수 있다. 

~~~java
for (ProcessHandle ph : iterableOf(ProcessHandle.allProcesses())) {
}

public static <E> Iterable<E> iterableOf(Stream<E> stream) { // iterable<E> 타입 반환
    return stream::iterator;
}
~~~

## API 에서 Iterable 만 반환하는 경우 
- 반환된 객체들이 반복문에서만 쓰일걸 안다면 Iterable 을 반환하자.
- stream 코드가 편한 개발자들에게 불편함을 초래한다. 
- 이 경우도 Iterable 을 Stream 으로 중개해주는 메서드를 생성하자. 
~~~java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
~~~

## 모두를 배려하는 API 
- 공개 API를 작성하는 경우 
- 한 방식만 사용할 거라는 근거가 없는 경우  
*Collection* 이나 그 하위 타입으로 반환하도록 하자.
  - Collection 인터페이스는 Iterable 의 하위 타입이고, stream 메서드도 제공하므로 반복과 스트림을 동시에 지원하기 때문이다. 

### 컬렉션 내의 시퀀스가 크면 전용 컬렉션을 구현하라. 
- 반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList, HashSet 같은 표준 컬렉션 구현체를 반환하는 것이 최선일 수 있다.
- 반환할 시퀀스가 크지만, 표현을 간결하게 할 수 있다면, 전용 컬렉션을 구현해보자.

<details><summary> 예시 코드</summary>
<div>

### 입력 집합의 멱집합을 전용 컬렉션에 담아 반환한다.
- AbstractList 를 이용하여 전용 컬렉션 구현
- 입력 집합의 원소 수가 30을 넘으면 예외를 던진다. 
- Collection 을 사용할 경우의 단점이 될 수 있다. 
- 만약 (반복 시작 전에 시퀀스 내용을 확정할 수 없다는 등의 이유로) contains 와 size 를 구현하는 게 불가능하다면, 스트림이나 Iterable 을 반환하는 편이 낫다.

~~~java
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
       List<E> src = new ArrayList<>(s);
       if(src.size() > 30) {
           throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최대 30개).: " + s);
       }

       return new AbstractList<Set<E>>() {
           @Override
           public int size() {
               return 1 << src.size();
           }

           @Override
           public boolean contains(Object o) {
               return o instanceof Set && src.containsAll((Set) o);
           }

           @Override
           public Set<E> get(int index) {
               Set<E> result = new HashSet<>();
               for (int i = 0; index != 0; i++, index >>=1) {
                   if((index & 1) == 1) {
                       result.add(src.get(i));
                   }
               }
               return result;
           }
       };
    }
}
~~~

### Stream 을 이용한 방법 
~~~java
public class SubList {

    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()), 
                             prefixes(list).flatMap(SubList::suffixes));
    }

    public static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                        .mapToObj(end -> list.subList(0, end));
    }

    public static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.rangeClosed(0, list.size())
                        .mapToObj(start -> list.subList(start, list.size()));
    }
}
~~~

</div>
</details>

## 결론 
원소 시퀀스를 반환하는 메서드를 작성할 때는 
- 스트림으로 처리하려는 사용자와 반복으로 처리하려는 사용자 모두 있음을 생각하고, 컬렉션으로 반환하자. 
  - 원소의 개수가 적거나, 컬렉션에 담아 관리한다면, ArrayList 같은 표준 컬렉션에 담아 반환하자. 
  - 그것도 안 된다면, 전용 컬렉션을 구현할지 고민하자. 
- 컬렉션 반환이 불가능하면
  - 스트림과 Iterator 중 더 자연스러운 것을 반환하자.

---
## Iterator & Iterable
### Iterable
- 자바에서 반복 가능한 객체를 나타내는 인터페이스 
- 주요 메서드 iterator() 를 가지고, Iterator 객체를 반환한다. 
- ArrayList, HashSet 등의 컬렉션 클래스들은 iterable 인터페이스를 구현하고 있다. 

### Iterator
- 항목의 시퀀스(일련의 원소)를 순회하는 인터페이스 
- 주요 메서드는 hashNext() (다음 항목이 있는지 확인), next() (다음 항목을 반환)가 있다. 

## 중개 어댑터 
- 호환되지 않는 인터페이스 사이의 다리 역할을 하는 디자인 패턴이다. 
- 두 시스템 간의 인터페이스가 호환되지 않는다면 중개 어댑터를 사용하여 이를 연결할 수 있다. 
- 이 패턴은 코드의 재사용성을 높이고, 다양한 시스템이나 라이브러리 간의 연결을 유연하게 만들어준다. 
- 주요 구성 요소 
  - Target : 현재 코드가 사용하고 있는 인터페이스 
  - Adaptee : 적응시키고자 하는 즉 현재 코드와 호환되지 않는 인터페이스
  - Adapter : Target 인터페이스를 구현하고, 내부에서 Adaptee 를 사용한다. Adapter 는 Target 을 Adaptee 에 적응시키는 역할을 한다. 



