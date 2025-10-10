-----
### 인덱스와 LIKE 범위 검색
-----
1. 💡 LIKE 절에서 인덱스를 사용하려면, 와일드카드(%)가 검색어의 뒤쪽에 위치해야 함
   - WHERE item_name LIKE '게이밍%' : 인덱스 사용 가능 (O)
   - WHERE item_name LIKE '%게이밍' : 인덱스 사용 불가 (X)
   - WHERE item_name LIKE '%게이밍%' : 인덱스 사용 불가 (X)

2. %가 앞에 있으면 시작점이 불분명해져 정렬된 인덱스를 활용할 수 없기 때문임
3. LIKE 검색 성공 예제: 와일드카드(%)가 뒤에 오는 경우
   - "상품 이름이 '게이밍'으로 시작하는 모든 상품을 찾아보기"
   - 이 쿼리는 와일드카드( % )가 검색어 뒤에 붙어있으므로, item_name 으로 정렬된 인덱스를 효율적으로 활용할 수 있움
```sql
SELECT *
FROM items
WHERE item_name LIKE '게이밍%';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/9afc494e-f7e1-410b-8189-02aa6e869de2">
</div>

   - 실행 계획을 확인
```sql
EXPLAIN SELECT *
FROM items
WHERE item_name LIKE '게이밍%';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/149a5dc7-85a8-41cf-8f86-e4130f40a60a">
</div>

   - 여기서도 인덱스의 범위 검색(range)을 사용할 수 있음 : 왜냐하면 글자가 정렬되어 있기 때문임
   - 처음 한 번만 찾고, 이후에는 별도의 탐색 과정 없이 연속해서 결과를 구할 수 있음
   - 작동 순서를 확인
     + 먼저 인덱스의 item_name 항목에서 게이밍 으로 시작하는 조건을 찾음 : 이 조건은 이진 탐색의 원리를 사용하므로 매우 빨리 찾을 수 있음 (여기서는 게이밍 노트북을 찾음)
     + item_name이 순서대로 정렬되어 있기 때문에 인덱스의 바로 다음 행으로 넘어가서 게이밍으로 시작하는지 확인
       * 그리고 인덱스의 다음 행으로 넘어가면서 이 과정을 반복
       * 다음 행은 '게이밍 의자' : 게이밍으로 시작하는 조건에 부합하므로 결과의 대상
       * 다음 행은 'SQL 마스터 가이드' : 게이밍으로 시작하는 조건에 부합하지 않음
     + '게이밍'으로 시작하지 않는 항목을 발견했으므로 탐색을 종료

   - 이것이 가능한 이유는 국어사전에서 '게'로 시작하는 단어를 찾는 것과 원리가 같기 때문임 : 쉽게 이야기해서 글자순으로 정렬되어 있기 때문임
   - 데이터베이스는 idx_items_item_name 인덱스에서 '게이밍'으로 시작하는 첫 번째 위치를 빠르게 찾은 뒤, '게이밍'으로 시작하지 않는 단어가 나올 때까지 인덱스를 순차적으로 읽기만 하면 됨
<div align="center">
<img src="https://github.com/user-attachments/assets/3d262c7e-3a5e-4a34-bda1-3bc42ba45583">
</div>

   - (type : range) : item_name 이 '게이밍'으로 시작하는 특정 범위를 인덱스에서 스캔했음을 의미
     + 이는 풀 테이블 스캔(ALL)보다 훨씬 효율적이다.
   - (key : idx_items_item_name) : 생성한 idx_items_item_name 인덱스가 올바르게 사용
   - (rows : 2) : 옵티마이저는 약 2개의 행만 스캔하면 결과를 찾을 수 있을 것으로 예측

3. LIKE 검색 실패 예제 : 와일드카드(%)가 앞에 오는 경우
   - "상품 이름에 '게이밍'이 포함된 모든 상품을 찾아보기"
   - 이 쿼리는 실무에서 검색 기능을 만들 때 매우 흔하게 사용 : 하지만 와일드카드(%)가 검색어 앞에 붙어있기 때문에 인덱스의 장점을 활용할 수 없음
```sql
SELECT *
FROM items
WHERE item_name LIKE '%게이밍%';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/ea3ce78f-c9d1-4ac0-bc9b-522513a663d0">
</div>

   - 실행 계획을 분석
```sql
EXPLAIN SELECT *
FROM items
WHERE item_name LIKE '%게이밍%';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/51181b92-600c-43d8-8eab-c0fa7752732e">
</div>

  - (type : ALL) : 풀 테이블 스캔이 발생
  - (key : NULL) : idx_items_item_name 인덱스가 있음에도 불구하고 사용되지 않았음
     + 국어사전에서 중간에 '게이밍'이라는 글자가 들어간 단어를 찾으려면 어떻게 해야 할까?
     + '가'부터 '힣'까지 모든 단어를 하나씩 다 훑어보는 수밖에 없는데, 데이터베이스도 마찬가지
     +  item_name의 시작 글자를 알 수 없으므로, 정렬된 인덱스는 아무런 도움이 되지 않음
     + 결국 데이터베이스는 테이블의 모든 데이터를 처음부터 끝까지 스캔하는 최악의 방법을 선택하게 됨

4. 💡 중요 : LIKE 검색은 %를 마지막에 사용할 때만 인덱스를 활용할 수 있음
   - LIKE도 범위 검색을 사용할 수 있음
   - 단, 인덱스를 사용하려면 반드시 와일드카드(%)를 뒤에 사용해야 함

5. 💡 실무 팁 : 전문 검색, 전체 문자 검색 (Full-Text Search)
   - 이처럼 LIKE '%검색어%' 방식은 데이터가 많아질수록 성능이 심각하게 저하되어 실제 서비스에서는 사용하기 어려움
   - 이런 '내용 검색' 또는 '포함 검색' 문제를 해결하기 위해 데이터베이스는 전문 검색(Full-Text Search)이 라는 특수한 기능을 제공
   - 전문 검색 인덱스는 B-Tree 인덱스와는 달리, 텍스트를 단어(토큰) 단위로 쪼개서 인덱싱하는 방식
   - 이를 통해 텍스트 중간에 있는 단어도 매우 빠르게 검색할 수 있음
    - 만약 쇼핑몰에서 상품명 검색 기능을 구현해야 한다면, LIKE 대신 MATCH ... AGAINST 구문을 사용하는 전문 검색 기능을 도입하는 것이 해결 방법 (필요하다면 'FullText Search'라는 키워드로 검색)

