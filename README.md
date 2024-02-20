## effective-java-study

📚 이펙티브 자바 스터디

### 목표

+ 내용이 방대하여 혼자 독학하기 힘든 이펙티브 자바를 같이 공부한다

### 일정

+ 매주 화요일 22시 ~ 23시
+ 첫번째 스터디 시작일: 24.01.02 (화)

### 진행 방식

+ 각 주마다 지정된 범위의 아이템을 읽고 정리한다.
    + 이미지는 issue에 올리고, 그 주소를 사용하는 걸로 한다.
    + 주차별 아이템 범위는 최대 3개 이하로 한다.
+ 질문 및 논의할 사항이 있으면 issue에 올린다.
+ 발표 자료 정리가 끝난 후 PR을 날린다.
+ 발표는 주마다 랜덤 추첨으로 진행한다.
    + 한 사이클이 도는 동안 발표를 진행한 사람은 랜덤 추첨에서 제외된다.
    + 발표 시간은 30분을 넘지 않는다.
+ 발표가 끝난 후 내용에 대해 토론 한다.

### 폴더 구조

```
|-- 02장
  |-- 아이템_01
     - 생성자_대신_정적_팩터리_메서드를_고려하라_{이름}.md
     - 생성자_대신_정적_팩터리_메서드를_고려하라_{이름}.md
     - 생성자_대신_정적_팩터리_메서드를_고려하라_{이름}.md
```

### 스터디 주차별 목록

#### 1주차 - 24.01.09

| 아이템                                    | 발표자 | 
|----------------------------------------|-----|
| 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라         | 윤용현 | 
| 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라         | 윤용현 |
| 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라 | 윤용현 | 
| 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라  | 윤용현 |

#### 2주차 - 24.01.16

| 아이템                                  | 발표자 | 
|--------------------------------------|-----|
| 아이템 5. 지원을 직접 명시하지 말고 의존 객체 주입을 사용하라 | 김우철 | 
| 아이템 6. 불필요한 객체 생성을 피하라               | 김우철 |
| 아이템 7. 다 쓴 객체 참조를 해제하라               | 김우철 | 

#### 3주차 - 24.01.23

| 아이템                                            | 발표자 | 
|------------------------------------------------|-----|
| 아이템 8. finalizer와 cleaner 사용을 피하라              | 정교성 | 
| 아이템 9. try-finally 보다 try-with-resources를 사용하라 | 정교성 |

#### 4주차 - 24.02.06

| 아이템                                     | 발표자 | 
|-----------------------------------------|-----|
| 아이템 10. equals는 일반 규약을 지켜 재정의하라         | 정교성 | 
| 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라 | 정교성 |

#### 5주차 - 24.02.13

| 아이템                          | 발표자 | 
|------------------------------|-----|
| 아이템 12. toString을 항상 재정의하라   | 윤용현 | 
| 아이템 13. clone 재정의는 주의해서 진행하라 | 윤용현 |

#### 6주차 - 24.02.20

| 아이템                           | 발표자 | 
|-------------------------------|-----|
| 아이템 14. Comparable을 구현할지 고려하라 | 김우철 | 
| 아이템 15. 클래스와 멤버의 접근 권한을 최소화하라 | 김우철 |

#### 7주차 - 24.02.20

| 아이템                                               | 발표자 | 
|---------------------------------------------------|-----|
| 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라 |     | 
| 아이템 17. 변경 가능성을 최소화하라                             |     |


### Git 사용 방식

1. 스터디원 각자 본인만의 branch를 생성하여 해당 branch 에서 작업(코드/md 작성)을 진행한다.
    1. 커밋 메세지는, 아이템명과 동일하다.
        1. `[아이템 08] finalizer와 cleaner 사용을 피하라`
2. branch에서의 작업이 완료되었으면 해당 작업에 대한 PR(main 브랜치)을 스터디 시작 1시간 전까지 요청한다.
    1. PR 제목은 아래 형식으로 한다.
        1. `김우철 - 1주차 정리 및 발표자료 (item 01,02,03,04)`
3. 궁금하거나 토론할 주제가 있으면 깃허브 issue를 올린다.
    1. 이슈 제목은 아래 형식으로 한다.
        1. `[아이템 08]. {제목}`
4. 스터디가 끝난후 스터디장은 주차별 PR을 merge한다.
    1. squash and merge로 PR을 merge한다.
    2. source 브랜치는 삭제한다.
5. PR merge가 모두 완료된 후에 main 브랜치를 base로 1번 과정을 다시 반복한다.
