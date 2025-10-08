-----
### 💡 조인의 특징
-----
1. 기준으로 삼는 테이블의 한 행(Row)이 다른 쪽 테이블의 여러 행과 연결될 수 있다면, 결과의 전체 행 수는 늘어남
    - 반대로 한 행이 다른 쪽 테이블의 단 하나의 행과 연결되거나, 아무 행과도 연결되지 않는다면 행의 수는 늘어나지 않음
    - 이 원리를 이해하려면 테이블 간의 관계, 특히 기본 키(Primary Key, PK)와 외래 키(Foreign Key, FK)의 관계를 알아야 함

<div align="center">
<img src="https://github.com/user-attachments/assets/d09cc193-6ce5-4a32-a877-b52bd2e652c6">
</div>

   - Primary Key (PK) : 테이블에서 각 행을 고유하게 식별하는 값
      + users 테이블의 user_id나 products 테이블의 product_id가 여기에 해당
      + PK는 테이블 내에서 절대로 중복될 수 없음

   - Foreign Key (FK):  다른 테이블의 PK를 참조하는 값
     + orders 테이블의 user_id는 users 테이블의 user_id를 참조하는 FK
     + FK는 참조하는 테이블에서 여러 번 중복되어 나타날 수 있음
     + 예를 들어, 한 명의 고객(user_id)이 여러 번의 주문(orders)을 할 수 있기 때문임

2. 이 관계를 '부모-자식 관계'에 비유하면 이해하기 쉬움
   - 부모 테이블 (Parent Table) : PK를 가지고 있는 테이블 (users, products)
   - 자식 테이블 (Child Table) : FK를 통해 부모 테이블을 참조하는 테이블 (orders)
   - 💡 이 개념을 바탕으로 조인 시 데이터 행 수가 어떻게 변하는지 정리하면 다음과 같음
      + 자식-부모 조인 (FK → PK 참조): 행 개수가 늘어나지 않음
         * orders 테이블을 기준으로 users 테이블을 조인하는 경우
         * 자식 테이블(orders)의 각 주문 정보는 반드시 단 한 명의 부모(users)하고만 연결
         * 주문 하나가 여러 고객의 것일 수는 없기 때문임
         * 따라서 기준 테이블인 orders의 행 개수가 그대로 유지
         * PK는 유일한 하나의 값만 저장 : 따라서 PK 방향으로 참조하는 경우 행 개수가 늘어나지 않음

      + 부모-자식 조인 (PK → FK 참조) : 행 개수가 늘어날 수 있음
         * users 테이블을 기준으로 orders 테이블을 조인하는 경우
         * 부모 테이블(users)의 한 고객은 여러 명의 자식(여러 건의 orders)을 가질 수 있음
         * 이 경우, 한 고객의 정보를 여러 주문 정보에 각각 매칭시켜야 하므로, 고객 정보 행이 주문 건수만큼 복제되어 전체 행의 수가 늘어남
         * FK는 같은 값을 여러개 저장할 수 있음
         * 따라서 FK 방향으로 참조하는 경우 행 개수가 늘어날 수 있음

3. 특정 고객 '션'으로 조인 원리 파헤치기
   - 먼저 users 테이블과 orders 테이블에서 '션'의 데이터를 확인
   - users 테이블에서 '션'(user_id:1)의 정보는 단 한 행 : user_id가 PK이므로 유일하게 하나여야 함
```sql
SELECT user_id, name, email
FROM users
WHERE user_id = 1;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/d4aaa4a5-3837-40c8-941c-da105c44160d">
</div>

   - orders 테이블을 보면 '션'(user_id=1)이 주문한 내역은 두 행
   - user_id가 FK이므로 이렇게 중복이 가능하고, 여러 행이 조회될 수 있음
```sql
SELECT order_id, product_id, user_id
FROM orders
WHERE user_id = 1;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/7a7ef1ae-4fb8-4454-8930-d3f868a9b9ec">
</div>

   - 경우 1 : 자식-부모 조인 (orders → users, 행 개수 유지)
      + 먼저 orders 테이블을 기준으로 '션'의 주문 내역에 고객 이름을 붙여보기
      + 이것이 바로 자식 테이블(orders)에서 부모 테이블(users)로, 즉 FK → PK로 조인하는 경우
