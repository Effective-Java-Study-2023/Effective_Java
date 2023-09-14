# 아이템 24. 멤버 클래스는 되도록 static으로 만들라

보기 편하게 [notion](https://obtainable-poppyseed-72e.notion.site/item-24-static-99a24eb5913f42dea548c2edbf00a0e7?pvs=4) 으로 정리했습니다.

### 아이템 24의 (주관적인) 3줄 요약
1. 중첩 클래스의 종류와 적절한 쓰임이 중요하다
2. 멤버 클래스 경우, 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적(non-static)
3. 그렇지 않으면 정적(static member class)으로 만들자