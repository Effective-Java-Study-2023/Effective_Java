# 아이템 30. 이왕이면 제네릭 메서드로 만들어라

### 메서드에 제네릭을 쓸 수 있다.
- 로 타입 사용하지 말고, 메서드에 **제네릭을 사용**하여 타입 안전성을 보장 받을 수 있다. 
~~~java
public static <E> Set<E> uni(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
~~~

- < E > : 메서드에서 타입 매개변수 E 선언. 메서드 내에서 사용될 제네릭 타입을 나타낸다. (타입 매개변수 목록)
- Set< E > : 메서드의 반환 타입

<details><summary>main 클래스</summary>
<div>

* 로타입을 작성한 경우
~~~java
public static void main(String[] args){
    Set set1=new HashSet<>();
    Set set2=new HashSet<>();
    
    set1.add(1);
    set1.add(2);
    set1.add(3);
    
    set2.add("헤이");
    set2.add("거기");
    set2.add("메롱");
    
    Set result=union(set1,set2);
    System.out.println(result);
}
~~~

* 제네릭을 사용하여 메서드 타입을 안전하게 만든 경우 (컴파일에러)
~~~java
public static void main(String[] args) {
    Set<Integer> set3 = new HashSet<>();
    Set<String> set4 = new HashSet<>();
    set3.add(1);
    set3.add(2);
    set3.add(3);

    set4.add("헤이");
    set4.add("거기");
    set4.add("메롱");
    Set<String> result1 = uni(set3, set4); // 컴파일 에러

    System.out.println(result1);
}
~~~
</div>
</details>
<details><summary>
한정적 와일드카드 타입 
</summary>
<div>

~~~java
public static <E> Set<E> uni(Set<? extends E> s1, Set<? extends E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
~~~

~~~java
Set<Integer> intSet = new HashSet<>(Arrays.asList(1, 2, 3));
Set<Double> doubleSet = new HashSet<>(Arrays.asList(1.1, 2.2, 3.3));
Set<Number> numberSet = uni(intSet, doubleSet);
~~~
한정적 와일드카드 타입을 사용하여 메서드가 더 유연해져서 서로 다른 서브타입의 Set 인스턴스를 받아들일 수 있다. 
</div>
</details>

### 제네릭 싱글턴 팩터리 
- 제네릭은 런타임에 타입 정보가 소거되므로, 제네릭 싱글턴 팩터리를 사용하면 불변 객체를 다양한 타입 매개변수에 대해 안전하게 재사용할 수 있다.   
- 제네릭 싱글턴 팩터리는 싱글턴 객체를 여러 타입으로 사용할 수 있게 만든다. 
  - 생성은 한 번만 되고, 다양한 타입 매개변수에 대해 재사용할 수 있다.
~~~java
public static <T> Comparator<T> reverseOrder() {
    return (Comparator<T>) ReverseComparator.REVERSE_ORDER;
}
~~~
 - 런타임시 이 메서드가 반환하는 객체 타입은 Comparator
 - 한 번의 객체 생성 : ReverseComparator.REVERSE_ORDER 싱글턴 인스턴스로 한 번만 생성
 - reverseOrder 에 요소가 없으므로 T에 어떤 타입 요청이 와도 비검사 형변환해도 타입 안전 보장

### 재귀적 타입 한정
> 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정한다.  
> 타입 매개변수 자체가 해당 클래스나 인터페이스를 참조하는 형태로 정의 된다.  
~~~java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    ...
}
~~~

- E의 정의가 재귀적으로 자신을 포함하므로 재귀적 타입 한정이라고 한다.
- <E extends Comparable<E>> 을 사용하면 Comparable 의 하위 구현체인 타입만 올수 있다는 것을 명시 
- 즉, 모든 타입 E는 자기 자신과 비교할 수 있다. 


## 결론 
형 변환을 해줘야 하는 메서드는 제네릭하게 만들어 안전하고, 사용하기 쉽게 만들자.