-----
### CROSS 조인
-----
1. INNER , OUTER , SELF 조인 : 모두 ON 이라는 연결고리를 통해 테이블에 '이미 존재하는' 관계를 찾아내는 작업
   - 즉, 특정 조건에 맞는 짝을 찾는 것이 핵심

2. 쇼핑몰에서 새로운 기본 티셔츠를 판매하려고 하며, 사이즈는 S, M, L, XL 네 종류이고, 색상은 Red, Blue, Black 세 종류로, 판매를 시작하기 전에, 재고 관리를 위해 가능한 모든 사이즈와 색상 조합을 담은 상품 마스터 데이터를 미리 생성하고 싶을 경우
   - 이 업무는 'S 사이즈이면서 Red 색상인 상품', 'M 사이즈이면서 Red 색상인 상품' ... 'XL 사이즈이면서 Black 색상인 상품' 까지, 생각할 수 있는 모든 경우의 수를 목록으로 만들어야 함
   - sizes 테이블과 colors 테이블 사이에는 미리 정해진 연결고리(ON 조건)가 없음 : 둘을 강제로 조합해야 한다.
   - 이럴 때 사용하는 조인이 바로 CROSS JOIN

3. CROSS JOIN의 개념과 카테시안 곱(Cartesian Product)
   - CROSS JOIN은 조인 조건 없이, 한쪽 테이블의 모든 행을 다른 쪽 테이블의 모든 행과 하나씩 전부 연결하는, 가장 단순한 조인 방식
   - 연결고리가 없기 때문에 ON 절을 사용하지 않음
   - CROSS JOIN의 결과를 수학 용어로 카테시안 곱(Cartesian Product) 또는 데카르트 곱
     + 만약 A 테이블에 m개의 행이 있고, B 테이블에 n개의 행이 있다면, 두 테이블을 CROSS JOIN 한 결과는 m * n 개의 행을 갖게 됨

   - 우리 예시의 sizes 테이블은 4개의 행을, colors 테이블은 3개의 행을 가지고 있음
     + 따라서 두 테이블을 CROSS JOIN 하면 4 * 3 = 12 개의 행으로 이루어진 결과가 나올 것

4. 실습 : 상품 옵션 조합 만들기
   - 이제 CROSS JOIN 을 사용해 티셔츠의 모든 사이즈와 색상 조합을 만들어보기
   - sizes 테이블과 colors 테이블의 데이터를 확인
```sql
SELECT *
FROM sizes;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/49928b1f-f725-4ebb-968e-3ae34867e0aa">
</div>

```sql
SELECT *
FROM colors;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/495f283b-cbfd-4ec2-abcf-f0dd403583a5">
</div>

   - 이제 두 테이블을 CROSS JOIN 하여 모든 조합을 생성
```sql
SELECT
   s.size,
   c.color
FROM
   sizes s
CROSS JOIN
   colors c;
```
   - ON 절이 없는 것을 다시 한번 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/23238bb5-10b9-4fc8-b17d-3a7339148b6e">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/3febc975-c7f8-4e15-92d9-d1d4ac219c5d">
</div>

   - sizes 테이블의 각 행이 colors 테이블의 모든 행과 하나씩 짝을 이뤄 총 4 x 3, 12개의 완벽한 조합이 만들어짐
   - 여기서 한 걸음 더 나아가, 이 조합을 이용해 상품명을 자동으로 생성해 볼 수도 있음 : 문자열을 합치는 CONCAT 함수를 활용하면 됨
```sql
SELECT
   CONCAT('기본티셔츠-', c.color, '-', s.size) AS product_name,
   s.size,
   c.color
FROM
   sizes AS s
CROSS JOIN
   colors AS c;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/ff53e80a-80c2-4bd3-8164-0afe6b46593b">
</div>

   - 이 결과를 새로운 테이블에 INSERT 하면, 상품 마스터 데이터를 손쉽게 구축할 수 있음

5. INSERT INTO ... SELECT 로 상품 옵션 마스터 테이블 만들기
   - 이제 CROSS JOIN 으로 만든 상품 옵션 조합을 새로운 테이블에 저장해 보기
   - 이렇게 데이터를 한 번에 대량으로 삽입 할 때 INSERT INTO ... SELECT 구문을 사용
      + 티셔츠 상품에 대한 모든 사이즈와 색상 조합을 저장할 product_options 테이블을 만들 것
      + 이 테이블은 option_id, product_name, size, color 필드를 가진다.
```sql
CREATE TABLE product_options (
   option_id BIGINT AUTO_INCREMENT,
   product_name VARCHAR(255) NOT NULL,
   size VARCHAR(10) NOT NULL,
   color VARCHAR(20) NOT NULL,

   PRIMARY KEY (option_id)
);
```
   - 테이블이 준비되었으니, 이제 CROSS JOIN의 결과를 product_options 테이블에 INSERT 
```sql
INSERT INTO product_options (product_name, size, color)
SELECT
   CONCAT('기본티셔츠-', c.color, '-', s.size) AS product_name,
   s.size,
   c.color
FROM
   sizes AS s
CROSS JOIN
   colors AS c;
```
   - 💡 INSERT INTO ... SELECT : SELECT 문으로 조회된 결과를 즉시 다른 테이블에 삽입하는 기능
     + 이 방법을 사용하면 대량의 초기 데이터를 효율적으로 구축하거나, 기존 데이터를 가공하여 새로운 테이블에 저장하는 등의 작업을 할 수 있음

   - 이제 product_options 테이블에 데이터가 잘 들어갔는지 확인
```sql
SELECT * FROM product_options;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/1b5598a5-7132-403a-8ab2-6d9d8bac5b23">
</div>

   - CROSS JOIN으로 생성한 12개의 상품 옵션 조합이 product_options 테이블에 성공적으로 삽입된 것을 확인할 수 있음
   - 이처럼 CROSS JOIN은 단순히 데이터를 조합하는 것을 넘어, 새로운 데이터 세트를 만들거나 초기 시스템을 구축할 때 유용하게 활용될 수 있음
   - 특히 상품의 다양한 옵션(사이즈, 색상, 재질 등)을 조합하여 마스터 데이터를 생성하는 시나리오에서 좋은 도구가 됨

6. 실무에서의 치명적인 주의사항
   - CROSS JOIN은 모든 경우의 수를 만들어주기 때문에 유용하지만, 실무에서는 아주 신중하게 사용해야 하는, 어떻게 보면 가장 위험한 조인이기도 함
   - 왜냐하면 결과의 행 수가 기하급수적으로 늘어날 수 있기 때문임
     + 만약 당신이 실수로 100만 건의 데이터가 있는 users 테이블과 10만 건의 데이터가 있는 products 테이블을 CROSS JOIN 한다면, 결과는 100만 * 10만 = 1000억 건이 됨
     + 이 쿼리를 실행하는 순간 데이터베이스 서버는 아마 응답을 멈추거나 다운될 것
