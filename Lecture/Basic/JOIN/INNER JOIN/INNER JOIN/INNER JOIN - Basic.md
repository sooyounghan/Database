-----
### 내부 조인
-----
1. 내부 조인의 개념
   - 💡 두 테이블을 연결할 때, 양쪽 테이블에 모두 공통으로 존재하는 데이터만을 결과로 보여줌
   - 기준이 되는 컬럼(예) orders.user_id와 users.user_id)의 값이 서로 일치하는 행들만 짝을 지어주는 것
   - orders 테이블에 user_id가 존재하고, 연결되는 users 테이블에도 해당 user_id를 가진 사용자가 존재할 때만 결과에 포함
      + 예를 들어 orders 테이블에 user_id가 1인 주문이 있고 users 테이블에 user_id가 1인 회원이 있다면 둘은 함께 결과에 포함

   - 예를 들어 orders 테이블에 user_id가 99인 주문이 있지만 users 테이블에 user_id가 99인 회원이 없다면, 결과에서 제외 (실제로는 데이터 무결성을 위해 외래 키(FOREIGN KEY) 제약조건을 사용하므로 이런 경우는 발생하지 않음)

2. 내부 조인 문법
```sql
SELECT 컬럼1, 컬럼2, ...
FROM 테이블A
INNER JOIN 테이블B
ON 테이블A.연결컬럼 = 테이블B.연결컬럼;
```
   - FROM : 기준이 되는 첫 번째 테이블을 지정
   - INNER JOIN : 연결할 두 번째 테이블을 지정
   - ON : 조인에서 가장 중요한 부분
     + 두 테이블을 어떤 조건으로 연결할지 명시하는 연결고리
     + ON 절의 조건이 참(true)이 되는 행들만 결과에 포함

3. INNER 생략  
   - 내부 조인에서 INNER 는 생략할 수 있음
   - 그냥 JOIN 이라고만 쓰면 INNER JOIN 으로 작동
   - 내부 조인을 사용하는 경우 실무에서는 대부분 INNER를 생략하고 JOIN만 사용

4. 실습 : 주문별 고객 정보 조회하기
   - 1단계 : 두 테이블을 그대로 연결하기
      + 먼저 orders 테이블과 users 테이블을 orders.user_id와 users.user_id를 기준으로 연결
      + SELECT * 를 사용해서 어떤 결과가 나오는지 눈으로 직접 확인
```sql
SELECT *
FROM orders
INNER JOIN users
ON orders.user_id = users.user_id;
```
   - 실행 결과는 orders 테이블의 모든 컬럼과 users 테이블의 user_id가 같은 모든 컬럼이 옆으로 합쳐진, 아주 넓은 테이블이 됨
   - 조인은 쉽게 이야기해서 테이블을 옆으로 합치는 것
<div align="center">
<img src="https://github.com/user-attachments/assets/e715d022-556a-4d0b-845b-c58674430ae5">
</div>

   - 데이터베이스는 orders 테이블과 users 테이블을 조회
   - JOIN ON에 의해 orders의 user_id와 users의 user_id를 기준으로 조인
   - orders의 user_id의 값으로 users의 user_id 항목을 찾고 연결
      + order_id:1의 경우 user_id:1 이므로, 따라서 users의 user_id:1 항목과 연결
      + order_id:2의 경우 user_id:1 이므로, 따라서 users의 user_id:1 항목과 연결
      + order_id:3의 경우 user_id:2 이므로, 따라서 users의 user_id:2 항목과 연결
      + order_id:4의 경우 user_id:3 이므로, 따라서 users의 user_id:3 항목과 연결
      + order_id:7의 경우 user_id:2 이므로, 따라서 users의 user_id:2 항목과 연결
      + user_id:6인 레오나르도 다빈치의 경우 연결할 대상이 없으므로, 따라서 내부 조인 대상에 포함되지 않음
<div align="center">
<img src="https://github.com/user-attachments/assets/9f5feca2-fc95-4d00-b242-39bb5cd89ee7">
</div>

   - 결과에서 볼 수 있듯이, orders와 users의 모든 컬럼이 옆으로 합쳐짐
   - user_id 컬럼이 두 번 나타나는 이유는 orders, users 두 테이블 모두에 존재하기 때문임
   - 💡 참고 : SELECT에 * 대신 컬럼을 직접 나열하는 경우 user_id처럼 중복으로 나타나는 컬럼은 테이블 명을 지정해주어야 함

   - 2단계 : 필요한 컬럼만 선택하고, WHERE 로 필터링하기
     + SELECT * 는 구조를 파악하는 데는 좋지만, 실제 보고서에는 불필요한 정보가 너무 많음
     + '고객 ID', '고객 이름', '주문 날짜'만 선택하고, WHERE절을 추가하여 주문이 완료된 COMPLETED 상태의 주문만 필터링
