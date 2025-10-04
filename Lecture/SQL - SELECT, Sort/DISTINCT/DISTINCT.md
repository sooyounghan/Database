-----
### DISTINCT - 중복 제거
-----
1. 데이터를 조회하다 보면, 우리가 원하지 않는 중복된 값들이 계속 나타나는 경우가 있는데, 유일한 값들만 추출 하는 방법
```sql
SELECT customer_id FROM orders;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/08f8b4b1-408f-40a3-baa9-c9ea4b9f7ac2">
</div>

  - 중복된 값을 제거하고 순수하게 '어떤 종류'의 데이터가 있는지 확인할 수 있어야 함

2. 해결책 - DISTINCT
   - DISTINCT는 SELECT 문에서 컬럼 이름 바로 앞에 붙여 사용하며, 조회된 결과에서 중복된 행을 모두 제거하고 유일한(unique) 값만 남겨줌
```sql
SELECT DISTINCT 컬럼명 FROM 테이블명;
```
   - 문제 상황을 DISTINCT 를 사용해서 다시 해결
   - 주문한 고객의 ID를 중복 없이 보고 싶으니, customer_id 컬럼에 DISTINCT 적용
```sql
SELECT DISTINCT customer_id FROM orders;
```

   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/d601a95b-942e-4373-a10a-a5e595ad0b27">
</div>

   - 이제 각 고객 ID가 한 번씩만 나타남 : 1, 2, 3 번 고객이 우리 쇼핑몰에서 주문을 했다는 사실을 명확하게 알 수 있음

3. 여러 컬럼에 DISTINCT 적용하기
   - DISTINCT 는 하나의 컬럼에만 사용할 수 있는 것이 아니라, 여러 개의 컬럼을 대상으로도 사용할 수 있음
   - 이 경우, 지정된 모든 컬럼의 조합이 유일한 것들만 보여줌
```sql
SELECT customer_id, product_id FROM orders;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/9cfce5b9-61f2-4a7f-9d9a-2589046373ea">
</div>

   - 여기서 보면 (2, 2) 라는 조합, 즉 세종대왕 고객이 LG 그램을 주문한 기록이 두 번 나타남
   - 만약 우리가 '고객-상품'의 유일한 구매 조합만 보고 싶다면 이 중복은 제거하는 것이 좋음
   - DISTINCT 사용
```sql
SELECT DISTINCT customer_id, product_id FROM orders;
```
   - 💡 DISTINCT는 customer_id 와 product_id 두 컬럼을 하나의 묶음으로 보고, 이 묶음이 중복되는지를 판단
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/f35f7cc7-0638-4a5c-98b3-d7ee77576c1f">
</div>

   - 결과를 보면, 중복이었던 (2, 2) 행이 하나로 합쳐져 총 4개의 유일한 '고객-상품' 조합만 남은 것을 확인할 수 있음
   - 이처럼 DISTINCT는 여러 컬럼에 걸쳐 중복을 제거할 때도 매우 유용

4. 실무 팁
   - 실무에서 DISTINCT는 데이터를 탐색하고 분석할 때 많이 사용
   - 데이터의 '종류'를 파악할 때 아주 유용
   - 다만 한 가지 기억해야 할 점이 있는데, DISTINCT는 결과를 보여주기 전에 내부적으로 중복을 제거하기 위한 과정을 거쳐야 함
   - 따라서 데이터가 아주 많은 경우 일반적인 SELECT 보다 성능이 저하될 수 있음
   - 대량의 데이터라면 성능을 확인할 필요가 있음