```sql
SELECT
   o.order_id,
   o.product_id,
   o.user_id AS orders_user_id,
   u.user_id AS users_user_id,
   u.name,
   u.email
FROM
   orders o
JOIN
   users u ON o.user_id = u.user_id
WHERE
   o.user_id = 1;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/518b5541-69f3-4569-b9ed-6cdb0d9b1a48">
</div>

   - 기준 테이블인 orders 에는 '션'의 주문이 2행 (order_id: 1 , order_id: 2)
   - 첫 번째 주문(order_id: 1)의 user_id는 1 : 이 값은 users 테이블에서 PK가 1인 '션'의 단일 행과 매칭
   - 두 번째 주문(order_id: 2)의 user_id 역시 1 : 이 값도 users 테이블에서 PK가 1인 '션'의 동일한 단일 행과 매칭
   - 각 주문 행이 오직 하나의 고객 행과 연결되므로, 기준 테이블(orders)의 행 수가 그대로 유지
<div align="center">
<img src="https://github.com/user-attachments/assets/1da92d31-0852-4bb2-9414-82501107d166">
</div>

   - orders 테이블은 조인 전에도 2행이었고, 조인 이후에도 2행이 그대로 유지
   - 이것이 자식-부모 조인의 특징
     
   - 경우 2 : 부모-자식 조인 (users orders, 행 개수 증가)
     + 이번에는 반대로 users 테이블을 기준으로 '션' 고객의 주문 내역을 붙여보기
     + 이것이 바로 부모 테이블(users)에서 자식 테이블(orders)로, 즉 PK → FK로 조인하는 경우
```sql
SELECT
   u.user_id AS users_user_id,
   u.name,
   u.email,
   o.order_id,
   o.product_id,
   o.user_id AS orders_user_id
FROM
   users u
JOIN
   orders o ON u.user_id = o.user_id
WHERE
   u.user_id = 1;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/aac50a70-7203-435d-8526-9ca8c5551438">
</div>

   - 기준 테이블인 users 에는 '션'의 정보가 단 1행 
   - 이 '션'의 user_id:1과 일치하는 행을 orders 테이블에서 찾음
   - orders 테이블에는 user_id가 1인 주문이 2개(order_id:1 , order_id:2)
     + users 의 '션' 정보 1행이 orders의 2개 행과 관계를 맺어야 함
   - 데이터베이스는 관계를 표현하기 위해 '션'의 정보 행을 복제하여 각 주문에 맞추어 하나씩 붙임
     + '션' 정보 (복제본 1) + 첫 번째 주문 정보
     + '션' 정보 (복제본 2) + 두 번째 주문 정보
   - 결과적으로 1개의 행이 2개의 행으로 늘어남
<div align="center">
<img src="https://github.com/user-attachments/assets/21043153-16b0-4fe9-9faf-afa6d3dd243b">
</div>

   - 하지만 기준으로 삼았던 users 테이블에서 '션'의 정보는 원래 1행
   - 그런데 조인의 결과는 2행이 됨 : 바로 이 지점이 JOIN 으로 인해 행이 증가하는 핵심 원리
   - 부모 테이블의 한 행이 자식 테이블의 여러 행과 관계를 맺을 때, 그 관계의 수만큼 행이 복제되어 결과가 늘어나는 것

4. 전체 데이터로 확장하기
   - users 테이블(6행)과 orders 테이블(7행)에서 전체 데이터를 확인
