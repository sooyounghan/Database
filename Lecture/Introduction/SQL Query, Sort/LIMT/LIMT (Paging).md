-----
### LIMIT - 개수 제한
-----
1. 데이터가 수백만, 수천만 건이 된다고 가정
   - 이 데이터를 SELECT 문으로 한 번에 모두 조회하는 것은 시스템에 엄청난 부담을 제공
   - 조회 속도가 느려지는 것은 물론이고, 심하면 서버가 다운될 수도 있으며, 네트워크도 부하가 됨
   - 전체 데이터가 아니라 '상위 N개'만 보거나, '특정 구간'의 데이터만 잘라서 보고 싶은 경우가 훨씬 많음 : 이럴 때 사용하는 것이 바로 LIMIT 절

2. 조회 결과 개수를 제한하는 LIMIT
  - 💡 LIMIT 는 ORDER BY 뒤에 위치하며, 조회할 결과의 최대 개수를 제한하는 역할
```sql
SELECT 열이름
FROM 테이블이름
WHERE 조건
ORDER BY 정렬기준
LIMIT 개수;
```
   - 💡 LIMIT는 ORDER BY와 함께 사용해야 진정한 의미가 있음
     + 데이터베이스는 결과의 순서를 보장하지 않음
     + 정렬되지 않은 상태에서 LIMIT 2을 쓴다면, 실행할 때마다 다른 2개의 데이터가 나올 수도 있음
     + '가장 비싼 상품 TOP 2'처럼 의미 있는 상위 N개를 뽑으려면, 반드시 먼저 ORDER BY로 정렬한 뒤 LIMIT 로 잘라내야 함

   - products 테이블에서 가장 비싼 상품 2개만 조회하기
```sql
SELECT * FROM products ORDER BY price DESC LIMIT 2;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/0e417507-c81e-426f-ada9-69800b2f8fdf">
</div>

   - 가장 비싼 'LG 그램'부터 순서대로 2개의 상품만 정확히 조회

3. 💡 페이징(Pagination) 처리
   - 게시판, 상품 목록, 고객 리스트 등 웹사이트나 앱에서 보는 거의 모든 목록 페이지는 '페이징(Paging)' 처리가 되어있음
     + 수십만 개의 게시글을 한 페이지에 다 보여주는 사이트는 없음
     + 보통 10개나 20개씩 끊어서 보여주고, 하단에 [1], [2], [3], [다음], [마지막] 같은 페이지 번호를 제공
     + 사용자가 2페이지를 클릭하면 1페이지에 보였던 데이터는 건너뛰고, 그 다음 데이터들을 보여줘야 함
     + 이 페이징 기능은 바로 LIMIT 의 또 다른 사용법, '오프셋(Offset)'을 이용

4. 💡 특정 범위의 결과만 조회하기 : LIMIT, 오프셋(Offset)
   - LIMIT 는 숫자 하나만 쓸 수도 있지만, 두 개를 쓸 수도 있음
```sql
LIMIT 건너뛸개수(offset), 가져올개수(row_count);
```
   - offset : 앞에서부터 몇 개의 데이터를 건너뛸 것인지를 지정
   - row_count : 건너뛴 다음부터 몇 개의 데이터를 가져올 것인지를 지정
   - 예를 들어, LIMIT 10, 5 라고 하면, 맨 앞의 10개 데이터는 건너뛰고, 11번째 데이터부터 5개를 가져오라는 의미

5. LIMIT 다양한 문법
   - MySQL에서는 LIMIT 0, 2와 같이 offset 이 0인 경우 LIMIT 2와 같이 줄여 쓸 수 있음
   - MySQL에서는 LIMIT 0, 2를 LIMIT 2 OFFSET 0 과 같이 OFFSET 키워드를 넣어 사용할 수 있음
   - 이것을 페이징에 적용
     + 한 페이지에 2개의 상품을 보여준다고 가정
     + 1페이지 : 처음부터 2개를 가져와야 한다. 건너뛸 필요가 없으니 offset은 0, LIMIT 0, 2
     + 2페이지 : 앞의 2개(1페이지 분량)를 건너뛰고 2개를 가져와야 하므로 offset은 2, LIMIT 2, 2
     + 3페이지 : 앞의 4개(1, 2페이지 분량)를 건너뛰고 2개를 가져와야 하므로 offset은 4, LIMIT 4, 2

   - 💡 규칙 : offset 을 계산하는 간단한 공식 (💡 offset = (페이지번호 - 1) * 페이지당_게시물수)
   - products 목록을 한 페이지당 2개씩 보여줄 때, 1페이지 조회하기 (offset = (1 - 1) * 2 = 0, LIMIT 0, 2)
     + 사용자에게 일관된 순서를 보여주기 위해 product_id로 정렬
```sql
SELECT * FROM products ORDER BY product_id ASC LIMIT 0, 2;
```
  - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/14c5cdf5-3c0b-432b-b718-2e7b55f94b1b">
</div>

   - products 목록의 2페이지 (3~4번째 상품) 조회하기 (offset = (2 - 1) * 2 = 2, LIMIT 2, 2)
```sql
SELECT * FROM products ORDER BY product_id ASC LIMIT 2, 2;
```
  - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/7934b3cd-21fa-49b7-afd3-1f562b71edb7">
</div>

   - products 목록의 3페이지 (5번째 상품) 조회하기 (offset = (3 - 1) * 2 = 4, LIMIT 4, 2)
```sql
SELECT * FROM products ORDER BY product_id ASC LIMIT 4, 2;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/9b4bf9fb-b00e-40d2-b861-7a8c65ec9b3e">
</div>

   - 마지막 페이지에서는 가져올 데이터가 1개뿐이므로 LIMIT 에 지정한 2 개보다 적은 1개의 행만 반환

6. 💡 이처럼 LIMIT와 offset을 이용하면 아무리 많은 데이터라도 잘게 쪼개서 효율적으로 사용자에게 보여줄 수 있음
  - 페이징은 실무에서 반드시 알아야 하는 핵심 기능
