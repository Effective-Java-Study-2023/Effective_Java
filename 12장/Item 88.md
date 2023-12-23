# 아이템 88. readObject 메서드는 방어적으로 작성하라.

### 방어적 복사를 사용하는 불변 클래스 

~~~java
// 아이템 50 에서 불변식을 지키고 불변을 유지한 날짜 클래스를 만들기 위해 생성자와 접근자에서 Date 객체를 방어적으로 복사한 예시 
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime()); // 가변인 Date 클래스의 위험을 막기 위해 새로운 객체로 방어적 복사
        this.end = new Date(end.getTime());

        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + " after " + end);
        }
    }

    public Date start() { return new Date(start.getTime()); }
    public Date end() { return new Date(end.getTime()); }
    public String toString() { return start + " - " + end; }
    // ... 나머지 코드 생략
}
~~~

### 위 클래스를 직렬화한다면?
- 클래스 선언에 implements Serializable 만 추가한다면 불변식을 더는 보장하지 못한다. 
- readObject 메서드가 또 다른 public 생성자이기 때문이다.
  - readObject 에서도 인수가 유효한지 검사하고, 매개변수를 방어적으로 복사해야 한다. 
  - readObject 메서드를 구현하는 것은 직렬화된 데이터를 역직렬화할 때 발생할 수 있는 보안 및 불변셩을 다루기 위해서이다.

<details><summary>역직렬화를 이렇게 한다면?</summary>

~~~java
public class BogusPeriod {
  // 불변식을 깨뜨리도록 조작된 바이트 스트림
  private static final byte[] serializedForm = {
          (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
          0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
          0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
          0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
          0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
          0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
          0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
          0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
          0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
          (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
          0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
          0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
          0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
          0x00, 0x78
  };

  public static void main(String[] args) {
    Period p = (Period) deserialize(serializedForm);
    System.out.println(p);
  }

  // 주어진 직렬화 형태로부터 객체를 만들어 반환한다. 
  static Object deserialize(byte[] sf) {
    try {
      return new ObjectInputStream(
              new ByteArrayInputStream(sf)).readObject();
    } catch (IOException | ClassNotFoundException exception) {
      throw new IllegalArgumentException(exception);
    }
  }
}
// 바이트 스트림의 정보는 start 시간이 end 시간보다 느리게 조작했다.
// 즉, 불변식을 깨뜨린 객체로 역직렬화하도록 조작
~~~
</details>

<details><summary>readObject</summary>
<div>

1. 객체의 필드 읽기 : 객체의 필드 값을 읽어 객체를 복원. 이 때 필드들은 직렬화된 바이트 스트림에서 읽힌 값으로 설정
2. 커스텀 역직렬화 구현 : 이 메서드를 이용하여 기본 역직렬화 동작으로 재정의할 수 있다. 추가적 로직 등을 수행 가능
3. 객체 유효성 검사 : 역직렬화가 완료된 후에는 객체의 유효성 검사하는 콜백을 실행할 수 있다. (내장되어 있음)
</div>
</details>


### 해결
- Period 의 readObject 메서드가 defaultReadObject 를 호출한 다음 역직렬화된 객체가 유효한지 검사한다.
- 즉, 역직렬화 직후 불변식을 만족하는지 확인한다. 
- 허용되지 않은 Period 인스턴스 생성을 막을 수 있다. 
- 그러나 완전히 안전하다고 할 수 없다. 
~~~java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject(); // 기본 직렬화를 수행한다.
    if (start.compareTo(end) > 0) { // 유효성 검사를 수행한다.
        throw new InvalidObjectException(start + " 가 " + end + " 보다 늦다.");
    }
}
~~~
- private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다. 

<details><summary>가변 공격의 예</summary>

~~~java
public class MutablePeriod {
	// Period 인스턴스
	public final Period period;

	// 시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
	public final Date start;

	// 종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
	public final Date end;