```sql
SELECT
     users.user_id, -- 테이블 명 생략 불가
     users.name, -- 테이블 명 생략 가능
     orders.order_date -- 테이블 명 생략 가능
FROM orders
INNER JOIN users ON orders.user_id = users.user_id
WHERE orders.status = 'COMPLETED';
```
   - 여기서 users.user_id는 테이블명.컬럼명 형식으로 작성한 것을 볼 수 있음
   - user_id 처럼 orders, users 두 테이블에 이름이 같은 컬럼이 있을 경우, 어떤 테이블의 user_id를 말하는지 명확히 지정해주어야 함 : 그렇지 않으면 오류가 발생
```
오류 메시지 : Column 'user_id' in field list is ambiguous (user_id 필드가 모호하다는 오류 메시지)
```
   - name, order_date의 경우 한 테이블에만 존재하기 때문에 모호함이 발생하지는 않음 : 이 경우 앞의 테이블명을 생략할 수 있음
   - 실무에서는 어떤 테이블의 필드인지 명시하는 것이 가독성을 높이고, 컬럼의 소속을 쉽게 인지할 수 있기 때문에 사용하는 것을 권장
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/86bba93b-22a4-4303-a468-bfe8c8a01c7b">
</div>

5. 조인 작동 순서
```sql
SELECT
     users.user_id,
     users.name,
     orders.order_date
FROM orders
INNER JOIN users ON orders.user_id = users.user_id
WHERE orders.status = 'COMPLETED';
```
   - 데이터베이스는 이 쿼리를 다음과 같은 논리적인 순서로 처리
   - FROM / JOIN : 가장 먼저 FROM 절의 orders 테이블과 INNER JOIN으로 연결된 users 테이블을 연결하기위해 ON 절에 명시된 orders.user_id = users.user_id 조건을 만족하는 행들을 결합하여 하나의 큰 가상테이블을 생성
<div align="center">
<img src="https://github.com/user-attachments/assets/68d6ad5a-621c-4db7-8827-994af29ef8cf">
</div>

   - WHERE : JOIN을 통해 생성된 가상 테이블에서 WHERE 절의 조건인 orders.status = 'COMPLETED'를 만족하는 행들만 필터링
     + 즉, COMPLETED 상태의 주문 데이터만 남게 됨
<div align="center">
<img src="https://github.com/user-attachments/assets/55199ad7-d437-4565-88c1-f71a76254bfe">
</div>

   - SELECT : 마지막으로, 필터링된 결과에서 SELECT절에 명시된 users.user_id, name과 order_date 컬럼을 추출하여 최종 결과를 반환
<div align="center">
<img src="https://github.com/user-attachments/assets/93d2396a-9327-4f34-b0c4-17fd99b27332">
</div>

   - JOIN을 통해 두 테이블을 먼저 합친 가상의 테이블을 만든 후, WHERE 절의 조건에 따라 필요한 행을 걸러내고, 최종적으로 원하는 필드를 SELECT로 선택하는 순서로 작동
   - 요약 : 논리적 처리 순서 (FROM/JOIN (테이블 결합) → WHERE (조건 필터링) → SELECT (컬럼 선택))

6. 💡 쿼리 최적화기 (쿼리 옵티마이저)
   - 사용자와 데이터베이스 간의 논리적 순서는 일종의 약속이지만, 데이터베이스 내부의 쿼리 최적화기(Query Optimizer)는 쿼리를 더 효율적인 방식으로 실행
   - 예를 들어, orders 테이블에서 COMPLETED 상태의 행만 먼저 선택한 다음, 남은 행을 기준으로 users 테이블과 조인하는 방식으로 진행하면 조인 대상이 줄어들어 성능이 더 최적화 될 수 있음
   - 쿼리 최적화기를 통해 실제 물리적인 실행 순서는 달라질 수 있지만 어떻게 작동하든 최종 결과는 논리적인 순서와 동일
   - 하지만 데이터베이스의 최적화 방식을 잘 이해하면 같은 결과를 얻으면서도 조회 성능을 최적화할 수 있음
   
