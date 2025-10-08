-----
### 내부 조인 - 조인과 집합
-----
1. 내부 조인을 이해하는 또 다른 방법은 바로 '집합'의 관점에서 바라보는 것
2. 내부 조인( INNER JOIN )은 두 테이블의 교집합을 찾는 것과 같음
    - 두 집합(테이블)에서 공통된 원소(연결 컬럼의 값이 일치하는 데이터)만을 결과로 반환
    - A 집합 : orders 테이블에 있는 모든 사용자들의 user_id 집합
    - B 집합 : users 테이블에 있는 모든 user_id 집합
    - INNER JOIN은 A 집합과 B 집합에 모두 포함된 user_id에 해당하는 데이터만을 결합하여 보여줌

3. 벤 다이어그램으로 이해하기
<div align="center">
<img src="https://github.com/user-attachments/assets/b34d3508-8adc-454a-b6ba-125eddf2d5fb">
</div>

   - orders 집합 (왼쪽 원) : 지금까지 들어온 모든 주문을 나타내며, 주문 데이터에 기록된 user_id는 {1, 2, 3, 4, 5} 존재 (참고로 집합은 중복 값은 제거하고 표현)
   - users 집합 (오른쪽 원) : 우리 쇼핑몰에 가입한 모든 회원을 나타내며, user_id가 {1, 2, 3, 4, 5, 6} 존재

<div align="center">
<img src="https://github.com/user-attachments/assets/1ad0d06a-d295-44b8-b529-5ac53708882b">
</div>

   - 교집합 (가운데 영역) : users와 orders 양쪽에 모두 존재하는 user_id의 집합 {1, 2, 3, 4, 5}으로, 내부 조인의 결과는 바로 이 교집합에 해당하는 데이터들

4. 용어 - 내부 조인
   - 벤 다이어그램을 보면 내부 조인(INNER JOIN)의 이름을 왜 내부라고 지었는지 알 수 있음
   - 내부 조인은 벤 다이어그램에서 둘의 겹친 영역인 교집합 영역을 말함
   - 교집합 영역은 벤 다이어그램에서 내부에 있는 데이터를 뜻함
   - 이후에 설명할 외부 조인(OUTER JOIN)은 교집합 영역의 밖(OUTER)의 행까지 포함한다는 의미

5. 집합의 관점에서 제외되는 데이터
<div align="center">
<img src="https://github.com/user-attachments/assets/dec9e6ea-b27b-4f82-8ea9-dcc5be8e3f5e">
</div>

   - 주문 기록이 없는 회원 : user_id가 6인 '레오나르도 다빈치'는 users 테이블에는 존재하지만, 아직 한 번도 주문한 적이 없으므로 orders 테이블에는 해당 user_id 가 없음
   - 따라서 이 회원은 users 집합에는 속하지만 교집합에는 속하지 않으므로 INNER JOIN 결과에서 제외

<div align="center">
<img src="https://github.com/user-attachments/assets/a0073512-287a-4789-b01f-056e3790ea4e">
</div>

   - 존재하지 않는 회원의 주문 : 만약 orders 테이블에 user_id가 99인 주문이 있는데, users 테이블에 user_id가 99인 회원이 없다면 어떻게 될까?
     + 이 주문 데이터는 orders 집합에는 속하지만 교집합에는 포함되지 않으므로 INNER JOIN 결과에서 제외 (실제로는 데이터 무결성을 위해 외래 키(FOREIGN KEY) 제약조건을 사용하므로 이런 경우는 발생하지 않음)
   - 결론적으로, INNER JOIN은 어느 한쪽에만 데이터가 존재하는 경우는 결과에 포함시키지 않고, 양쪽 모두에 명확하게 연결고리가 있는 데이터만을 짝지어 보여줌

6. 쿼리로 확인하기
   - 내부 조인을 사용했을 때 교집합에 존재하는 user_id : {1, 2, 3, 4, 5} 만 선택되었는지 쿼리로 확인해보자.
```sql
SELECT
    orders.order_id,
    orders.order_date,
    orders.user_id AS orders_user_id,
    users.user_id AS users_user_id,
    users.name
FROM orders
INNER JOIN users ON orders.user_id = users.user_id
ORDER BY orders.order_id;
```
  - 이 쿼리는 orders 테이블과 users 테이블의 user_id를 기준으로 교집합을 찾은 다음(INNER JOIN), 최종적으로 요청한 컬럼을 선택(SELECT)한 것
  - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/baa13f7d-b0dc-4a66-83bb-c1d3199fe91e">
</div>

   - user_id가 양쪽에 모두 있는 {1, 2, 3, 4, 5}가 선택
   - 주문 기록이 없는 user_id:6 ('레오나르도 다빈치')는 포함되지 않은 것을 다시 한번 확인할 수 있음
   - 이처럼 조인을 집합의 관점에서 이해하면, 벤 다이어그램의 어떤 부분을 결과로 가져오는지 훨씬 직관적으로 파악할 수 있게 됨

7. 내부 조인과 조인 방향
   - 내부 조인은 양방향 :A 테이블과 B 테이블이 있다고 하면, A → B로 조인할 수 있다면 반대로 B → A로 조인할 수 있음
   - 그리고 그 결과는 항상 동일 : 내부 조인은 두 테이블 간의 교집합을 찾는 연산이기 때문이므로, A와 B의 교집합 과 B와 A의 교집합 이 같은 것과 같은 원리
<div align="center">
<img src="https://github.com/user-attachments/assets/2fe21023-15df-4d01-9bce-a704596cddf5">
</div>

   - orders가 왼쪽, users가 오른쪽
   - 교집합은 {1, 2, 3, 4, 5}

<div align="center">
<img src="https://github.com/user-attachments/assets/53dd6972-5369-450b-bac2-8dd7e049beb7">
</div>

   - users가 왼쪽, orders가 오른쪽
   - 교집합은 {1, 2, 3, 4, 5}
   - 이런 이유로 교집합을 확인하는 내부 조인은 A → B로 조인하든 B → A로 조인하든 결과가 항상 같음

8. 그림으로 확인하기
<div align="center">
<img src="https://github.com/user-attachments/assets/a0e1f8b2-53f4-4b54-aea2-95739ab25d3a">
</div>

   - 데이터베이스는 orders 테이블과 users 테이블을 조회
   - orders → users 조인: orders 테이블을 한 줄씩 읽으며, 각 주문의 user_id에 해당하는 users 정보를 찾아 옆에 붙임
<div align="center">
<img src="https://github.com/user-attachments/assets/b1d7d340-cde9-430e-b083-a1c322483e73">
</div>

   - 데이터베이스는 users 테이블과 orders 테이블을 조회 (그림에서 화살표의 방향이 반대인 것을 확인)
   - users → orders 조인: users 테이블을 한 줄씩 읽으며, 각 사용자의 user_i 와 일치하는 모든 orders 정보를 찾아 옆에 붙임
      + 예를 들어 users의 user음
