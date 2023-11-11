# 아이템 54. null이 아닌 빈 컬렉션이나 배열을 반환하라

### null을 반환하면 안되는 이유
~~~Java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
~~~
이처럼 null을 반환할 경우, API 사용자는 null 처리 코드를 추가로 작성해야 한다.
~~~Java
public static void main(String[] args) {
        Shop shop = new Shop();
        List<Cheese> cheeses = shop.getCheeses();
        if (cheeses != null && cheeses.contains(Cheese.STILTON))
            System.out.println("Good!");
}
~~~
어떤 API가 컨테이너를 반환하는데 비어있을 경우에만 null을 반환한다면, 항상 위 예시와 같은 방어 코드를 넣어줘야 한다.
객체가 0개일 상황이 매우 드물게 발생한다면, 방어 처리 코드 부재로 인한 오류가 뒤늦게 발견될 것이다.

### 빈 컨테이너를 반환하는 비용이 부담된다?
본문에서는 두 가지 측면에서 이 주장을 반박한다.
- 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못 된다.
- 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.
~~~Java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
~~~
위와 같이 작성하면 빈 컬렉션까지 올바르게 반환하게 된다.
~~~Java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
~~~
toArray 메서드는 인자로 주어진 배열이 충분히 크면 그 안에 컬렉션의 원소를 담아 리턴하고, 그렇지 않으면 새로운 배열을 만들어 리턴한다.
## 정리
- null을 반환하면 API를 사용하기 어렵게 만들고, 오류 처리 코드도 늘어난다.
- 그렇다고 성능이 좋은 것도 아니다.
- 따라서 (컬렉션을 반환하는 경우에) null 대신 빈 컬렉션이나 배열을 반환하자.