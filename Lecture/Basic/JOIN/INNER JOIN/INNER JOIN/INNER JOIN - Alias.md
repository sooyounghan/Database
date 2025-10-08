-----
### 내부 조인 순서
-----
1. 실무 팁: 조인 순서는 언제 중요할까?
   - 내부 조인에서는 결과가 같으므로 어떤 순서로 작성해도 무방
   - 하지만 쿼리를 읽는 사람의 입장에서 어떤 데이터가 중심이 되는가에 따라 순서를 정하면 가독성이 높아짐
      + 주문 목록을 중심으로 고객 정보를 추가하고 싶다면, FROM orders JOIN users
      + 고객 목록을 중심으로 주문 정보를 조회하고 싶다면, FROM users JOIN orders

2. 이렇게 같이 작성하는 것이 논리의 흐름을 이해하기 더 쉬움
3. 나중에 배울 OUTER JOIN 에서는 조인 순서가 결과에 매우 큰 영향을 미치지만, INNER JOIN에서는 교집합을 선택하기 때문에 순서와 상관없이 결과는 항상 음
4. 조인은 우리가 원하는 데이터가 여러 테이블에 흩어져 있을 때, 그 정보들을 하나로 모으는 가장 기본적인 방법

-----
### 가독성을 높이는 테이블 별칭(Alias)
-----
1. 앞선 쿼리들을 보면 users.user_id , orders.status 처럼 테이블 이름을 계속 반복해서 작성해야 했는데, 쿼리가 길어지고 복잡해질수록 이는 매우 번거롭고 가독성을 떨어뜨림
2. 이럴 때 사용하는 것이 바로 테이블 별칭(Alias) : AS 키워드를 사용하거나, AS를 생략하고 한 칸 띄우고 원하는 별칭을 붙여주면 됨
```sql
SELECT
   u.user_id,
   u.name,
   o.order_date
FROM orders AS o
INNER JOIN users AS u ON o.user_id = u.user_id
WHERE o.status = 'COMPLETED';
```
   - orders AS o : orders 테이블에 o라는 별칭
   - users AS u : users 테이블에 u라는 별칭
   - 이제 orders.status 대신 o.status, users.name 대신 u.name과 같이 간결하게 컬럼을 지정할 수 있음

3. 실무에서는 AS 생략
   - 실무에서는 테이블 별칭을 붙일 때 AS 키워드를 생략하는 경우가 더 많음
   - 테이블 별칭의 AS 를 생략하고, 추가로 INNER JOIN 의 INNER 도 생략했다.
```sql
SELECT
     u.user_id,
     u.name,
     o.order_date
FROM orders o
JOIN users u ON o.user_id = u.user_id
WHERE o.status = 'COMPLETED';
```
   - FROM orders o 처럼 테이블 이름 뒤에 한 칸 띄고 별칭을 적는 것만으로도 충분
   - 이 방식이 더 간결하기 때문에 실무 개발자들이 선호하는 스타일
   - 앞으로의 모든 예제에서는 이 방식을 사용할 것
   - INNER JOIN에서 INNER는 생략 가능하므로 JOIN만 사용해도 됨 (그냥 JOIN 이라고 하면 자동으로 INNER JOIN이 됨)
   - 실무에서는 대부분 INNER JOIN을 사용할 때 INNER는 생략하고 JOIN만 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/f8ae3641-07dc-4de8-9c47-e5f849fb0fd9">
</div>

   - 별칭을 사용해도 결과는 당연히 동일

4. 실무 팁 - 테이블과 컬럼의 별칭 AS 생략
   - 테이블 별칭 : AS 생략
      + 테이블 이름 뒤에 오는 별칭은 문법적으로 혼동의 여지가 거의 없음
      + FROM table_name alias_name과 같이 AS 없이 써도 코드를 읽고 해석하는 데 문제가 되지 않음
      + 따라서 코드를 더 간결하게 만들기 위해 AS를 생략하는 경우가 많음
      + 예를 들어, FROM employees e는 FROM employees AS e와 동일하게 작동하며, 더 짧고 깔끔하게 보임

   - 컬럼 별칭 : AS 사용
      + 반면, 컬럼 목록에서는 AS 를 생략하면 어떤 것이 원래 컬럼 이름이고 어떤 것이 별칭인지 즉시 파악하기 어려울 수 있음
      + 특히 여러 컬럼을 나열하거나 복잡한 함수를 사용할 때 AS 를 명시적으로 써주면 코드의 의도를 명확하게 전달하여 가독성을 크게 향상시킴
    - 가독성이 낮은 예시
```sql
SELECT salary * 12 annual_salary
```
   - 가독성이 높은 예시
```sql
SELECT salary * 12 AS annual_salary
```
   - 이처럼 AS를 사용하면 salary * 12의 결과를 annual_salary라는 이름으로 부른다는 의미가 훨씬 명확해짐
   - 이러한 관행은 필수는 아니지만 많은 개발자가 코드의 가독성과 유지보수성을 높이기 위해 따르는 일종의 코딩 컨벤션(약속)
     
5. 실무 팁 정리
   - 테이블에 별칭으로 사용하는 AS는 주로 생략
   - 컬럼에 별칭으로 사용하는 AS는 가독성을 위해 생략하지 않고 사용
