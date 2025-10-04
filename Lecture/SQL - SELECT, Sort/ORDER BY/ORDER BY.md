-----
### ORDER BY - 정렬
-----
1. 데이터베이스는 데이터를 가장 효율적으로 저장하고 관리할 뿐, 우리가 조회할 때 특정 순서대로 보여줘야 할 의무가 없음
2. 어떨 때는 기본 키(PRIMARY KEY) 순서로, 어떨 때는 데이터가 삽입된 순서로 보이는 등 일관성이 없음
3. 따라서 데이터를 분석하거나 보고서를 만들 때는 의미 있는 순서로 나열하는 과정이 반드시 필요
   - '최신순', '가격 높은순', '이름 가나다순' 와 같이 사용하는 것이 ORDER BY 절
4. 결과의 순서를 지정하는 ORDER BY
```sql
SELECT 열이름
FROM 테이블이름
WHERE 조건
ORDER BY 정렬기준열 [정렬방식];
```
  - [정렬방식]은 생략 가능

5. 💡 ORDER BY 절은 SELECT 문의 가장 마지막에 위치하며, 조회된 결과를 특정 열의 값을 기준으로 정렬하는 역할
6. 정렬 방식에는 다음 두 가지
   - ASC (Ascending) : 오름차순 정렬. 작은 값에서 큰 값으로 (1, 2, 3... / A, B, C... / 오래된 날짜부터 최신 날짜) - ORDER BY 의 기본값이므로 생략할 수 있음
   - DESC (Descending) : 내림차순 정렬. 큰 값에서 작은 값으로 (10, 9, 8... / Z, Y, X... / 최신 날짜부터 오래된 날짜) - 내림차순을 원할 경우 반드시 명시해야 함

7. 첫 번째 요청, "최근에 가입한 고객 순서대로" 명단 생성
   - customers 테이블을 가입일(join_date) 최신순으로 정렬하기
```sql
SELECT * FROM customers ORDER BY join_date DESC;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/2406f24a-8e5d-42e5-b566-9524bb7efbeb">
</div>

  - 가입일(join_date)이 최신인 '장영실'부터 가장 오래된 '이순신' 순서로 정렬된 것을 볼 수 있음
  - 이번에는 반대로 오래된 가입 고객 순서대로 오름차순(ASC)으로 출력
```sql
SELECT * FROM customers ORDER BY join_date ASC;
SELECT * FROM customers ORDER BY join_date; -- ASC는 생략 가능
```
  - 정렬의 기본 값은 오름차순(ASC)으로, 따라서 오름차순일 경우 ASC를 생략할 수 있음
  - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/c0201dc4-fcb5-4cd9-8d30-8c05d10f6463">
</div>

   - 두 번째 요청, "가격이 낮은 상품부터" 보여달라고 한다면 가격(price)을 기준으로 오름차순(ASC) 정렬  
   - ASC는 생략 가능하므로 ORDER BY price 라고만 써도 됨
   - products 테이블을 가격(price) 낮은순으로 정렬하기
```sql
SELECT * FROM products ORDER BY price ASC; -- 'ASC'는 생략 가능
```
  - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/7f703367-8e73-430d-a773-d6900286386b">
</div>

   - 가격이 3000원인 '에어팟'부터 20000원인 'LG 그램' 순서로 완벽하게 정렬

8. 실무 팁
   - 오름차순인 경우 ASC 키워드는 생략할 수 있음 (정렬의 기본 값이기 때문임)
   - 실무에서는 오름차순인 경우에는 ASC 키워드를 대부분 생략하고 ORDER BY만 사용
   - 물론 내림차순의 경우 DESC를 필수로 적어야 하기 때문에 생략하면 안 됨

9. 다중 열 정렬 (Multi-column Sort)
    - ORDER BY 절에 여러 개의 열 이름을 콤마(,)로 구분하여 나열
    - 정렬의 우선순위는 당연히 먼저 쓴 열부터 적용
```sql
ORDER BY 1차정렬열 [정렬방식], 2차정렬열 [정렬방식], ...
```
   - 각 열마다 ASC나 DESC 를 개별적으로 지정 가능
   - products 테이블을 재고수량 내림차순, 가격 오름차순으로 정렬하기
```sql
SELECT * FROM products ORDER BY stock_quantity DESC, price ASC; -- price의 ASC는 생략 가능
```
   - 💡 해당 SQL 쿼리는 상품 데이터를 재고 수량(stock_quantity)이 많은 순서로 먼저 정렬하고, 만약 재고 수량이 같다면 가격(price)이 낮은 순서로 다시 정렬
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/5cefdcb7-08a6-4705-866e-224b3355a220">
</div>

   - 결과를 자세히 보자. '갤럭시'와 '아이폰'은 재고 수량이 55개로 동일
   - 이 경우 2차 정렬 기준인 price ASC 가 적용되어, 가격이 더 낮은 '아이폰'(5000원)이 '갤럭시'(10000원)보다 먼저 표시
