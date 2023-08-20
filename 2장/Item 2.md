# 아이템2. 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩토리와 생성자에는 똑같은 단점이 존재하는데, 선택적 매개변수가 많을 경우 적절히 대응하기가 어렵다.

1. 점층적 생성자 패턴  
   "필수 매개변수"만 받는 생성자부터, "필수 매개변수 + 선택 매개변수" 까지 생성자를 늘려가는 방식이다.
    <details>
    <summary>예시 코드</summary>

   ```java
        public class NutritionFacts {
        
            private final int servingSize;
            private final int servings;
            private final int calories;
            private final int fat;
            private final int sodium;
            private final int carbohydrate;
            
            public NutritionFacts(int servingSize, int servings) {
            this(servingSize, servings, 0);
            }
            
            public NutritionFacts(int servingSize, int servings, int calories) {
            this(servingSize, servings, calories, 0);
            }
            
            public NutritionFacts(int servingSize, int servings, int calories, int fat) {
            this(servingSize, servings, calories, fat, 0);
            }
            
            
            public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
            this(servingSize, servings, calories, fat, sodium, 0);
            }
            
            public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium,
            int carbohydrate) {
            this.servingSize = servingSize;
            this.servings = servings;
            this.calories = calories;
            this.fat = fat;
            this.sodium = sodium;
            this.carbohydrate = carbohydrate;
            }
        }
   ```

    ```
   // 사용
    NutritionFacts nutritionFacts = new NutritionFacts (20, 10, 32,45);
   ```
    </details>

    + 특징
        + 사용자가 설정을 원치 않는 매개변수까지 포함해야하는 경우가 발생할 수 있다.
        + 매개 변수의 순서를 바꿔 보낼 경우 런타임 에러가 나지 않아, 에러 찾기가 매우 어렵다.


2. 자바 빈즈 패턴  
   필드와 메서드를 각각 작성하고, "매개변수가 없는 생성자로 객체" 를 만든 후 "세터 메서드"를 호출하여 매개변수의 값을 설정하는 방식이다.
   <details>
   <summary>예시 코드</summary>
   <div>

   ```java
    public class User {
        private String name;
        private int age;

        public User() {
        }

        public void setName(String name) {
            this.name = name;
        }

        public void setAge(int age) {
            this.age = age;
        }
    }
    ```
   ```java
    public class Example02 {

        User user = new User();
        user.setName("hee");

    }
    ```
   만약 user는 반드시 name 과 age 를 모두 가져야 하는데, name 만 설정한 경우 user 객체는 일관성이 무너진 상태가 된다.
   이때, 수십개를 설정한다면? 큰 오류를 범할 수 있다.
   </div>
   </details>

    + 특징
        + 객체 하나를 만들기 위해서는 여러 개의 메서드를 호출해야 한다.
        + 객체가 완전히 생성되기 전까지 일관성을 보장 받지 못한다.
        + 점층적 생성자 패턴보다는 읽기 쉽니다.


3. 빌더 패턴 (점층적 생성자 패턴의 안전성 + 자바 빈즈 패턴의 가독성)  
   필수 매개변수만으로 생성자 혹은 정적 팩토리를 호출하여 빌더 객체를 얻은 다음, 빌더 객체가 제공하는 세터 메서드들로 원하는 선택 매개변수들을 설정한다.
    <details>
    <summary>예시 코드</summary>
   <div>

    ```java
        public class User {
          private final String name;
          private final int age;
          private final int tall;
          private final String address;

          public static class Builder {
              // 필수
              private final String name;
              private final int age;

              //선택 : 기본 값으로 초기화
              private int tall = 0;
              private String address = "경기도";

              public Builder(String name, int age) {
                  this.name = name;
                  this.age = age;
              }
              public Builder tall(int value) {
                  tall = value;
                  return this;
              }

              public Builder address(String value) {
                  address = value;
                  return this;
              }

              public User build() {
                  return new User(this);
              }
          }

          private User(Builder builder) {
              name = builder.name;
              age = builder.age;
              tall = builder.tall;
              address = builder.address;
          }
      }
   ```
   ```java
      // 사용 
      	User user = new User.Builder("hee", 14)  // 필수
                                     .tall(140)  // 선택
                                     .build();
    ```  
   User는 private final 로 인해 불변 객체가 되었다.
   </div>
   </details>

    + 특징
        + 빌더는 생성할 클래스 안에 주로 정적 멤버 클래스를 만든다.
        + 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓸 대 빛을 발한다 :)


