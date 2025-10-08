-----
### 외부 조인
-----
1. 내부 조인( INNER JOIN )을 통해 우리는 양쪽 테이블에 모두 존재하는, 짝이 맞는 데이터들을 성공적으로 연결
   - 하지만 실무에서는 종종 짝이 없는, 소외된 데이터를 찾아야 할 때가 있음
2. 실습 : '한 번도 주문하지 않은 고객' 찾기
   - '한 번도 주문하지 않은 고객'을 찾는 문제를 단계별로 접근
   - 1단계 : 각 테이블 개별 조회하기
      + 먼저 users 테이블과 orders 테이블을 각각 조회해서 어떤 고객이 있고, 어떤 주문이 있는지 직접 확인
```sql
-- users 테이블 조회 (최소한의 필드만)
SELECT user_id, name, email
FROM users;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/53c96f96-5425-4f15-a56f-7eff1f23b383">
</div>

```sql
-- orders 테이블 조회 (최소한의 필드만)
SELECT user_id, order_id
FROM orders
order by user_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/3dd89298-a4f5-4e41-be10-69b6541acd5d">
</div>

   - users 테이블에는 user_id가 {1, 2, 3, 4, 5, 6} 인 고객이 있음
   - 그런데 orders 테이블에는 user_id가 {1, 2, 3, 4, 5}인 주문만 있고, user_id가 6인 주문은 없음
   - 즉, '레오나르도 다빈치'는 가입은 했지만 한 번도 주문하지 않은 고객

   - 2단계 : 내부 조인으로 시도해보기
      + 결과를 한 번에 쉽게 확인하기 위해 내부 조인( INNER JOIN )을 사용해서 고객과 주문을 연결
```sql
SELECT
     u.user_id,
     u.name,
     o.user_id,
     o.order_id
FROM users u
JOIN orders o ON u.user_id = o.user_id
ORDER BY u.user_id;
```
   - INNER는 생략 : INNER를 생략하고 JOIN만 적으면 내부 조인(INNER JOIN)으로 작동
<div align="center">
<img src="https://github.com/user-attachments/assets/d2e3ec04-4b2c-4a46-a191-72f7bad2827d">
</div>

   - 조인 과정을 보면 레오나르도 다빈치의 users.user_id:6에 해당하는 orders.user_id:6이 없음
   - 따라서 레오나르도 다빈치는 조인 대상에서 제외
<div align="center">
<img src="https://github.com/user-attachments/assets/2de2564a-0c96-4f99-8ed5-81f79a40000d">
</div>

   - 결과를 보면 '레오나르도 다빈치'(user_id : 6)가 결과에서 완전히 사라짐
   - 내부 조인(INNER JOIN)은 양쪽 테이블에 모두 존재하는 데이터만 보여주기 때문임 : 즉, 주문 기록이 없는 고객은 orders 테이블에 짝이 없어서 결과에서 제외

   - 외부 조인(OUTER JOIN)의 필요성
     + 내부 조인(INNER JOIN)으로는 주문 기록이 없는 고객을 찾을 수 없음
     + 이런 경우에는 외부 조인(OUTER JOIN)이 필요 : 외부 조인을 사용하면 한쪽 테이블에만 존재하는 데이터도 결과에 포함시킬 수 있음
     + 이처럼 한쪽에는 데이터가 있지만, 다른 한쪽에는 없는 데이터까지 모두 포함해서 보고 싶을 때 사용하는 기술이 바로 외부 조인

3. 외부 조인의 필요성 및 개념
  - 조인은 본질적으로 두 테이블의 집합에 대한 이야기 : 그 중에 내부 조인은 교집합
<div align="center">
<img src="https://github.com/user-attachments/assets/32d05bab-984f-4cd5-9b1f-07e96ba8eae4">
</div>

   - 내부 조인은 교집합 영역이기 때문에 users에만 속한 '레오나르도 다빈치'(user_id : 6)를 결과에 포함할 수 없음
   - 💡 외부 조인의 필요성 : 외부 조인(OUTER JOIN)은 두 테이블을 조인할 때, 특정 테이블의 데이터는 ON 조건이 맞지 않더라도 모두 결과에 포함시키는 방법
<div align="center">
<img src="https://github.com/user-attachments/assets/bdf13d8f-5d43-445f-b843-d6bd651a7230">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/29da096b-7c68-490d-ad0b-88825fc21c26">
</div>

   - 💡 이때 기준이 되는 테이블이 어느 쪽이냐에 따라 LEFT OUTER JOIN과 RIGHT OUTER JOIN으로 나뉨
     + 그리고 교집합 영역은 물론이고, 기준이 되는 테이블의 데이터는 결과에 모두 포함
     + 레오나르도 다빈치'(user_id : 6)를 결과에 포함하려면 LEFT OUTER JOIN을 선택하면 됨

