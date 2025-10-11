-----
### 복합 인덱스 실패 예제
-----
1. 복합 인덱스 대원칙
  - 인덱스는 순서대로 사용 (왼쪽 접두어 규칙)
  - 등호(=) 조건은 앞으로, 범위 조건(<, >)은 뒤로 설정
  - 정렬(ORDER BY)도 인덱스 순서를 따를 것

2. 복합 인덱스 실패 예제 1 : 인덱스 순서 무시
   - 복합 인덱스의 첫 번째 컬럼(category)을 건너뛰고, 두 번째 컬럼(price)만으로 데이터를 검색할 경우
   - 카테고리와 상관없이 가격이 80,000원인 상품을 찾아보기
```sql
EXPLAIN SELECT *
FROM items WHERE price = 80000;
```
   - 복합 인덱스를 실습하기 위해 반드시 기존에 존재하는 idx 로 시작하는 모든 인덱스를 제거
   - idx로 시작하는 인덱스는 idx_items_category_price 인덱스만 존재해야 함

<div align="center">
<img src="https://github.com/user-attachments/assets/8a93b6a7-ee5f-4c0f-97a2-bb16e2647409">
</div>

<img width="584" height="121" alt="image" src="" />

   - (type : ALL) : 풀 테이블 스캔이 발생
   - (key : NULL) : idx_items_category_price 인덱스가 있음에도 불구하고, 옵티마이저는 이 인덱스를 사용하지 않음
     + 인덱스 왼쪽 접두어 규칙 때문에 발생
     + idx_items_category_price 인덱스는 category로 먼저 정렬되어 있으므로, price가 80000인 상품은 '전자기기' 카테고리에도 있을 수 있고, 만약 있다면 '패션' 카테고리에도 있을 수 있음
     + 즉, price 값은 인덱스 전체에 흩어져 있기 때문임
<div align="center">
<img src="https://github.com/user-attachments/assets/466d1d31-f0c9-492a-9bfc-ca8ffe3d4b4c">
</div>

   - 데이터베이스 입장에서 price 만으로 데이터를 찾으려면, '도서' 카테고리 섹션을 처음부터 끝까지 다 보고, '생활용품' 섹션도 다 보고, '전자기기', '패션' 등 모든 카테고리 섹션을 다 뒤져봐야 함
   - 이는 인덱스 전체를 스캔하는 작업으로, 차라리 원본 테이블 전체를 스캔하는 것보다 나을 것이 없음
   - 전화번호부에서 성은 모르고 이름이 '수로'인 사람을 찾으려면, 가나다순으로 모든 페이지를 넘겨봐야 하는 것과 같은 이치로, 이처럼 복합 인덱스는 선행 컬럼 조건 없이는 제 역할을 하지 못함

3. 복합 인덱스 실패 예제 2 : 범위 조건을 먼저 사용
   - 💡 선행 컬럼에 범위 조건(>, <, BETWEEN, LIKE %)이 사용되면, 그 뒤에 오는 컬럼은 인덱스를 제대로 활용할 수 없음
   - 카테고리명이 '패션' 이상인 상품들 중에서, 가격이 정확히 20,000원인 상품을 출력하는 요구사항
   - 문자열 정렬 순서로 보면 카테고리명이 패션 이상인 것은 '패션', '헬스/뷰티'