+ 객체를 만들기 위해서는 빌더부터 만들어야한다.
+ 빌더 패턴은 매개변수가 4개 이상은 되어야 쓸만하다.(그러나 종종 API는 시간이 지날 수록 매개변수가 많아진다고 한다. ㅎㅎ)

---
** 여기서 잠깐! effective java 의 예시 코드에 제네릭에 대해 다루는데, 제네릭에 대해 잠깐 보고 갑시다.
<details>
<summary>*제네릭*</summary>
제네릭은 다양한 타입의 객첵들을 다루는 메서드나 컬렉션 클래스에 컴파일 시의 타입을 체크해주는 기능이다. 

장점
1. 타입 안정성을 제공한다.
    + 타입 안정성을 높인다는 것은 의도하지 않은 타입의 객체를 저장하는 것을 막고, 저장된 객체를 꺼내올 때 원래의 타입과 다른 타입으로 형변환 되어 발생하는 오류 줄여준다는 의미
2. 타입 체크와 형변환을 생략할 수 있어 코드가 간결해 진다.

예시)
   ```java
// Tv 객체만 저장할 수 있는 ArrayList 생성
   ArrayList<Tv> tvList = new ArrayList<>();
   tvList.add(new Tv()) // 가능 
   tvList.add(new Audio()) // 컴파일 에러
   ```

```java
class Box<T>{}
```
* Box<T> 제네릭 클래스 T의 Box or T Box 라고 읽는다.
* Box 는 원시 타입
* T는 타입 변수 또는 타입 매개변수라고 한다.
* 타입 변수는 여러개 가능하다 예) Map<K, V>


이렇게 클래스의 타입을 컴파일 시점에 정하여 타입 예외에 대한 안정성을 확보할 수 있지만, 너무 자유룝다는 것이 단점이 될 수 있다.
예로써, 계산기 코드를 짤 때, 제네릭으로 정수, 실수 구분없이 받을 수 있게 만들었지만, 부작용으로 String도 받을 수 있을 것이다.  
=> 따라서 의도와는 다른 자료형이 들어올 수 없게 만드는 것이 "제한된 타입 매개변수" 이다.
### 제한된 타입 매개변수
```java
class Calculator<T> {
   void add(T a, T b){}
   void min(T a, T b){}
   void mul(T a, T b){}
   void div(T a, T b){}
}

// 아래와 같이 아무 타입이나 모두 할당이 가능하다. 
public class Main {
   public static void main(String[] args) {
      Calcultor<Number> cal1 = new Calculator<>();
      Calcultor<Object> cal2 = new Calculator<>();
      Calcultor<String> cal3 = new Calculator<>();
   }
}
```
* 타입 한정 키워드 extends를 사용하여 제네릭을 number 클래스 와 그 하위 타입만 받도록 범위를 제한할 수 있다.
```java
class Calculator<T extends Number> {
   void add(T a, T b){}
   void min(T a, T b){}
   void mul(T a, T b){}
   void div(T a, T b){}
}

// 아래와 같이 아무 타입이나 모두 할당이 가능하다. 
public class Main {
   public static void main(String[] args) {
      Calcultor<Number> cal1 = new Calculator<>();
      Calcultor<Integer> cal2 = new Calculator<>();
      Calcultor<Double> cal3 = new Calculator<>();
      
      // extends 안에 들지 못한 클래스는 오류가 난다. 
      Calcultor<Number> cal4 = new Calculator<>();
      Calcultor<Object> cal5 = new Calculator<>();
      Calcultor<String> cal6 = new Calculator<>();
   }
}
```
### 재귀적 타입 한정
이와 비슷하지만, 조금 다른 "재귀적 타입 한정" 이란 것도 있다.
재귀적 타입 한정이란, 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정시키는 것을 말한다.  
주로 빌더 패턴에서 하위 클래스가 상위 클래스의 메서드를 오버라이드하면서 해당 하위 클래스의 타입을 반환할 필요가 있을 때 사용한다.  
타입 매개변수 자체가 해당 클래스나 인터페이스를 참조하는 형태로 정의 된다.


* 간단하게 말하면, "제한된 타입 매개변수" 는 "이 변수는 extends 뒤 타입 또는 그 하위 타입만 허용한다" 이며, "재귀적 타입 한정" 은 "이 변수는 이 클래스의 파생된 타입이어야 한다" 라는 조건을 설정하는 것이다.

</details>

<details>
<summary>이펙티브 자바 예시 코드</summary>

