-----
### 스칼라 서브쿼리
-----
1. 가장 먼저 배울 것은 단일 컬럼, 단일 행 서브쿼리
   - 이름에서 알 수 있듯이, 서브쿼리를 실행했을 때 그 결과가 오직 하나의 행, 하나의 컬럼으로 나오는 경우
   - 서브쿼리의 결과가 하나의 값으로 정해지기 때문에 이것을 스칼라 서브쿼리

2. 용어 - 스칼라
   - 스칼라(Scalar)는 원래 수학과 물리학에서 온 단어로, '단 하나의 값'을 의미
   - 결과가 '하나의 값'으로 정해지기 때문에, 우리는 이 값을 익숙한 단일 행 비교 연산자들(=, >, <, >=, <=, <>)과 함께 사용할 수 있음
   - "A는 B보다 크다"처럼, 비교 대상 B가 하나의 명확한 값이어야 말이 되는 것과 같은 이치

3. 특정 주문(order_id = 1)을 한 고객과 같은 도시에 사는 모든 고객을 찾고 싶을 경우
   - 서브쿼리 없이 2단계로 해결하기
   - 1단계 : order_id가 1인 고객의 도시 알아내기
      + order_id가 1인 주문을 찾고, 그 주문을 한 고객의 user_id를 알아낸 뒤, users 테이블과 조인하여 그 고객의 주소(도시)를 알아내야 함
```sql
SELECT u.address
FROM users u
JOIN orders o ON u.user_id = o.user_id
WHERE o.order_id = 1;
```
   - orders 테이블을 보면, order_id 1번은 user_id 1번('션')이 주문
   - users 테이블에서 '션'의 주소는 '서울시 강남구'
<div align="center">
<img src="https://github.com/user-attachments/assets/dfa9588e-4133-4e31-8f3e-164e06567bec">
</div>

   - 주문자 '션'이 '서울시 강남구'에 산다는 것을 알아냈음

   - 2단계 : 해당 도시에 사는 모든 고객 찾기
     + 이제 위에서 얻은 '서울시 강남구'라는 값을 이용해 users 테이블에서 동일한 주소를 가진 고객을 모두 찾음
```sql
SELECT name, address
FROM users
WHERE address = '서울시 강남구';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/ffd2753a-ebe4-4148-b112-44347c396f97">
</div>

   - 결과를 보면 주문을 한 '션'뿐만 아니라, 같은 '서울시 강남구'에 사는 '마리 퀴리'도 함께 조회되는 것을 확인할 수 있음
   - 이렇게 우리는 2단계에 걸쳐 원하는 결과를 얻음

4. 단일 행 서브쿼리로 해결하기
   - 1단계 쿼리가 "비교할 기준값(도시 이름)을 찾아내는" 역할을 하므로, 이 쿼리를 통째로 서브쿼리로 만들어 WHERE 절의 비교 대상 위치에 넣어주면 됨
```sql
SELECT name, address
FROM users
WHERE address = (SELECT u.address
                 FROM users u
                 INNER JOIN orders o ON u.user_id = o.user_id
                 WHERE o.order_id = 1);
```
<div align="center">
<img src="https://github.com/user-attachments/assets/2676f6e3-1858-4f41-abaa-51afc6853539">
</div>

5. 쿼리 실행 흐름
   - 괄호 안의 서브쿼리가 먼저 실행되어 단일 값 '서울시 강남구' 를 반환
   - 메인쿼리는 WHERE address = '서울시 강남구' 와 동일한 형태로 바뀜
   - 최종적으로 메인쿼리가 실행되어 우리에게 원하는 결과를 보여줌 :  결과는 당연히 2단계로 나누어 실행했을 때와 동일
   - 이처럼 서브쿼리를 사용하면 여러 번의 쿼리를 하나로 합쳐 코드를 간결하게 만들고, 애플리케이션과 데이터베이스 간의 통신 횟수를 줄여 성능상 이점을 얻을 수 있음

6. 스칼라 서브쿼리의 치명적 오류
   - 단일 행을 반환하는 스칼라 서브쿼리를 사용할 때 가장 주의해야 할 점은, 서브쿼리의 결과가 반드시, 무슨 일이 있어도, 단 하나의 행만 반환해야 한다는 것
   - 만약 서브쿼리가 두 개 이상의 행을 반환하면 어떻게 될까?
    + 데이터베이스는 address = ('결과1', '결과2') 와 같은 형태의 비교를 이해할 수 없음
    + 하나의 값과 비교해야 하는 = 연산자는 비교 대상이 여러 개 들어오면 무엇을 해야 할지 알 수 없게 됨
   - 예를 들어, "서울 또는 성남에 사는 고객의 주소를 모두 조회"하는 서브쿼리를 만들어 비교한다고 가정
```sql
-- 이 쿼리는 의도적으로 오류를 발생
SELECT name, address
FROM users
WHERE address = (SELECT address FROM users WHERE name IN ('션', '네이트'));
```
   - 괄호 안의 서브쿼리 SELECT address FROM users WHERE name IN ('션', '네이트') 는 '서울시 강남구'와 '경기도 성남시'라는 두 개의 행을 반환
   - 메인쿼리의 WHERE address = ... 비교문은 이 두 개의 값을 어떻게 처리해야 할지 몰라 혼란에 빠지므로, 결국, 데이터베이스는 이런 에러를 발생시키며 실행을 멈춤
```
Error Code: 1242. Subquery returns more than 1 row
```

   - 이처럼 단일 행 비교 연산자(=, >, < 등)를 사용할 때는, 서브쿼리의 결과가 반드시 단일 행일때만 사용
   - 예를 들어 order_id나 user_id처럼 PK나 UNIQUE 제약 조건이 걸린 컬럼을 조건으로 조회하는 경우가 대표적
   - 이때는 = 같은 단일 행 연산자가 아닌, 여러 개의 값을 다룰 수 있는 IN 과 같은 다중 행 연산자를 사용해야 함
