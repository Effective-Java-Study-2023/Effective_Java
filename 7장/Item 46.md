# 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라

스트림은 함수형 프로그래밍에 기초한 패러다임. 스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성. 이때 각 단계는 가능한 한 이전 단계의 결과를 받아 처리 하는 순수 함수여야 한다. 순수 함수란 입력만이 결과에 영향을 주는 함수이다.

다른 가변 상태를 참조하지도 않고, 함수 스스로도 다른 상태를 변경하지 않는다. 이렇게 하려면 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야한다.
~~~Java
public class Freq {

    public static void main(String[] args) throws FileNotFoundException {
        File file = new File("./words.txt");
        Map<String, Long> freq = new HashMap<>();
        try (Stream<String> words = new Scanner(file).tokens()) {
            words.forEach(word -> {
                freq.merge(word.toLowerCase(), 1L, Long::sum);
            });
        }
        System.out.println(freq);
    }

}
~~~
해당 코드는 텍스트 파일에서 단어별 수를 세어 map에 저장하고 있는 코드이다.
메서드 참조도 사용했고, 결과도 올바르지만 이는 스트림 코드라 할 수 없다.
스트림 API의 이점을 살리지 못하여 같은 기능의 반복 코드 보다 길고, 읽기 어렵고 유지보수 에도 좋지 않다.
모든 연산이 종단 연산인 forEach에서 일어나는데 이때 외부 상태(freq)를 수정하는 람다를 실행하면서 문제가 생긴다.

forEach 연산은 스트림 결과를 보여주는일 일 이상을하면 안된다.
forEach에서 계산하는건 스트림 스럽지 않다.
아래 코드가 올바르게 작성한 모습이다.
~~~Java
public class Freq {

    public static void main(String[] args) throws FileNotFoundException {
        Map<String, Long> freq2 = new HashMap<>();
        try (Stream<String> words = new Scanner(file).tokens()) {
            freq2 = words.collect(groupingBy(String::toLowerCase, counting()));
        }
    }

}
~~~

## 다양한 스트림 연산
### Collector
collector는 스트림을 사용하려면 꼭 알아야하는 개념. java.util.stream.Collectors 클래스는 메서드를 39개나 가지고 있다.
### 예시 (2개만)
1. toList
~~~Java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
~~~
2. toMap
~~~Java
Map<String, Operation> collect = Stream.of(Operation.values())
        .collect(toMap(Object::toString, e -> e));
~~~
## 정리
스트림을 올바르게 사용하기 위해서는 수집기를 잘 알아야하고, 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining 등이 있다.