	public MutablePeriod() {
		try {
			ByteArrayOutputStream bos = new ByteArrayOutputStream();
			ObjectOutputStream out = new ObjectOutputStream(bos);

			// 유효한 Period 인스턴스를 직렬화한다.
			out.writeObject(new Period(new Date(), new Date()));

			/*
			 * 악의적인 '이중' 인스턴스를 만든다. (이전 객체 참조, 자바 객체 직렬화 명세 6.4절)
			 * 직렬화된 바이트 스트림을 역직렬화하여 Period 인스턴스를 만든다.
			 * 그리고 내부의 Date 필드를 훔쳐낸다.
			 */
			byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
			bos.write(ref); // 시작 필드
			ref[4] = 4; // 참조 #4
			bos.write(ref); // 종료 필드

			// Period의 내부 Date 필드들을 훔쳐낸다.
			ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
			period = (Period) in.readObject();
			start = (Date) in.readObject();
			end = (Date) in.readObject();
		} catch (IOException | ClassNotFoundException e) {
			throw new AssertionError(e);
		}
	}

	
	// 공격 시작
	public static void main(String[] args) {
		MutablePeriod mp = new MutablePeriod();
		Period p = mp.period;
		Date pEnd = mp.end;

		// 내부의 Date 필드를 훔쳐낸다.
		pEnd.setYear(78);
		System.out.println(p);

		// 내부의 Date 필드를 훔쳐낸다.
		pEnd.setYear(69);
		System.out.println(p);
	}
}
~~~
- 정상 Period 인스턴스의 바이트스트림 끝에 private Date 필드로의 참조를 추가해 가변 Period 인스턴스를 만들 수 있다. 
- 객체를 역직렬화할 때는 클라이언트가 소유해서 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다. 
- 즉 readObject 에서는 불변클래스 내부의 모든 private 가변 요소를 방어적으로 복사해야 한다. 
</details>

### 방어적 복사와 유효성 검사를 수행하는 readObject 메서드 
~~~java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 가변 요소를 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    // 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
~~~
- 방어적 복사를 유효성 검사보다 앞에 둔다. 
- final 필드는 방어적 복사가 불가능하므로, 한정자를 제거한다. 

### final 이 아닌 직렬화 가능 클래스의 경우 
> 생성자와 마찬가지로, readObject 도 재정의 가능 메서드를 호출해서는 안된다. 
> 해당 메서드가 재정의되면, 하위클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스의 재정의된 메서드가 실행되어 프로그램 오작동된다. 


### 기본 readObject 메서드를 사용가능한지 판단 방법
- (transient 필드 제외한) 모든 필드 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 되는가?
  - NO 라면...
  - 커스텀 readObject 메서드를 만들어 모든 유효성 검사와 방어적 복사를 수행해야 한다. 
  - 또는, 직렬화 프록시 패턴을 사용하는 방법도 있다. 


## 결론
### 안전한 readObject 메서드를 작성하는 지침
1. private 이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하자. 
2. 모든 불변식을 검사하여 어긋나는게 발견되면 InvalidObjectException 을 던진다. 
3. 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하자. 
4. 직접적이든 간접적이든 재정의할 수 있는 메서드는 호출하지 말자. 

------
<details><summary>정리</summary>
<div>

### 직렬화

1. 직렬화란 무언인가 
- 자바 직렬화는 외부 다른 자바 시스템에서 사용할 수 있도록 자바 객체나 데이터를 바이트 형태로 변환하는 기술을 말한다.
- 직렬화란 간단하게 포맷을 변환하는 기술이라고 할 수 있다. 

2. 자바 직렬화 사용
- 객체가 직렬화 가능한 클래스의 인스턴스여야 한다. 
- 직렬화 가능한 클래스로 만드는 방법은 'java.io.Serializable' 인터페이스를 구현(implements) 하도록 선언하면 된다. 
- 또는 직접 구현하지 않고, 직렬화 가능한 클래스를 상속하면 직렬화 가능한 클래스가 된다. 
- 객체를 직렬화할 때 특정 멤버 필드를 제외하고 싶다면 멤버 변수에 'transient' 키워드를 선언하면 된다. 

### 역직렬화
1. 역직렬화 
- 직렬화된 데이터를 다시 객체로 만드는 것을 역직렬화

2. 역직렬화 사용 
- 역직렬화는 ObjectInputStream
- 역직렬화를 할 때는 직렬화된 객체의 클래스가 반드시 클래스 경로에 존재햐아하며, import 된 상태여야 한다. 


### readObject 
- 자바 직렬화 또는 역직렬화 과정에서 별도의 처리가 필요할 때 클래스 내부에 선언해준다.  
- 직렬화 과정에서는 wirteObject, 역직렬화 과정에서는 readObject 메서드가 자동으로 호출된다.  
- ObjectInputStream 클래스는 데이터를 바이트 스르림에서 읽어와 객체로 역직렬화하는데 사용된다.  
- readObjec() 메서드는 이러한 역직렬화 프로세스를 수행하고, 해당 바이트 스트림에서 읽은 데이터를 객체로 변환한다.  
- Object 클래스의 참조를 반환하므로, 역직렬화된 객체 사용시 적절한 형변환을 해줘야 한다. 
- 반드시 priavte 으로 선언하여 재정의할 수 없게 만든다. 

</div>
</details>

[자바 객체 직렬화 명세 6.4](https://docs.oracle.com/en/java/javase/11/docs/specs/serialization/protocol.html#grammar-for-the-stream-format) 

<details><summary>직렬화 프록시</summary>

### 직렬화 프록시 패턴
- 디자인 패턴의 프록시 패턴을 응용한 것과 같다. 
- 실제 객체를 숨기고 대변인을 통해 직렬화/역직렬화를 수행한다. 
- 바깥 클래스의 논리적 상태를 표현하는 중첩 클래스를 설계해 private static 으로 선언한다. 
- 이 중청 클래스가 바깥 클래스의 직렬화 프록시다. 
- 중첩 클래스의 생성자는 단 하나여야 하고, 바깥 클래스를 매개변수로 받아야 한다. 
- 이 생성자는 단순히 인수로 넘어온 인스턴스의 데이터를 복사한다. 
- 일관성 검사나 방어적복사는 필요 없다. 
- 바깥 클래스, 직렬화 프록시 모두 Serializable 을 구현한다고 선언 해야한다. 
</details>