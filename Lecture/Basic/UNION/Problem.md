-----
### 문제와 풀이
-----
1. 문제 1 : 전체 고객 목록 조회하기
   - 쇼핑몰의 모든 고객(활동 고객과 탈퇴 고객)의 이름과 이메일을 중복 없이 조회하여 하나의 목록으로 만들기
<div align="center">
<img src="https://github.com/user-attachments/assets/4c22a57b-50ce-4d07-b4ba-1fd5b49ecb20">
</div>

```sql
SELECT name AS 이름, email AS 이메일
FROM users

UNION

SELECT name, email
FROM retired_users;
```

2. 💡 문제 2 : 특별 이벤트 대상자 목록 만들기 (중복 포함)
   - 마케팅팀에서 두 그룹의 고객에게 이벤트를 진행하려고 함
     + '전자기기' 카테고리 상품을 한 번이라도 구매한 고객
     + 한 번의 주문으로 상품을 2개 이상 구매한 고객
   - 두 그룹에 모두 해당하는 고객도 있을 수 있음
   - 성능을 고려하여, 두 그룹의 목록을 중복 제거 없이 모두 합쳐서 조회
   - 컬럼 별칭은 고객명 , 이메일로 지정
```sql
-- 전자기기 구매 고객
SELECT DISTINCT u.name AS 고객명, u.email AS 이메일
FROM users u
INNER JOIN orders o ON u.user_id = o.user_id
INNER JOIN products p ON o.product_id = p.product_id
WHERE p.category = '전자기기'

UNION ALL

-- 한 번에 2개 이상 구매한 고객
SELECT DISTINCT u.name AS 고객명, u.email AS 이메일
FROM users u
INNER JOIN orders o ON u.user_id = o.user_id
WHERE o.quantity > 1;
```
   - 전자기기 구매 고객의 중복을 DISTINCT로 제거

3. 문제 3 : 회사 주요 이벤트 타임라인 만들기
   - 회사의 주요 이벤트 타임라인을 만들려고 함 : '고객 가입' 이벤트와 '상품 주문' 이벤트를 시간 순서대로 정렬하여 조회
      + users 테이블의 created_at 은 '고객 가입' 이벤트로 간주
      + orders 테이블의 order_date 는 '상품 주문' 이벤트로 간주
      + 결과는 이벤트_날짜, 이벤트_종류, 상세_내용 컬럼으로 구성
      + 상세_내용 에는 고객 가입 시 고객의 이름, 상품 주문 시 상품의 이름을 표시
      + 최신 이벤트가 가장 위에 오도록 내림차순으로 정렬
<div align="center">
<img src="https://github.com/user-attachments/assets/d1971817-5ba7-4b4b-b5ea-ba239c5d4f10">
</div>

   - 고객 가입 이벤트의 날짜는 쿼리 실행 시점의 created_at 값에 따라 달라질 수 있음
```sql
SELECT
   created_at AS 이벤트_날짜,
   '고객 가입' AS 이벤트_종류,
   name AS 상세_내용
FROM users

UNION ALL

SELECT
   o.order_date AS 이벤트_날짜,
   '상품 주문' AS 이벤트_종류,
   p.name AS 상세_내용
FROM orders o
INNER JOIN products p ON o.product_id = p.product_id

ORDER BY 이벤트_날짜 DESC;
```

4. 문제 4 : 회사 전체 인명록 만들기
   - 회사의 모든 관련 인물(users의 고객과 employees의 직원)을 통합한 인명록을 제작
   - 결과에는 이름, 역할 ('고객' 또는 '직원'), 이메일 컬럼이 포함
   - 고객의 연락처는 email 컬럼을 사용
   - 직원의 연락처는 name 컬럼 뒤에 '@my-shop.com' 을 붙여서 생성
   - 최종 결과는 이름(가나다/알파벳 순)으로 오름차순 정렬
<div align="center">
<img src="https://github.com/user-attachments/assets/f6b99719-f047-439a-99b8-f451a8723167">
</div>

```sql
SELECT
   name AS 이름,
   '고객' AS 역할,
   email AS 이메일
FROM users

UNION ALL

SELECT
   name AS 이름,
   '직원' AS 역할,
   CONCAT(name, '@my-shop.com') AS 이메일
FROM employees

ORDER BY 이름;
```
