# 0. ORM과 JPA란 무엇인가?
## ORM?
- Object Relational Mapping 프레임워크
- 객체-RDB 간의 차이를 중간에서 해결\

## JPA?
- 자바 진영의 ORM 기술 표준
    - 실행 시점에 SQL 생성
- 장점
    - 직접 SQL 작성할 필요 없음
    - 조회 된 record -> 객체(역직렬화) 자동 처리
    - RDB인 경우 코드 수정 없이 DB 수정 손쉽게 가능
- 성능에 대한 대안
    - JPQL 사용
    - DB 쿼리 힌트 사용
 
# 1. SQL을 직접 다룰 때 발생하는 문제점
## 반복적 작업
- JDBC API를 부르고, SQL을 입력하는 과정에서 수많은 코드가 필요함
- 각 SQL 실행 마다 이를 불러야하므로, 코드의 중복이 불가피

## SQL 의존적 개발
- 연관된 객체 사용 여부는 SQL에 달려있음
    - 데이터 접근 계층(DAO)를 사용하여 SQL을 숨겨도 결국 다시 열어서 확인 필요
    - 진정한 의미의 계층 분할이 어려움
- Entity 신뢰 불가

## JPA와 문제 해결
1. JDBC API를 직접 부르지 않고 JPA API를 사용하여 쿼리 메소드 작성
2. 수정 메소드 별도 제공 X : 객체 조회 후 값 변경 감지 시 트랜잭션을 커밋할 때 Update SQL문 실행
3. Entity에서 연관 관계 설정 시, 마음껏 조회 가능

# 2. 패러다임의 불일치
- RDB는 데이터 중심 구조화, 집합적 사고 요구
    - 객체 지향의 특징(상속, 추상화, 다형성) 개념 없음
    - **객체-RDB의 패러다임 불일치** 발생

## 상속
- JPA에서 자바 컬렉션에 객체를 저장하듯이 JPA에 객체를 저장하여 상속처럼 사용
```java
// 객체 모델 코드
abstract class Item {
  Long id;
  String name;
  int price;
}

class Album extends Item {
  String artist;
}
```
```java
// JPA의 persist 메소드를 사용하여 객체 저장
jpa.persist(album)
```
```sql
-- 아래의 SQL을 JPA가 실행
INSERT INTO ITEM (id, name, price) VALUES (1, "name", 10000);
INSERT INTO ALBUM (item_id, artist) VALUES (1, "artist");

-- jpa.find(Album.class, 1); SQL 실행
SELECT i.*, a.* FROM item i JOIN album a on i.id = a.item_id;
```

## 연관 관계
- 연관 관계 설정 by 객체(참조) / 테이블(FK, join을 통한 조회)
- 객체는 참조가 있는 방향으로만 조회 가능, 테이블은 외래 키 하나로 양방향 조회 가능
- JPA에서 개발자는 두 entity의 관계를 설정하고, 참조될 객체를 저장하면 됨
    - 객체 조회 시 외래 키를 참조로 변환하는 일은 JPA 담당

## 객체 그래프 탐색
- 참조를 이용하여 연관된 객체를 찾는 일
- SQL을 직접 다룰 시, 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색 가능한지 정해짐
  - 객체 지향적이지 않고 제약이 큼
- JPA 사용 시, 연관 관계가 있다면 객체 그래프를 마음껏 탐색 가능
  - 즉시로딩, 지연로딩이 존재함

## 비교
- DB는 PK로 row(record) 비교
- 객체는 동일성(같은 주소인지), 동등성(내용물이 같은지, equals() 메소드 사용)으로 비교
- JPA에서는 동일 트랙잭션일 때 같은 객체가 조회되는 것을 보장함
