-----
### 복합 인덱스
-----
1. 예) "카테고리가 '전자기기'인 상품들 중에 가격이 100,000원 이상인 상품을 출력"과 같은 요구사항
```sql
SELECT *
FROM items
WHERE category = '전자기기' AND price >= 100000
```
   - 이런 다중 조건 쿼리의 성능을 최적화하기 위해 사용하는 것이 바로 복합 인덱스(Composite Index) 또는 다중 컬럼 인덱스(Multi-column Index)

2. 복합 인덱스는 이름 그대로 두 개 이상의 컬럼을 묶어서 하나의 인덱스로 만드는 것
   - 💡 하지만 복합 인덱스를 제대로 사용하려면 한 가지 매우 중요한 규칙을 이해해야 하는데, 바로 '컬럼의 순서'

3. 왜 컬럼 순서가 중요할까?
   - 복합 인덱스의 동작 원리는 우리가 실생활에서 사용하는 '전화번호부'나 '국어사전'과 똑같음
     + 전화번호부 : '성(Last Name)'으로 먼저 정렬된 후, 같은 성 안에서 '이름(First Name)'으로 다시 정렬
     + 국어사전 : '첫 번째 글자'로 먼저 정렬된 후, 같은 첫 글자로 시작하는 단어들끼리 '두 번째 글자'로 다시 정렬

   - items 테이블에 (category, price) 순서로 복합 인덱스를 만들었다고 상상
     + 이 인덱스는 내부적으로 다음과 같이 정렬
     + category를 기준으로 먼저 정렬 ('도서', '생활용품', '전자기기', '패션', '헬스/뷰티' 순서)
     + 같은 category 내에서는 price 를 기준으로 다시 정렬
<div align="center">
<img src="https://github.com/user-attachments/assets/8007cc10-2079-4beb-8638-fda8147bc599">
</div>

   - category로 검색 : 매우 효율적
     + 인덱스의 앞부분만 보고 빠르게 찾을 수 있음 (예: '전자기기' 섹션으로 바로 점프)
   - category와 price로 검색 : 역시 매우 효율적
     + '전자기기' 섹션을 찾은 뒤, 그 안에서 price 순으로 정렬된 데이터를 탐색하면 됨
     + '전자기기' 안에서는 price 가 항상 정렬된 상태를 유지
     + 각각의 카테고리 안에서는 price 가 항상 정렬된 상태를 유지
     + 1차 정렬의 기준 안에서 2차 정렬은 항상 정렬된 상태를 유지

   - price만으로 검색 : 매우 비효율적
     + 전화번호부에서 성은 모르고 이름 만으로 찾는 것과 같음
     + price 값은 각 category 섹션마다 흩어져 있기 때문에, 인덱스 전체를 다 훑어봐야 함
     + 이런 경우 옵티마이저는 차라리 풀 테이블 스캔을 선택할 수도 있음

   - price만으로 검색하면 왜 인덱스를 사용하기 어려운가?
     + idx_items_category_price 인덱스 예시에서 price 컬럼만 딱 분류해서 보면, 가격순으로 정렬이 되어 있는 것 같아 보이지만 중간에 정렬이 다시 틀어짐
     + 결과적으로 price 컬럼만 보면 정렬이 되어 있지 않음
     + 결과적으로 1차 정렬의 기준 없이 2차 정렬의 값만 가지고는 정렬된 상태를 유지할 수 없음

<div align="center">
<img src="https://github.com/user-attachments/assets/05d7941e-1e12-4c46-8800-6d583048abc4">
</div>

4. 💡 이처럼 복합 인덱스는 첫 번째 컬럼을 기준으로 정렬된 상태에서만 제 역할을 할 수 있음
   - 이를 인덱스 왼쪽 접두어 규칙(Index Left-Prefix Rule)이라고 함
   - 인덱스를 (A, B, C) 순서로 생성했다면, WHERE 조건에 다음과 같이 사용될 때 효율적
      + (A)
      + (A, B)
      + (A, B, C)
   - 하지만 (B), (C), (B, C) 와 같이 첫 번째 기준인 A가 빠진 조건으로는 인덱스를 제대로 활용할 수 없음

5. 💡 복합 인덱스 대원칙
   - 인덱스는 순서대로 사용 (왼쪽 접두어 규칙)
   - 등호(=) 조건은 앞으로, 범위 조건(<, >)은 뒤로 설정
   - 정렬(ORDER BY)도 인덱스 순서를 따를 것

