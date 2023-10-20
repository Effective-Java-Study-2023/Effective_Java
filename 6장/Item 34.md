# 아이템 34. int 상수 대신 열거 타입을 사용하라
## int 상수 패턴
~~~Java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
~~~
경우의 수가 한정될 때 각 경우를 상수 값으로 치환하여 표현하는 것이다.
## int 상수 패턴의 단점
- 타입 안전을 보장할 방법도 없고 , 표현력도 좋지 않다.
 - 오렌지를 보낼 메서드에 사과를 보내고 == 연산자로 비교해도 정상적인 결과가 나올 것이다.
- 앞에 전부 ORANGE , APPLE 등의 접두어를 붙인 이유는 네임스페이스가 없기 때문이다.
- 만일 중간에 값 하나가 빠져서 상수 값이 바뀌면 모두 다시 작성해야 한다.
- 문자열로 출력하기 까다롭다.
## 열거 타입(Enum Type)
~~~Java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum ORANGE { NAVEL, TEMPLE, BLOOD }
~~~
- 상수 하나당 인스턴스를 만들어 public static final 필드로 공개하는 것이다.
  - 인스턴스가 통제된다.
  - 원소가 하나이면 싱글턴으로 볼 수 있다.
- 컴파일 타입 안정성을 제공한다.
  - 이전의 int 상수 패턴처럼 ORANGE가 갈 곳에 APPLE이 간다면 , 명확히 타입 에러가 발생한다.
- 네임스페이스를 제공하여 , 이름이 같은 상수도 평화롭게 공존할 수 있다.
  - APPLE.RED와 ORANGE.RED는 구분된다.
- toString()이 출력하기에 적합한 문자열을 내어준다.
- 열거 타입에는 다양한 메서드나 필드도 추가가 가능하다.
  - 추가로 임의의 인터페이스도 구현하게 할 수 있다.
## 열거 타입 예시
~~~Java
enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass; // 질량 (단위: 킬로그램)
    private final double radius; // 반지름 (단위: 미터)
    private final double surfaceGravity; // 표면중력 (단위: m / s^2)

    // 중력상수 (단위: m^3 / kg s^2)
    private static final double G = 6.677300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
~~~
- 열거타입 클래스는 내부에서 특정 필드에 값을 할당하고 싶다면 , 열거 선언 후 바로 생성자를 통해 할당하면 된다.
  - 열거 타입은 근본적으로 불변이라 모든 필드는 final이어야 한다.
## enum.values()
~~~Java
@Test
public void useValueOfEnumTest() {
    double earthWeight = Double.parseDouble("185");
    double mass = earthWeight / Planet.EARTH.surfaceGravity;

    for (Planet value : Planet.values()) {
        System.out.printf("%s에서의 무게는 %f이다.%n", value, value.surfaceWeight(mass));
    }
}
~~~
- enum.values() 메서드는 해당 enum 네임스페이스에 선언된 모든값을 배열의 형태로 반환한다. 
### 출력값
~~~Java
MERCURY에서의 무게는 69.912739이다.
VENUS에서의 무게는 167.434436이다.
EARTH에서의 무게는 185.000000이다.
MARS에서의 무게는 70.226739이다.
JUPITER에서의 무게는 467.990696이다.
SATURN에서의 무게는 197.120111이다.
URANUS에서의 무게는 167.398264이다.
NEPTUNE에서의 무게는 210.208751이다.
~~~
## 열거타입 상수마다 동작이 달라지는 메서드 구성
~~~Java
enum Operation {
    PLUS {
        @Override
        public double apply(double x, double y) {
            return x+y;
        }
    },
    MINUS {
        @Override
        public double apply(double x, double y) {
            return x-y;
        }
    },
    TIMES {
        @Override
        public double apply(double x, double y) {
            return x*y;
        }
    },
    DIVIDE {
        @Override
        public double apply(double x, double y) {
            return x/y;
        }
    };

    public abstract double apply(double x, double y);
}

@Test
public void operationApplyTest() {
    double x = 10;
    double y = 15;

    for (Operation value : Operation.values()) {
        System.out.printf("%f %s %f = %f%n", x, value, y, value.apply(x, y));
    }
}
~~~
- 상수별 클래스 몸체에 apply 메서드를 재정의하였다.
- 위와 같이 추상 메서드를 사용하면 , case문을 사용하지 않고 반환값을 달라지게 할 수 있다.
- 실수로 재정의하지 않았다면 컴파일 오류로 알려준다.

열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성된다.
열거 타입의 toString 메서드를 재정의했다면, toString 이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 고려해볼 수 있다.
~~~Java
private static final Map<String, Operation> stringToEnum =
		Stream.of(values()).collect(Collectors.toMap(Object::toString, e -> e));

/*
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(Collectors.toMap(new Function<Operation, String>() {
        @Override
        public String apply(Operation o) {
            return o.toString();
        }
    }, new Function<Operation, Operation>() {
        @Override
        public Operation apply(Operation o) {
            return o;
        }
    }));
*/
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
~~~
그런데, 이런 상수별 메서드에도 단점은 있다. 열거 타입 상수끼리 코드를 공유하기가 어려운 점.
~~~Java
enum PayrollDay {
	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
	SATURDAY, SUNDAY;

	private static final int MINS_PER_SHIFT = 8 * 60; 

	int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

		int overtimePay;
		switch(this) {
			case SATURDAY: case SUNDAY: // 주말
				overtimePay = basePay / 2;
				break;
			default: // 주중
				if (minutesWorked <= MINS_PER_SHIFT) {
					overtimePay = 0	;
				} else {
					overtimePay = (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
				}
		}
		return basePay + overtimePay;
	}
}
~~~
- switch문을 이용한 형태는 이전과 같이 case를 반드시 같이 추가해주어야 한다.
  - 동작은 하지만, 관리 관점에서 위험한 포인트가 있는 코드이다.
- 가능한 해결책
  - 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣는다.
  - 계산 코드를 평일용과 주말용으로 나눈다.
  - 위 두 방법은 모두 가독성이 크게 떨어진다.
## 전략 열거 타입 패턴
안전성과 유연함을 고려한다면 전략 열거 타입 패턴을 고려해볼 수 있다. 
switch 문이나 상수별 메서드 구현이 필요 없어진다. 새로운 상수를 추가할 때마다 잔업수당 전략을 선택하도록 하는 것이다.
~~~Java
enum PayrollDay {
	MONDAY(), TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
	SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

	private final PayType payType;

	PayrollDay() {
		this(PayType.WEEKDAY);
	}

	PayrollDay(PayType payType) {
		this.payType = payType;
	}

	enum PayType {
		WEEKDAY {
			int overtimePay(int minsWorked, int payRate) {
				int overtimePay;
				if (minsWorked <= MINS_PER_SHIFT) {
					overtimePay = 0;
				} else {
					overtimePay = (minsWorked - MINS_PER_SHIFT) * payRate / 2;
				}
				return overtimePay;
			}
		},
		WEEKEND {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked * payRate / 2;
			}
		};

		abstract int overtimePay(int mins, int payRate);

		private static final int MINS_PER_SHIFT = 8 * 60; 

		int pay(int minutesWorked, int payRate) {
			int basePay = minutesWorked * payRate;
			return basePay + overtimePay(minutesWorked, payRate);
		}
	}
}
~~~

## 정리
1. 필요한 원소를 컴파일 타임에 알 수 있는 상수 집합이라면 열거 타입을 사용하자. 
2. 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.