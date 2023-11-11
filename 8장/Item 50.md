# 아이템 50. 적시에 방어적 복사본을 만들라 

### 자바는 안전한 언어다. 
- 메모리 충돌 오류에서 안전하다. 
- 자바로 작성한 클래스는 불변식이 지켜진다. 

### 방어적 복사의 이점
1. 불변 객체를 유지할 수 있다. 
2. 보안에 유리하다. 
3. 다중스레드 환경 및 멀티 스레드 환경에서 값 변경을 막을 수 있다. 

### 클래스 내부 수정을 본의아니게 허락할 수 있다.  

예시 : 시작 시간이 종료 시간보다 빨라야 하는 Period 객체
~~~java
public final class Period {
	private final Date start;
	private final Date end;

	public Period(Date start, Date end) {
		if (start.compareTo(end) > 0) {
			throw new IllegalArgumentException(
				start + "가" + end + "보다 늦다.");
		}
		this.start = start;
		this.end = end;
	}
	public Date start() {
		return start;
	}

	public Date end() {
		return end;
	}
}
~~~
- 불변 클래스처럼 보이고, 시작 시각이 종료 시간보다 빨라야 한다는 불변식을 지킬거라 생각할 수 있다. 
- 여기서 문제점 Date 가 가변이다. 

~~~java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78);
~~~
<details><summary>위 코드에 대한 조언?</summary><div>

- Date 인스턴스는 값을 변경할 수 있는 메서드를 가지고 있기 때문에 예기치 못하게 위 클래스의 불변식을 깬다.
- 이를 위해 `Instant` 를 사용하자.
- 또는, `LocalDateTime` 이나 `ZonedDateTime` 을 사용하자.
</div></details>

### 다른 스레드가 원본 객체를 수정할 위험이 존재한다. 
> 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사해야 한다. 

~~~java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    
    if (this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException(
            this.start + "가 " + this.end + "보다 늦다."
        );
    }
}
~~~

매개변수의 유효성 검사전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사하자. 
~~~java
멀티 스레드 환경에서 원본객체의 유효성을 검사한 후 복사본을 만드는 찰나에 
다른 스레드가 원본 객체를 수정할 위험이 있다.
방어적 복사를 매개변수 유효성 검사전에 수행하면 위험에서 해방~! 
~~~

매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone 을 사용하면 안 된다. 
~~~java
Date 는 final 이 아니므로 clone 이 Date 가 정의한 것이 아닐 수 있다. 
- 변경가능한 Date 객체는 멀티스레딩 환경에서 안전하지 않다. 
- clone 메서드의 불확실한 동작으로 인해 예측할 수 없는 결과 초래 
~~~


### 접근자 메서드가 내부의 가변 정보를 직접드러 내면 위험하다. 

~~~java
Date start = new Date();
Date end = new Date();
Period period = new Period(start, end);
period.end().setYear(78); // period 내부 변경!! 
~~~
> 단순히 접근자가 가변 필드의 방어적 복사본만 반환하면 문제 해결!

~~~java 
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}

public int getYear() {
	return year; // 새로운 접근자
}
~~~
- 새로운 접근자까지 작성하면 Period 불변임
- 자신말고는 가변 필드에 접근할 방법이 없고, 모든 필드가 객체 안에 완벽히 캡슐화 됨
- 생성자와 달리 접근자 메서드에서는 방어적 복사에 clone 을 사용해도 됨
  - 인스턴스를 복사하는데 일반적으로 생성자나 정적 팩터리를 쓰는 것이 좋다. 


### 방어적 복사의 필요성
- 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관할 때 그 객체가 언제 변경될 수 있는지 생각해 보기 
  - 확실 못한다면, 방어적 복사본을 만들어 저장하자
- 되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사할 일이 줄어든다(아이템 17)
- 호출자가 내부자를 수정하지 않는다고 확신하면 방어적 복사 생략 가능
  - 단 문서화 할 것 
- 방어적 복사를 생략해도 되는 상황은 클래스-클라이언트가 상호 신뢰 가능할 떄, 불변식이 깨지더라도 영향이 오직 호출한 클라이언트에게만 국한 될 때(래퍼 클래스)

## 결론
- 클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 요소는 반드시 방어적 복사
- 복사 비용이 너무 크거나, 클라이언트가 요소를 수정하지 않는다고 신뢰할 경우, 방어적 복사 수행하는 대신 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시

---
<details>
<summary>깊은 복사, 얕은 복사, 방어적 복사(아이템 17)</summary>
<div>

### 깊은 복사
- 객체의 모든 내부 상태를 완전히 복사하여 새로운 객체를 만드는 방식이다.
- 원본 객체 내부의 모든 객체들도 재귀적으로 복사한다.
- 복사된 객체와 원본 객체는 서로 다른 객체를 참조하고, 하나의 객체를 수정하더라도 다른 객체에 영향을 미치치 않는다.

### 얕은 복사
- 얕은 복사는 원본 객체의 멤버 변수들의 값만을 복사하는 방식이므로, 원본 객체와 복사본 객체의 기본타입의 멤버 변수는 독립적이다.
- 하지만, 참조 타입의 멤버 변수는 객체의 주소값을 복사하여 복사본이 원본과 같은 객체를 참조한다.
- 복사본에서 객체 내부의 값을 변경하면 원본 객체도 영향을 받는다.
- 자바에서 객체의 얕은 복사는 ' = ' 연산자 사용
    - 그럼 언제 사용하는게 좋을까?
        - 깊은 복사보다 속도가 빨라야 할 때
        - 특정 상황에서 내부 데이터나 상태를 공유해야할 때
        - 어떤 객체에 일시적인 작업을 수행할 때

### 방어적 복사
- 복사본이 원본 주소를 그대로 참조하지 않지만, 복사본 객체 내부에 있는 객체들은 원본과 동일한 주소를 참조한다.
- List 는 같은 주소를 참조하지 않지만, List 내 각 요소들은 원본과 동일한 주소를 참조한다.
- 요소를 변경할 수 있다는 의미이다.
    - 왜 사용할까?
        - 객체의 내부 상태를 외부에 직접 노출시키지 않고, 복사본을 제공하여 객체의 불변성을 유지할 때 주로 쓴다.
</div>
</details>