6. 복합 인덱스 준비 과정
   - 기존에 존재하는 idx 로 시작하는 모든 인덱스를 제거
```sql
SHOW INDEX
FROM items;
```
   - 실행 결과 idx로 시작하는 모든 인덱스를 제거 (인덱스는 아쉽게도 IF EXISTS 구문이 없으므로, 직접 제거)
```sql
DROP INDEX idx_items_item_name ON items;
DROP INDEX idx_items_price_name ON items;
DROP INDEX idx_items_price ON items;
DROP INDEX idx_items_price_category_temp ON items;
DROP INDEX idx_items_category_price ON items;
```
   - 실습을 위해 (category, price) 순서로 복합 인덱스를 생성
```sql
CREATE INDEX idx_items_category_price
ON items (category, price);
```
   - 만들어진 인덱스를 최종 확인
```sql
SHOW INDEX FROM items;
```
   - 결과는 반드시 다음과 같이 idx로 시작하는 인덱스가 idx_items_category_price 하나만 추가되어야 함
<div align="center">
<img src="https://github.com/user-attachments/assets/a8543819-a3d7-4285-86ae-ea99ce09dd81">
</div>

   - idx_items_category_price는 하나의 복합 인덱스
   - Seq_in_index는 복합 인덱스의 컬럼 순서를 나타냄 : idx_items_category_price의 경우 1번은 category, 2번은 price 순서로 만들어진 것을 확인할 수 있음

7. 주의
   - 복합 인덱스를 실습하기 위해 반드시 기존에 존재하는 idx로 시작하는 모든 인덱스를 제거
   - idx로 시작하는 인덱스는 idx_items_category_price 인덱스만 존재해야 함

8. 복합 인덱스 성공 예제 1 : category 사용
   - 복합 인덱스의 첫 번째 컬럼만 WHERE 절에 사용하는 경우
   - 이는 인덱스 왼쪽 접두어 규칙을 가장 잘 활용하는 기본적인 예시
   - 상황 : 카테고리가 '전자기기'인 모든 상품을 찾아보기
   - 이제 이 인덱스가 있는 상태에서 쿼리의 실행 계획을 확인
```sql
EXPLAIN SELECT *
FROM items
WHERE category = '전자기기';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/35d55ecf-1a13-4ed7-b56a-d7464f004878">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/84857e9d-faad-462b-a016-6f141519049b">
</div>

   - (type : ref) : category 가 '전자기기'인 조건을 만족하는 데이터를 찾기 위해 동등 비교(=)나 JOIN 을 사용하고, 인덱스를 효율적으로 사용했음을 의미
     + 쉽게 이야기해서 Key 값 하나를 딱 집어서 찾은 것
   - (key : idx_items_category_price) : 우리가 만든 복합 인덱스가 사용
   - (rows : 10) : 옵티마이저는 '전자기기' 카테고리에 해당하는 상품이 약 10건 있을 것으로 정확히 예측하고, 그 부분만 탐색

   - 이는 (category, price) 로 정렬된 인덱스에서 category 가 '전자기기'인 첫 번째 위치를 빠르게 찾아낸 뒤, '전자기기'가 끝날 때까지 인덱스를 순차적으로 읽기만 하면 되므로 매우 효율적
   - 전화번호부에서 '김'씨를 찾는 것과 같이, 'ㄱ' 섹션으로 바로 이동해서 '김'씨가 끝날 때까지 읽는 것과 같음

9. 복합 인덱스 성공 예제2 : category, price 함께 사용
   - 이번에는 복합 인덱스를 구성하는 모든 컬럼을 WHERE 절에 사용하는 경우
   - 인덱스의 필터링 능력을 최대로 활용하는 상황
   - 상황 : 카테고리가 '전자기기'이면서, 가격이 정확히 120,000원인 상품을 찾아보기
```sql
EXPLAIN SELECT *
FROM items
WHERE category = '전자기기' AND price = 120000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/bcbd5e29-fdf3-4d10-b7c2-3e93008dc610">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/5268dfb1-e1c5-40e1-94ea-a2765c07b2b4">
</div>

   - (type : ref) : 두 개의 컬럼 조건(category, price)을 모두 만족하는 데이터를 찾기 위해 인덱스를 사용
   - (rows : 1) : 탐색할 행의 수가 단 1개로 예측
      + 데이터베이스는 먼저 category가 '전자기기'인 섹션을 찾고, 그 안에서 price가 120000 인 지점을 탐색
      + '전자기기' 섹션 내부는 이미 price 순으로 정렬되어 있으므로, 원하는 데이터를 매우 빠르게 특정할 수 있음

   - 이는 전화번호부에서 '김수로'라는 사람을 찾는 것과 같음 : '김'씨 섹션을 찾고, 그 안에서 '수로'라는 이름을 찾아내는 과정은 매우 빠름
   - 이처럼 인덱스를 구성하는 모든 컬럼을 조건으로 사용하면 가장 효과적으로 데이터를 필터링할 수 있음

