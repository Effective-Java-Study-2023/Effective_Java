# 아이템9. try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에서는 자원 사용 후 close()로 닫아줘야 자원이 많다. 
예를들면 InputStream, OutputStream, java.sql.connection이 좋은 예다.

### 왜 파일 입출력 , 네트워크 연결관련 자원 , 데이터베이스 연결 관련 자원 등은 close()를 해주어야 하나?
자원 관리: 파일, 네트워크 연결, 데이터베이스 연결과 같은 작업은 메모리만을 관리하는 것이 아니라, 외부 리소스에 접근하거나 열결을 유지하는 등의 작업을 수반. 이러한 리소스들은 메모리 이외의 자원을 차지하며, GC는 이러한 외부 리소스의 관리를 담당할 수 없다.

접근 제어와 정리 작업: 파일이나 네트워크 연결, 데이터베이스 연결과 같은 리소스는 열린 상태에서의 접근과 사용이 필요하다. 따라서 이러한 리소스를 단순히 GC에 의해 메모리에서 해제하는 것만으로는 충분하지 않다. 리소스를 사용한 후에는 명시적으로 접근을 닫아주고, 사용한 리소스를 정리(Close 또는 Disconnect)하는 작업이 필요하다.

만약 GC가 파일이나 네트워크 연결, 데이터베이스 연결과 같은 리소스를 자동으로 해제해준다면, 개발자가 리소스를 닫지 않는 경우에도 자원이 해제되기 때문에 심각한 문제가 발생할 수 있다. 개발자가 명시적으로 리소스를 정리하도록 하는 것은 안정성을 확보하기 위해 중요하다.

전통적으로 이러한 자원들은 try-finally를 통해 자원을 닫아주었다.
``` java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```
만약 자원이 늘어난다면 ,
``` java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
    OutputStream out = new FileOutputStream(dst);
    try {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0)
    out.write(buf, 0, n);
    } finally {
    out.close();
    }
    } finally {
    in.close();
    }
}
```

### try-finally의 단점 1
첫 번째 단점은 가독성이 안좋다는 것이다.

### try-finally의 단점 2
두 번째 단점은 스택 추적이 어려울 수 있다는 것이다.
위의 firstLineOfFile 메소드를 통해 좀 더 자세히 살펴보자.
예외는 try 블록과 finally 블록 모두에서 발생할 수 있다.
예를 들어 기기에 물리적인 문제가 생긴다면 firstLineOfFile 메서드 내의
readLine 메서드가 예외를 던지고, 같은 이유로 close 메서드도 실패한다.
예외가 두개가 생겨버리고 close 예외가 readLine 예외를 덮어버린다.
실제로 스택 추적 내역에 첫 번째 예외에 관한 내용은 등장하지 않게 되고,
이는 실제 시스템에서의 디버깅을 몹시 어렵게 한다.

왜냐하면 첫 번째 예외부터 해결하는 것이 보통 에러 해결의 순서이기 때문이다.

두 번째 예외 대신 첫 번째 예외를 기록하도록 코드를 수정할 수도 있지만
코드가 너무 지저분해진다.



따라서 자바 7에서부터 제공하는 try-with-resource 구문을 사용하여 자원을 처리하는 것이 좋다. 이 구조를 사용하려면 자원이 AutoCloseable 인터페이스를 구현해야 한다. 이 인터페이스는 close 메소드만 정의한 인터페이스다.

try-with-resources를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.
왜냐하면 close 메소드를 무조건 호출하기 위해 AutoCloseable를 사용하기 때문이다.
AutoCloseable 인터페이스는 close 메서드 하나만 정의되어 있다.
자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스가 이미 AutoCloseable을 구현하거나 확장해뒀다.
닫아야 하는 클래스를 작성한다면 AutoCloseable을 반드시 구현하여 try-with-resources를 사용하자.
``` java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

``` java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    }
}
```

try-finally보다 문제를 진단하기도 훨씬 편하다. readLine과 코드에는 보이지 않는 close 모두에서 예외가 발생한다면
스택 추적 내역에 readLine에서 발생한 예외가 먼저 표시된다.
그리고 close에서 발생한 예외는 숨겨졌다는 꼬리표인 suppressed를 달고 그 이후에 같이 출력된다.
try-finally와 달리 첫 번째 예외부터 확인할 수 있는 것이다.

아래 코드를 통해 예외를 보자.
``` java
class ExampleResource implements AutoCloseable{
    public void accessResource(){
        throw new RuntimeException("난 아마 보일 예외");
    }

    @Override
    public void close() throws Exception{
        throw new NullPointerException("난 아마 suppressed에서 보여질 예외");
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        try(ExampleResource exampleResource = new ExampleResource()){
            exampleResource.accessResource();
        }
    }
}
```
![이미지](https://velog.velcdn.com/images/dkdk/post/6bd6258b-9b1f-4f11-b083-2b53fb588d26/image.png)
아래처럼 accessResource에서 발생하는 예외가 먼저 표시되고 , suppressed에서 close메서드 호출에서 나온 예외를 볼 수 있다.
