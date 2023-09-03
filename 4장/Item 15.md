# 아이템15. 클래스와 멤버의 접근 권한을 최소화하라
## 잘 설계된 컴포넌트의 기준
푸드코트와 비슷하다고 생각
- 내부 데이터와 내부 구현 정보를 외부로부터 얼마나 잘 숨겼느냐가 중요
  - 정보은닉, 캡슐화라고 한다.
  - 시스템을 구성하는 컴포넌트를 독립시켜 개발, 테스트, 최적화, 적용, 분석, 수정이 개별적으로 가능해야 함
## 정보 은닉의 장점
  - 컴포넌트간 의존성이 없어서 독립적으로 병렬로 개발 가능하기 때문에, 시스템 개발 속도가 높아진다. (각 코너별 음식 만들기)
  - 컴포넌트별 디버깅이 가능하기 때문에, 시스템 관리 비용이 낮아진다. (어떤 코너가 문제가 발생했는지 확인 가능)
  - 컴포넌트별 최적화가 가능하기 때문에, 성능 최적화 하기가 쉽다. (해당 코너만 음식 최적화가 가능)
  - 컴포넌트간 의존성이 없기 때문에, 어디든 넣어 재사용 가능할 가능성이 높음. (다른 백화점에서 해당 코너 그대로 사용가능)
  - 컴포넌트별 테스트가 가능하기 때문에, 큰 시스템의 제작 난이도를 낮춰준다. (해당 코너 독립적 음식 테스트 가능)

  컴포넌트간 의존성을 없애는 것의 핵심은 접근 제한자이다.

## 정보 은닉의 기본 원칙
모든 클래스와 멤버의 접근 제한자를 가능한 가장 낮은 접근 수준으로 유지해야 한다.

## 멤버에 적용 가능한 접근 수준
- private: 멤버를 선언한 톱레벨 클래스에서만 접근 가능
- package-private: 멤버가 소속된 패키지 안의 모든 클래스에서 접근 가능
- protected: package-private이 가지는 접근 범위 + 상속받은 하위 클래스에서도 접근 가능
- public: 모든 곳에서 접근 가능

## 접근 제어자 관리 방법
- 공개할 API를 세심히 설계한 후 그 외의 모든 멤버는 private으로 설정한다.
- 다른 클래스가 접근해야 하는 멤버에 한해 package-private으로 풀어줄 수 있다.
- 권한을 풀어줘야 하는 일이 잦다면, 다른 클래스가 하나 더 만들어져야 하는 것은 아닌지 생각해보자.
- private과 package-private 멤버는 모두 해당 클래스 내부 구현에 해당하므로 공개 API에 영향을 주지 않는다.
- 만약 private, package-private 외에 protected로 풀어준다면, 그 멤버는 API로써 계속 제공되어야 하기 때문에 신중해야 한다.
- API로써 제공된다면 어떤 종류의 요청과 응답을 지원하는지에 대한 명세서를 제공해야 함. 이 명세서는 어떤 종류의 데이터 요청이 가능하며 어떤 종류의 데이터 응답을 받을 수 있는지를 설명해야 함.
  - 이러한 이유로 protected, public의 수는 정말 최소화해야 한다.
- Serialize 인터페이스를 구현하게 되면, 필드들이 의도치 않게 공개 API가 될 수 있으므로 주의해야 한다. (아이템 86,87)

## 테스트를 위해 접근 범위를 넓히지 않도록 주의하라
- 테스트를 위해 필드를 public으로 만들지 말 것.
  - 테스트 코드를 테스트 대상과 같은 패키지에 두면 package-private 요소에 접근 가능.
  - private 메서드 테스 관련 [링크](https://mangkyu.tistory.com/235).
## 정보은닉을 위한 클래스 설계 시 몇가지 주의점
- public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.
  - public 가변 필드를 갖는 클래스는 일반적으로 thread-safe하지 않다.
- public 접근자인데 필드가 final이며, 불변 객체를 참조한다고 해서 안심하면 안됨.
  - 리팩토링할 때 이미 이 클래스를 다른 클래스가 사용중이라면 public 필드는 함부로 없애지 못한다. 
- 배열은 public final로 제공하더라도, 배열 내용을 변경할 수 있음 주의. 
  - public static final Thing[] VALUES = { ... };
  - 사용자는 이 배열의 내용을 변경할 수 있음.
배열을 public으로 공개하고 싶다면, 아래 코드처럼 작성 가능.
```Java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> values = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```
또는
```Java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values(){
    return PRIVATE_VALUES.clone();
        }
```
값의 확인을 원하는지 복사를 원하는지에 따라 위의 코드처럼 구현하면 될 것이다.
      
## 자바의 모듈 개념
자바 내부에 module-info.java 파일을 보면 자신이 속하는 패키지 중 공개할 것을 정하는 부분을 볼 수 있다.
```Java
module java.base {
  exports java.io;
  exports java.lang;
  exports java.lang.annotation;
  exports java.lang.constant;
  exports java.lang.invoke;
  exports java.lang.module;
  exports java.lang.ref;
  exports java.lang.reflect;
  exports java.lang.runtime;
  exports java.math;
    ...
}
```

- 위와 같이 공개할 패키지를 정하면, 공개하지 않은 패키지는 아무리 public 접근 제어자를 갖는 멤버가 있더라도 접근 불가능하다.
- 모듈에 적용하는 접근 수준은 상당히 주의해서 사용해야 한다.
  - 모듈의 JAR 파일을 자신의 모듈 경로가 아닌 애플리케이션의 클래스패스(class-path)에 두면 그 모듈 안의 모든 패키지는 마치 모듈이 없는 것처럼 행동한다.
  - 즉, 모듈이 공개했는지 여부와 상관없이 public, protected 멤버를 모듈 밖에서도 접근 가능하게 된다.
  - 모듈은 아직 널리 받아들여질지 확실하지 않은 개념이다. 꼭 필요한 경우가 아니라면 사용하지 말자. (라고 하지만 대부분 라이브러리에서 사용하고 있는것 같다.)
Lombok에서도 이렇게 사용한다.
```Java
    module lombok {
    requires java.compiler;
    requires java.instrument;
    requires jdk.unsupported;

    exports lombok;
    exports lombok.experimental;
    exports lombok.extern.apachecommons;
    exports lombok.extern.flogger;
    exports lombok.extern.jackson;
    exports lombok.extern.java;
    exports lombok.extern.jbosslog;
    exports lombok.extern.log4j;
    exports lombok.extern.slf4j;
    exports lombok.launch to
    lombok.mapstruct;

    provides javax.annotation.processing.Processor with
    lombok.launch.AnnotationProcessorHider.AnnotationProcessor;
    }
```

## 정리
- 접근제한자는 가능한 최소로 주자.
- 꼭 필요한 것만 골라 최소한의 public API를 설계하는 편이 좋다.
- 멤버가 의도치 않게 API로 공개되는 일이 없도록 하자.
- public 클래스는 상수용 public static final 필드 외에 어떠한 public 필드도 가져선 안된다.
- public static final 필드가 참조하는 객체가 불변인지 확인하라.