10. 복합 인덱스 성공 예제3 : 복합 인덱스와 정렬
   - 복합 인덱스의 진정한 힘은 정렬(ORDER BY) 작업을 피할 때 드러남
   - WHERE 절의 필터링과 ORDER BY 의 정렬 방향이 인덱스 순서와 일치하면, 데이터베이스는 불필요한 filesort 작업을 생략해서 성능을 크게 향상시킬 수 있음
   - 상황 : 카테고리가 '전자기기'이면서 100,000원 초과인 상품을 가격 오름차순으로 정렬해서 출력하라는 요구사항
```sql
EXPLAIN SELECT *
FROM items
WHERE category = '전자기기' AND price > 100000
ORDER BY price;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/12eac282-719c-4291-a0f1-7d916aab6419">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/59923628-dcaa-4b87-a694-0bc6a5a4302b">
</div>

   - 가장 주목해야 할 부분은 Extra 컬럼에 Using filesort 가 없다는 점
   - 데이터베이스의 동작 과정
      + idx_items_category_price 인덱스를 사용해 category 가 '전자기기'인 섹션으로 빠르게 이동
      + 해당 섹션 내에서, price가 100000을 초과하는 첫 번째 데이터를 찾음
      + 그 지점부터 '전자기기' 섹션이 끝날 때까지 인덱스를 순서대로 읽기만 하면 됨

   - 왜냐하면 인덱스의 '전자기기' 섹션은 이미 price 순서로 완벽하게 정렬되어 있기 때문임
   - 따라서 데이터베이스는 별도로 데이터를 모아 다시 정렬할 필요 없이, 인덱스를 읽는 즉시 ORDER BY price 조건을 만족하는 결과를 얻게 됨
   - 💡 이처럼 WHERE 절의 조건과 ORDER BY 절의 조건이 복합 인덱스의 순서(category → price)와 일치하면, 데이터베이스는 가장 효율적인 방식으로 데이터를 찾고 정렬까지 한 번에 처리
   - filesort 를 피하는 것이야말로 복합 인덱스를 사용하는 핵심적인 이유 중 하나

11. 쉽게 이야기하면 ORDER BY를 사용할 때 복합 인덱스의 순서대로 정렬하면 추가적인 정렬(filesort)을 피할 수 있다는 것
    - 다음 예에서, 여기서 복합 인덱스가 category, price 순서이므로 ORDER BY도 다음과 같이 이 순서에 맞추어 사용하면 정렬을 최적화 할 수 있음
```sql
EXPLAIN SELECT *
FROM items
WHERE category = '전자기기' AND price > 100000
ORDER BY category, price; -- category, price 순서로 정렬
```
   - 그런데 여기서 선택된 category는 '전자기기' 단 하나이므로 이런 경우에는 category를 정렬 조건에 넣을 필요가 없음
   - 정렬은 최소 2개 이상 있을 때 의미가 있음 : 이런 경우에는 ORDER BY 에 category를 두는 것이 의미가 없으므로 다음과 같이 생략
```sql
EXPLAIN SELECT *
FROM items
WHERE category = '전자기기' AND price > 100000
ORDER BY price;
```
   - 당연한 이야기지만 만약 ORDER BY item_name 처럼 인덱스 순서와 다른 컬럼으로 정렬을 요청했다면, 조회한 결과를 다시 정렬해야 하기 때문에 Extra 컬럼에 Using filesort 가 발생할 것
```sql
EXPLAIN SELECT *
FROM items
WHERE category = '전자기기' AND price > 100000
ORDER BY item_name;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/de530b6e-88b1-447c-92a1-0bbcf418fb1f">
</div>
