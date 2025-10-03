-----
### DML - 삭제
-----
1. DELETE FROM 은 테이블에서 행을 삭제하는 명령어
   - DELETE 문의 기본 문법
```sql
DELETE FROM table_name
WHERE condition;
```
   - table_name : 데이터를 삭제할 테이블의 이름
   - WHERE condition : 삭제할 행을 식별하는 조건 (UPDATE와 마찬가지로 이 부분을 생략하면 테이블의 모든 데이터가 삭제되는 문제가 발생하므로 반드시 작성해야 함)

2. DELETE 기본 구문
  - '베이직 반팔 티셔츠' 상품이 단종되어 쇼핑몰에서 더 이상 판매하지 않기로 했다고 가정
```sql
SELECT * FROM products
WHERE product_id = 1; -- 상품 ID가 1번이라고 가정
```
<div align="center">
<img src="https://github.com/user-attachments/assets/aa66a7d3-ee24-4083-a1b3-d7bb9a9fd054">
</div>

   - DELETE FROM으로 삭제
```sql
DELETE FROM products
WHERE product_id = 1; -- 상품 ID가 1번이라고 가정
```

   - 실행 결과
```sql
SELECT * FROM products;
```

<div align="center">
<img src="https://github.com/user-attachments/assets/e841419c-95ab-48de-a90e-ec70375ba31e">
</div>

   - product_id=1 베이직 반팔 티셔츠 상품이 제거된 것을 확인할 수 있음

3. WHERE 절의 절대적인 중요성
   - DELETE는 UPDATE 보다 더 파괴적
   - UPDATE는 데이터를 잘못 바꿔도 원래 값을 추적할 여지라도 있지만, DELETE 는 행 자체를 없애버림
```sql
-- 절대로 따라하지 말 것!
DELETE FROM customers;
```

   - 실행 결과
```sql
SELECT * FROM customers;
```

<div align="center">
<img src="https://github.com/user-attachments/assets/77b56f5a-5d51-4873-9017-3c67d56d280b">
</div>

  - UPDATE와 마찬가지로, DELETE 역시 실행 전 SELECT 문으로 삭제 대상을 확인하는 것이 철칙

4. 전체 UPDATE, DELETE 를 실행했을 때 벌어지는 끔찍한 상황과 대처법
  - 사고 발생 : WHERE 절 없는 UPDATE를 실행했다는 사실을 깨달음
  - 즉시 보고 및 서비스 점검 : 즉시 DBA를 호출하고, 상급자에게 상황을 보고
    + 가능하다면 추가적인 피해를 막기 위해 서비스를 점검 상태로 전환
    + 전문적인 지식을 갖춘 DBA가 있다면 데이터베이스 복원 기능으로 빠르게 복구할 수도 있음
  - 백업 확인 : 가장 먼저 확인해야 할 것은 가장 최근의 데이터베이스 백업
    + 정기적인 백업이 있었다면, 백업 시점 이후의 데이터 유실을 감수하고 복구(Restore)를 진행할 수 있음
  - 로그 분석: 백업이 없다면, 데이터베이스 로그를 분석하여 변경 이전의 값을 추적하는 복잡한 작업을 시도해 볼 수 있으나, 이는 매우 어렵고 시간이 오래 걸림

5. DELETE vs TRUNCATE
<div align="center">
<img src="https://github.com/user-attachments/assets/98aed265-f7be-40d7-a8e4-3a4eea2e275e">
</div>

   - '탈퇴한 회원 한 명의 정보만 지우고 싶다' 또는 '특정 조건에 맞는 주문 기록만 삭제하고 싶다'와 같이 선별적인 삭제가 필요할 때는 DELETE 를 사용해야 함 : 일반적인 비즈니스 로직은 항상 DELETE를 사용
   - ' 테스트용으로 넣었던 수백만 건의 데이터를 모두 지우고 처음부터 다시 시작하고 싶다' 와 같이 테이블의 모든 데이터를 깨끗하게 비울 목적이라면 TRUNCATE가 훨씬 빠르고 효율적인 선택