```java
public abstract class Pizza {
   public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

   final Set<Topping> toppings;

   // 재귀적 타입 한정을 이용한 추상 빌더 클래스 
   // T 가 Builder<T> 의 하위 타입이어야 한다. 
   // 즉 Builder<T>를 상속 받거나 자기 자신의 타입이어야 한다. 
   abstract static class Builder<T extends Builder<T>> { 
	   // enumset 초기화
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

	  // 토핑을 추가하는 메서드, 반환 타입은 T 이며, 빌더의 하위 타입이나 빌더 자신이 된다. 
      public T addTopping(Topping topping) {
         toppings.add(Objects.requireNonNull(topping));
         return self();
      }

	  // 피자 객체를 생성하고, 반환하는 추상메서드로써, 하위 클래스에서 구체적으로 구현된다. 
      abstract Pizza build();
	  
	  // 현재 빌더 객체를 반환하는 추상메서드를 정의
      protected abstract T self();

   }
   
   // pizza의 생성자로 어떤 타입의 Builder 객체도 받아 들일 수 있다. 
   Pizza(Builder<?> builder) {
	   //빌더에서 피자 객체의 토핑 집합을 복제하여 초기화 
      toppings = builder.toppings.clone();
   }

}

```

```java
// NyPizza 클래스를 정의하며 Pizza 를 상속 받는다. 
public class NyPizza extends Pizza{
	public enum Size{SMALL, MEDIUM, LARGE}
	private final Size size;

	// NyPizza에 대한 빌더를 정의한다. Pizza.Builder 를 상속 받는데, 제네릭 타입으로 Builder 자기 자신을 지정하고 있다.
	public static class Builder extends Pizza.Builder<Builder> {

		private final Size size;

		// 빌더의 생성자를 정의하고, 피자의 크기를 받는다. 
		public Builder(Size size) {
			// Null 이 아닌 크기를 변수에 할당
			this.size = Objects.requireNonNull(size);
		}

		@Override
        // 피자 객체를 생성하고, 반환하는 메서드를 구체적으로 구현
		public NyPizza build(){
			return new NyPizza(this);
		}

		@Override
        //현재 빌더 객체를 반환하는 메서드를 구체적으로 구현
		protected Builder self(){
			return this;
		}
	}

	// NyPizza의 private 생성자이다. 빌더 객체를 통해서만 NyPizza 객체를 생성할 수 있다. 
	private NyPizza(Builder builder) {
		super(builder);
		size = builder.size;
	}
}
```  
```java
public class PizzaChoice {
	// 빌더를 설정한 후 아래와 같이 사용 가능 
	NyPizza pizza = new NyPizza.Builder(Size.SMALL)
		.addTopping(Topping.SAUSAGE).addTopping(Topping.ONION).build();
}
```

* "Pizza" 의 "Builder" 클래스에서 T의 구체적인 타입은 Builder 클래스나 그 하위 클래스를 참조해야한다.
* 여기서 "NyPizza.Builder" 는 "Pizza.Builder<Builder>" 를 확장하므로, "NyPizza.Builder" 는 "Pizza.Builder<T>" 의 하위 타입이다.
* T는 "NyPizza.Builder"로 구체화 된다.
</details>

* 하위 타입이란 특정 클래스나 인터페이스를 상속받거나 구현한 다른 클래스를 말한다. 
---
* 스프링에서는 lombok 의 @Builder 를 이용하여 빌더 패턴을 쉽게 사용할 수 있다. 
* lombok 의 @Builder 를 클래스에 달아주면 @AllArgsConstructor 도 같이 쓰는 것과 같으므로 바람직 하지 않다. 
* 가급적 직접 만든 생성자에 달아주도록 하자. 
* <details><summary>설명</summary>
   <div>
   Finally, applying @Builder to a class is as if you added @AllArgsConstructor(access = AccessLevel.PACKAGE) to the class and applied the @Builder annotation to this all-args-constructor. This only works if you haven't written any explicit constructors yourself. If you do have an explicit constructor, put the @Builder annotation on the constructor instead of on the class. Note that if you put both @Value and @Builder on a class, the package-private constructor that @Builder wants to generate 'wins' and suppresses the constructor that @Value wants to make.  
  
   마지막으로, 크래스에 @Builder 를 적용하는 것은 클래스에 @AllArgsConstructor 를 추가하고 @Builder 를 적용한 것과 같다. 명시적 생성자를 직접 작성하지 않은 경우에만 작동합니다.명시적 생성자가 있는 경우 클래스 대신 생성자에 @Builder 주석을 추가합니다. @Value 와 @Builder 를 모두 클래스에 넣으면 @Builder 가 생성하려는 패키지 전용 생성자가 "승리"하고 @Value 가 만들고자하는 생성자를 억제한다. 
   </div>
   </details>

* 빌더에 대한 설명(lombok 문서) : https://projectlombok.org/features/Builder