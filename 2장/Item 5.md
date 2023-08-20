# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
많은 클래들은 하나 이상의 자원에 의존한다. 이런 클래스를 정적 유틸리티 클래스로
구현한 모습을 드물지 않게 볼 수 있다.

``` java
public class SpellChecker{
    private static final Lexicon dictionary = ...;
    
    private SpellChecker() {} // 객체 생성 방지
    
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String type) { ... }
}
```

싱글턴으로 사용한 경우도 드물다.

``` java
public class SpellChecker{
    private final Lexicon dictionary = ...;
    
    private SpellChecker(...) {}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String type) { ... }
}
```
위와 같은 방식은 사전을 단 한나만 사용이 가능하다. 만약 다른 나라의 언어사전이
필요하다면 코드에 수정이 불가피하다.

SpellChecker를 여러 나라의 사전으로 사용할 수 있도록 해보자.

``` java
public class SpellChecker{
    private Lexicon dictionary = ...;
    
    public SpellChecker(...) {}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String type) { ... }
    
    public void setDictionary(Lexicon dictionary){
        this.dictionary = dictionary;
    } // dictionary 변경 메서드
}
```

이렇게 사전을 변경해주는 메서드를 만들어서 다른 사전으로 교체가 가능하지만 ,
멀티스레드 환경에서 사용이 불가능하다. 
만약 스레드1이 한국 사전을 쓰는중에 쓰레드2가 일본 사전으로 
바꿔버리는 경우가 생길 수 있기 때문이다.

그렇기 때문에 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나
싱글턴 방식이 적합하지 않다.

클래스가 여러 자원 인스턴스를 지원해야 하고 , 클라이언트가 원하는 자원을 사용하려면 
생성자 의존객체 주입 방식을 사용해야 한다.

``` java
public class SpellChecker{
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary){
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String type) { ... }
}
```
위와 같이 의존 객체 주입방식을 사용하면 여러 클라이언트가 의존
객체들을 안심하고 공유할 수 있다. 또한 다른 사전으로의 교체가 가능하다.
또한 단위 테스트에 용이하다. 만약 나는 SpellChecker의 동작만 잘 돌아가는지 테스트를
 하고 싶다면 생성자 주입시에 Mock 객체를 주입시켜 사용이 가능하다.
``` java
public class SpellCheckerTest{
    private Lexicon mockDictionary;
    private SpellChecker spellChecker;
    
    @BeforeEach
    void before(){
        mockDictionary = mock(Lexicon.class);
        spellChecker = new SpellChecker(mockDictionary);        
    }    
    
    @Test
    void test(){
     아무거나 테스트.....
    }
}
```
위와같이 SpellChecker에 대한 단위 테스트를 진행할 수 있게 된다.
의존객체 주입이 유연성과 테스트의 용이성을 개선해 주지만 , 의존성이 엄청 많으면 복잡해진다. 스프링과 같은 의존객체 프레임워크를 사용하면 해결이 된다.(제어의 역전)