```sql
SELECT user_id, name, email
FROM users;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/e6d6305b-2d75-4060-ba30-1fc78aef6884">
</div>

```sql
SELECT order_id, product_id, user_id
FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f08b5d52-e717-4ec2-acf3-beccde8a7bff">
</div>

   - 전체 주문(Orders) 기준 (행 개수 유지) : orders 테이블을 기준으로 모든 주문에 고객 이름을 붙여보기 (자식-부모 조인)
```sql
SELECT
   o.order_id,
   o.product_id,
   o.user_id AS orders_user_id,
   u.user_id AS users_user_id,
   u.name,
   u.email
FROM
   orders o
JOIN
   users u ON o.user_id = u.user_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/1127b627-abce-43c5-ba98-78e680c76dd8">
</div>

   - orders 테이블의 각 행은 FK인 user_id를 가지고 있음
   - 이 FK는 users 테이블의 PK인 user_id를 참조
   - PK는 고유하므로, orders 테이블의 각 행은 언제나 users 테이블의 단 하나의 행하고만 연결
<div align="center">
<img src="https://github.com/user-attachments/assets/e59ae173-ab9c-4e0d-af19-09aa9f8c21a1">
</div>

   - 예상대로 결과는 7행 : 기준 테이블인 orders의 7개 주문이 각각 단 한 명의 고객과 연결되므로, 행의 개수는 그대로 유지
   - 전체 고객(Users) 기준 (행 개수 증가) : users 테이블(6행)을 기준으로 모든 고객의 주문 내역을 붙여보기 (부모-자식 조인)
```sql
SELECT
   u.user_id AS users_user_id,
   u.name,
   u.email,
   o.order_id,
   o.product_id,
   o.user_id AS orders_user_id
FROM
   users u
JOIN
   orders o ON u.user_id = o.user_id
ORDER BY users_user_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/9a58f445-9271-4075-ba3b-c73e7b89cb02">
</div>

   - 결과는 7행 : users 테이블은 6행이었지만, 조인 결과는 7행
   - '션'(user_id: 1)과 '네이트'(user_id: 2)가 각각 2건의 주문을 가지고 있었기 때문에, 이들의 정보가 한 번씩 더 복제되어 총 7개의 행
   - 참고로 주문이 없는 '레오나르도 다빈치'(user_id: 6)는 INNER JOIN의 특성상 orders 테이블에 매칭되는 데이터가 없으므로 결과에서 제외
   - 만약 OUTER JOIN 을 사용했다면 결과에 포함되었을 것

5. 💡 정리 : 언제 행이 늘어나고 언제 그대로인가?
   - 행 개수 유지 : 자식에서 부모로 조인할 때 (to-One) : FK → PK 조인
      + 방향 : FROM 자식 테이블 JOIN 부모 테이블 (예: FROM orders JOIN users )
      + 원리 : 자식 테이블의 모든 행은 부모 테이블의 단 하나의 행과 매칭 (주문은 반드시 한 명의 고객에게 속함)
      + 결과 : 기준이 되는 자식 테이블의 행 개수가 그대로 유지 (orders 테이블이 7행이면 결과도 7행)
   - 행 개수 증가 가능 : 부모에서 자식으로 조인할 때 (to-Many) : PK → FK 조인
      + 방향 : FROM 부모 테이블 JOIN 자식 테이블 (예: FROM users JOIN orders )
      + 원리 : 부모 테이블의 한 행은 자식 테이블의 여러 행과 매칭될 수 있음 (한 명의 고객이 여러 번 주문할 수 있음)
      + 결과 : 부모 행이 자식 행의 개수만큼 복제되면서 전체 행의 수가 기준 테이블보다 늘어날 수 있음 주문을 2번 한 고객은 결과 테이블에서 2개의 행을 차지하게 됨)