```sql
EXPLAIN SELECT *
FROM items
WHERE category >= '패션' AND price = 20000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/3609fb71-6ade-4b29-8f72-8820edecfe37">
</div>

   - type이 range 이고, key에 idx_items_category_price가 사용되었으니 언뜻 보기에는 인덱스를 잘 사용한 것처럼 보임
   - 하지만 여기서 주목해야 할 것은 filtered 컬럼의 값인 10.00%와 Extra 컬럼의 Using index condition
<div align="center">
<img src="https://github.com/user-attachments/assets/1f3d09eb-a813-4284-a71f-7693c5c709cc">
</div>

   - idx_items_category_price 인덱스를 사용해 category 가 '패션'인 위치를 찾음 (>= 조건의 시작점)
   - 거기서부터 인덱스의 끝까지 모든 데이터를 스캔 ('패션', '헬스/뷰티' 카테고리에 해당하는 모든 데이터)
   - 스캔하는 각 레코드마다 price = 20000 조건을 만족하는지 하나하나 검사 (인덱스 사용이 아니라 직접 필터링)
   - 이것이 바로 (filtered : 10.00%)의 의미
     + 옵티마이저는 category >= '패션' 조건으로 약 6개의 행을 찾을 것으로 예상하고(rows: 6), 그 중에서 price = 20000 조건을 만족하는 데이터는 10% 정도일 것이라고 예측
     + 즉, price 컬럼은 데이터를 효율적으로 '찾는(seek)' 데 사용된 것이 아니라, 일단 category 조건으로 넓게 가져온 데이터를 '걸러내는(filter)' 데만 사용된 것

   - 정리
     + 데이터베이스는 category >= '패션' 조건에 따라 인덱스에서 '패션'과 '헬스/뷰티' 섹션을 찾음
     + 하지만 그 다음 조건인 price = 20000 을 처리할 때 문제가 생김
       * '패션' 섹션은 price 순으로 정렬되어 있음
       * '헬스/뷰티' 섹션도 price 순으로 정렬되어 있음
       * 하지만 '패션' 섹션의 가격들과 '헬스/뷰티' 섹션의 가격들이 전체적으로 정렬된 것은 아님
       * '패션'의 마지막 상품 가격이 '헬스/뷰티'의 첫 상품 가격보다 높을 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/79fcbedb-7205-4cdf-8291-c7d6c7ddacfe">
</div>

   - 처음 조건으로 찾은 패션, 헬스/뷰티 의 가격을 찾은 순서대로 합치면 다음과 같음
<div align="center">
<img src="https://github.com/user-attachments/assets/1f0c5a1d-d264-4d68-bf52-b8a9181fb4e8">
</div>

   - 합쳐진 가격은 이제 정렬된 상태가 아님 (인덱스를 활용하려면 정렬된 상태여야 함)
   - 따라서 데이터베이스는 category >= '패션' 범위를 스캔하면서 가져온 각각의 데이터에 대해 price = 20000인지 일일이 확인하는 추가 작업을 해야만 함
   - 이렇게 찾은 패션 + 헬스 / 뷰티의 price 컬럼은 정렬되어 있지 않음
   - 따라서 인덱스를 통한 빠른 탐색이 어려움
   - 데이터베이스는 category >= '패션' 범위를 인덱스의 category 컬럼를 통해 스캔하면서 6개의 행을 빠르게 찾았음
     + 하지만 가져온 각각의 데이터에 대해 price = 20000 인지 일일이 확인하는 필터링 추가 작업을 해야만 함

4. 이처럼 복합 인덱스에서 앞선 컬럼(category)에 범위 조건(>=)이 걸리는 순간, 데이터베이스는 더 이상 뒤따라오는 컬럼(price)의 정렬 순서를 활용할 수 없게 됨
   - category 가 '패션'일 때의 price 정렬과 category 가 '헬스/뷰티'일 때의 price 정렬은 둘을 합치는 순간 정렬이 깨지기 때문임
   - 따라서 데이터베이스는 category >= '패션' 이라는 범위 조건만 사용해서 인덱스를 스캔하고, 나머지 price = 20000 조건은 빠르게 찾지 못하고, 필터링만 수행하는 것
   - 이는 인덱스의 성능을 절반만 활용하는 셈
   - 이처럼 복합 인덱스에서 어떤 컬럼에 범위 검색을 사용하는 순간, 그 뒤에 오는 컬럼들은 인덱스의 정렬 효과를 제대로 누릴 수 없게 됨
   - 따라서 인덱스를 설계할 때는 = 조건으로 사용될 컬럼을 범위 조건으로 사용될 컬럼보다 앞에 배치하는 것이 일반적인 최적화 전략
