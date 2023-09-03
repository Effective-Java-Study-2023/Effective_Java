# 아이템 13. clone 재정의는 주의해서 진행해라

보기 편하게 [notion](https://obtainable-poppyseed-72e.notion.site/clone-da235f49eb104e5da3dee9ae41010258?pvs=4) 으로 정리했습니다.

### 아이템 13의 3줄 요약
1. Cloneable을 확장하지 말고, 구현하지도 말자. 문제가 될 가능성이 높다.
2. 복제 기능을 원한다면 생성자와 팩터리를 이용하자.
3. 단, 배열은 유일하게 clone 메서드 방식이 깔끔한 예외이다.