6. 💡 실무에서 이것이 왜 중요할까?
   - 이 원리를 모르면 데이터를 잘못 분석하게 될 위험이 큼
   - 예를 들어, 모든 고객과 그들의 주문 정보를 보기 위해 FROM users JOIN orders를 수행했다고 가정
     + 이 결과에서 고객 수를 세기 위해 COUNT(u.user_id) 를 실행하면 어떻게 될까? 전체 고객 수인 6이 아닌, 주문을 여러 번 한 고객이 중복 계산되므로, 전체 주문 수인 7이 나옴
   - 이처럼 조인으로 인해 데이터가 어떻게 변하는지 정확히 이해해야만, 합계(SUM), 평균(AVG), 개수(COUNT) 같은 집계 함수를 올바르게 사용하고 원하는 분석 결과를 정확하게 도출할 수 있음
   - 쿼리를 작성하기 전에 항상 어떤 테이블을 기준으로 삼을지, 그리고 조인으로 인해 행 수가 증가하는 상황인지 아닌지, 먼저 생각하는 습관을 들이는 것이 중요

7. 참고 : 조인의 유연성
   - 데이터베이스에서 조인(JOIN)은 주로 기본 키(PK)와 외래 키(FK)를 써서 테이블을 연결하는 것이 가장 일반적이고 중요한 방식
   - 그러나 조인의 핵심 원리는 '두 테이블의 특정 열(column)의 값이 같은가?' 이기에, 실제로는 어떤 열이든 조인 조건으로 쓸 수 있음 (데이터 타입만 같으면 됨)
   - 다양한 조인 예시
     + PK-FK 관계가 아니더라도 여러 상황에서 조인을 활용할 수 있음
     + 동명이인 찾기 (이름으로 조인) : 고객 테이블과 직원 테이블에 모두 '이름' 열이 있다면, 이 두 테이블을 이름으로 조인하여 고객과 직원의 이름이 같은 경우를 찾아냄
 ```sql
SELECT A.이름, A.연락처, B.부서
FROM 고객 AS A
JOIN 직원 AS B ON A.이름 = B.이름;
 ```
   - 특정 날짜의 이벤트 연결 (날짜로 조인) : 주문 테이블의 '주문일자'와 마케팅_이벤트 테이블의 '이벤트_시작일'을 기준으로 조인하여, 특정 이벤트가 있던 날에 들어온 주문을 분석할 수 있음
 ```sql
SELECT A.주문번호, A.주문금액, B.이벤트명
FROM 주문 AS A
JOIN 마케팅_이벤트 AS B ON A.주문일자 = B.이벤트_시작일;
 ```
   - 지역별 데이터 분석 (지역 코드로 조인) : 매장 테이블의 '우편번호' 앞 두 자리와 지역별_인구통계 테이블의 '지역코드'를 연결하여, 매장이 있는 지역의 인구 통계 데이터를 함께 분석
 ```sql
SELECT A.매장명, B.평균소득
FROM 매장 AS A
JOIN 지역별_인구통계 AS B ON SUBSTRING(A.우편번호, 1, 2) = B.지역코드;
 ```
   - 로그 데이터 분석 (상태 코드로 조인) : 웹서버_로그 테이블의 '상태코드'와 에러_코드_정의 테이블의 '코드'를 조인하여, 로그에 기록된 에러가 무엇을 뜻하는 지 바로 파악할 수 있으
 ```sql
SELECT A.요청URL, B.에러설명
FROM 웹서버_로그 AS A
JOIN 에러_코드_정의 AS B ON A.상태코드 = B.코드;
 ```

8. PK-FK 조인이 중요한 이유
   - 이처럼 조인은 매우 유연하지만, 실무에서는 데이터의 정확성과 일관성을 위해 대부분 PK와 FK를 씀
   - 이름처럼 중복될 수 있거나, 언제든 바뀔 수 있는 값으로 조인하면 데이터가 엉뚱하게 연결될 위험이 크기 때문임
   - 안정적인 PK-FK 관계를 이해하는 것이야말로 조인을 제대로 활용하는 첫걸음
