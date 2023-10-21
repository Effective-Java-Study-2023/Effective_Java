# 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라

~~~Java
static class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return "Plant{" +
                "name='" + name + '\'' +
                ", lifeCycle=" + lifeCycle +
                '}';
    }
}

@Test
public void plantsByLifeCycleTest() {
    Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

    List<Plant> garden = new ArrayList<>(List.of(
            new Plant("A", Plant.LifeCycle.ANNUAL),
            new Plant("B", Plant.LifeCycle.PERENNIAL),
            new Plant("C", Plant.LifeCycle.BIENNIAL),
            new Plant("D", Plant.LifeCycle.ANNUAL)
    ));

    for (int i = 0; i < plantsByLifeCycle.length; i++) {
        plantsByLifeCycle[i] = new HashSet<>();
    }

    for (Plant plant : garden) {
        plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
    }

    for (int i = 0; i < plantsByLifeCycle.length; i++) {
        System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
    }
}
~~~
### 설명
1. Set 배열을 생성해 생애주기별로 관리한다. 총 3개의 배열이 만들어질 것이다. 각 배열을 순회하여 빈 HashSet으로 초기화 해준다.
2. plant 들을 배열의 Set에 추가한다. 이때 plant가 가지고있는 LifeCycle 열거타입의 ordinal 값으로 배열의 인덱스를 결정한다. 그 결과 식물의 생애주기 별로 Set에 추가된다.
3. 결과를 출력한다. 열거 타입의 values로 반환되는 열거 타입 상수 배열의 순서는 ordinal 값으로 결정되기 때문에 Set 배열의 각 Set이 의미하는 생애주기는 values의 순서와 같을것이다.

### 문제점
1. 배열은 제네릭과 호환되지 않는다. 따라서 비검사 형변환을 수행해야한다.
2. 사실상 배열은 각 인덱스가 의미하는 바를 알지못하기 때문에 출력 결과에 직접 레이블을 달아야 한다.
3. 정수는 열거 타입과 달리 타입 안전하지 않기 때문에 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다.

## EnumMap
이러한 단점들을 java.util 패키지의 EnumMap 을 사용하여 해결해보자. EnumMap은 열거 타입을 키로 사용하는 Map 구현체이다.

~~~Java
public static void usingEnumMap(List<Plant> garden) {
    Map<LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(LifeCycle.class);
    for (LifeCycle lifeCycle : LifeCycle.values()) {
        plantsByLifeCycle.put(lifeCycle,new HashSet<>());
    }
    for (Plant plant : garden) {
        plantsByLifeCycle.get(plant.lifeCycle).add(plant);
    }
    System.out.println(plantsByLifeCycle);
}
~~~
1. 이전 ordinal을 사용한 코드와 다르게 안전하지 않은 형변환을 사용하지 않는다.
2. 결과를 출력하기 위해 번거롭던 과정도 EnumMap 자체가 toString을 제공하기 때문에 번거롭지 않게되었다.
3. ordinal을 이용한 배열 인덱스를 사용하지 않으니 인덱스를 계산하는 과정에서 오류가 날 가능성이 존재하지 않는다.
4. EnumMap은 그 내부에서 배열을 사용하기 때문에 내부 구현 방식을 안으로 숨겨서 Map의 타입 안정성과 배열의 성능을 모두 얻어냈다.
여기서 EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다.

### 스트림 활용 코드
~~~Java
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle,
        () -> new EnumMap<>(LifeCycle.class), toSet())));
~~~
스트림을 사용하면 EnumMap만 사용했을 때와는 다르게 동작한다.
EnumMap 버전은 언제나 열거타입의 개수만큼 생성되지만 스트림 버전에서는 위처럼 garden에 존재하는 열거타입만 만든다.

### 2차원 EnumMap 사용
다음은 두 가지 상태(Phase)를 전이(Transition)와 매핑하는 예제이다. 
LIQUID에서 SOLID의 전이는 FREEZE가 되고, LIQUID에서 GAS로의 전이는 BOIL이 될것이다.
이번 예제에서도 Phase나 Transition의 상수의 선언 순서를 변경하거나 새로운 Phase 상수를 추가하는 경우에도 문제가 발생할 수 있다.
~~~Java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT,FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
~~~
위의 코드를 EnumMap을 사용해서 수정한다.
~~~Java
import java.util.EnumMap;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

enum Phase {

    SOLID, LIQUID, GAS;

    public enum Transition {
        
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> transitionMap = Stream.of(values())
                .collect(Collectors.groupingBy(t -> t.from, // 바깥 Map의 Key
                        () -> new EnumMap<>(Phase.class), // 바깥 Map의 구현체
                        Collectors.toMap(t -> t.to, // 바깥 Map의 Value(Map으로), 안쪽 Map의 Key
                                t -> t, // 안쪽 Map의 Value
                                (x,y) -> y, // 만약 Key값이 같은게 있으면 기존것을 사용할지 새로운 것을 사용할지
                                () -> new EnumMap<>(Phase.class)))); // 안쪽 Map의 구현체;

        public static Transition from(Phase from, Phase to) {
            return transitionMap.get(from).get(to);
        }
    }
}
~~~
위와 같은 방식을 사용하면 Enum값이 추가되어도 별다른 변경없이 사용이 가능하다.

### 정리
1. 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라. 
2. 다차원 관계는 EnumMap<..., EnumMap<...>> 으로 표현하라.