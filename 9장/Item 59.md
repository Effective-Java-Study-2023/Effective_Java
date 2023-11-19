# Item 59 - 라이브러리를 익히고 사용하라

<hr>

## 무작위 정수를 생성한다고 했을 때

<hr>

```Java
static Random rnd = new Random();

static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```

괜찮아 보이지만 3가지의 문제를 내포하고 있다.

- n 이 그렇게 크지 않은 2의 제곱수라면 같은 수열이 반복
- n 이 2의 제곱수가 아니라면 몇몇 숫자가 더 자주 반환

```Java
public static void main(String[] args) {
        int n = 2 * (Integer.MAX_VALUE / 3);
        int low = 0;
        for(int i = 0; i < 1000000; i++) {
        if(random(n) < n/2) {
        low++;
        }
        System.out.println(low);
        }
        }
```

    random 메서드가 이상적으로 동작한다면 약 50만개가 출력되어야 하지만, 실제로는 666,666에 가까운 값을 얻는다.
    무작위로 생성된 수 중에서 2/3 가량이 중간값보다 낮은 쪽으로 쏠린 것이다.

- 지정한 범위 '바깥' 의 수를 종종 반환
  - n 이 2의 제곱수가 아닐 때 random.nextInt 가 Integer.MIN_VALUE 를 종종 반환하고 abs 는 양수가 아닌 음수를 반환

## 어떻게 해결해야 하는가

<hr>

직접 해결하기 위해서는 의사난수 생성기, 정수론, 2의 보수 계산 등에 조예가 깊어야 한다.

하지만 라이브러리를 사용하면 이를 몰라도 쉽게 해결할 수 있다.

위의 예제에서는 Random.nextInt(int)를 사용하면 된다.

자바 7부터는 Random보다 성능이 좋은 ThreadLocalRandom을 사용하는 것이 좋다.

## 표준 라이브러리를 쓰면서 가질 수 있는 이점

<hr>

1. 그 코드를 작성한 전문가의 지식과 앞서 사용한 다른 프로그래머들의 경험을 활용 가능
2. 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간 허비를 안해도 됨
3. 따로 노력하지 않아도 성능이 지속해서 개선됨
4. 기능이 점점 많아짐
5. 사용한 라이브러리 코드는 다른 개발자가 보았을 때 이해도 쉽고 유지보수도 쉽다.

## InputStream 의 transferTo()

<hr>

```Java
public static void main2(String[] args) throws IOException {
        String url = "http://shouldItestprivateMethods.com";
        try (InputStream inputStream = new URL(url).openStream()) {
            inputStream.transferTo(System.out);
        }
}
```

지정한 url의 내용을 가져오는 명령줄 애플리케이션 코드이다. 
예전에는 작성하기 까다로운 기능이었지만, 자바 9에서 InputStream에 추가된 transferTo 메서드를 사용하면 쉽게 구할 수 있다.

라이브러리가 너무 방대하여 모든 API문서를 공부하는 것은 벅차겠지만, 

**자바 프로그래머라면 적어도 java.lang, java.util, java.io 와 그 하위 패키지들에는 익숙해져야 한다.**

    이외에도 컬렉션 프레임워크, 스트림 라이브러리, java.util.concurrent 

## 정리

<hr>

바퀴를 다시 발명하지 말자. 아주 특별한 나만의 기능이 아니라면 누군가 이미 라이브러리 형태로 구현해놓았을 가능성이 크다.

그런 라이브러리가 있다면, 쓰면 된다. 있는지 모르겠다면 찾아보라

일반적으로 라이브러리의 코드는 직접 작성한 것보다 품질이 좋고, 점차 개선될 가능성이 크다.