4. 용어 - 외부 조인 (OUTER JOIN)
   - 내부 조인(INNER JOIN)은 교집합 안(INNER)을 의미
   - 외부 조인은 교집합 밖(OUTER)의 영역을 포함한다는 의미

5. 풀 외부 조인(FULL OUTER JOIN)
   - 양쪽 모두를 함께 포함하는 풀 외부 조인이라는 방법
   - 실무에서 잘 사용하지 않고, MySQL은 지원하지 않음
<div align="center">
<img src="https://github.com/user-attachments/assets/2751b637-2a1e-4631-aad8-2620c9301579">
</div>

6. 💡 LEFT OUTER JOIN vs RIGHT OUTER JOIN
   - 외부 조인의 핵심은 '기준 테이블'을 정하는 것
     + LEFT OUTER JOIN은 왼쪽 테이블을, RIGHT OUTER JOIN은 오른쪽 테이블을 기준으로 삼는 것 (OUTER는 생략 가능)
     + LEFT OUTER JOIN → LEFT JOIN
     + RIGHT OUTER JOIN → RIGHT JOIN

   - LEFT JOIN (또는 LEFT OUTER JOIN ) : LEFT JOIN 구문의 왼쪽(FROM 절)에 있는 테이블이 기준
      + 일단 왼쪽 테이블의 모든 데이터를 결과에 포함시킴
      + 그다음, ON 조건에 맞는 데이터를 오른쪽 테이블에서 찾아 옆에 붙여줌
      + 만약 오른쪽 테이블에 짝이 맞는 데이터가 없다면, 그 자리는 NULL 값으로 채워짐

   - RIGHT JOIN (또는 RIGHT OUTER JOIN ) : RIGHT JOIN 구문의 오른쪽(JOIN 절)에 있는 테이블이 기준
      + 일단 오른쪽 테이블의 모든 데이터를 결과에 포함시킴
      + 그다음, ON 조건에 맞는 데이터를 왼쪽 테이블에서 찾아 붙여줌
      + 마찬가지로 왼쪽 테이블에 짝이 맞는 데이터가 없다면, 그 자리는 NULL 로 채워짐

7. 실습 1 : LEFT JOIN 으로 '한 번도 주문하지 않은 고객' 찾기
   - 첫 번째 질문, "한 번도 주문하지 않은 고객은 누구인가?" : 이 질문의 기준은 '고객'이므로, 따라서 users 테이블이 LEFT JOIN 의 왼쪽에 위치해야 함

   - 1단계 : users 를 기준으로 orders 테이블 LEFT JOIN 하기
      + 먼저 users 테이블의 모든 고객을 일단 다 보여주고, 오른쪽에 주문 정보를 붙여야 함
```sql
SELECT
     u.user_id,
     u.name,
     o.user_id,
     o.order_id
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
ORDER BY u.user_id;
```
   - LEFT JOIN 사용 : users가 왼쪽이고, orders가 오른쪽
   - SQL을 보면 FROM 다음에 users가 먼저 나오고 그 다음에 orders가 나옴
<div align="center">
<img src="https://github.com/user-attachments/assets/e4919d0a-fd22-4e7a-b36d-d04987628ffd">
</div>

   - INNER JOIN 에서는 보이지 않던 '레오나르도 다빈치(user_id:6)'가 나타남
   - 주문 기록이 없기 때문에, orders 테이블에서 가져온 user_id와 order_id 컬럼의 값이 모두 NULL로 채워져 있음

<div align="center">
<img src="https://github.com/user-attachments/assets/110066b9-5dd1-4be9-8f4f-9bb460e72786">
</div>

   - LEFT JOIN은 왼쪽에 있는 기준이 되는 테이블(users)의 모든 행을 포함
   - 참고로 여기서 레오나르도 다빈치(user_id:6)는 orders 에 조인할 대상이 없음
<div align="center">
<img src="https://github.com/user-attachments/assets/cde8ad37-a7d1-4a25-8a44-1943b25f12fe">
</div>

   - 이렇게 조인 대상이 없는 경우에는 나머지 값을 NULL로 채우면서 조인 결과에 포함
     + 💡 OUTER JOIN은 기준 테이블의 데이터를 모두 포함

   - 2단계 : NULL인 데이터만 필터링하기
     + 주문 정보가 NULL인 고객이 바로 '한 번도 주문하지 않은 고객'
     + WHERE 절을 사용해서 이들만 찾아보기
```sql
SELECT
   u.user_id,
   u.name,
   u.email
FROM users AS u
LEFT JOIN orders AS o ON u.user_id = o.user_id
WHERE o.order_id IS NULL;
```
   - NULL 비교 : NULL은 '값이 없음'을 나타내는 특별한 상태이므로, = 연산자로 비교할 수 없음
     + 반드시 IS NULL 또는 IS NOT NULL 을 사용해야 함함
