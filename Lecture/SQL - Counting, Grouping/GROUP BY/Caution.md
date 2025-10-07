-----
### GROUP BY - 주의사항
-----
1. 💡 GROUP BY를 사용할 때 SELECT 절에는 GROUP BY에 사용된 컬럼과 집계 함수만 사용할 수 있음
   - 그룹을 대표하는 값(GROUP BY 컬럼)이나, 그룹 전체를 요약한 값(집계 함수)만 조회할 수 있도록 규칙을 정한 것

2. "카테고리별로 묶어서 상품 개수를 보고 싶다"는 요구 사항을 받았다고 가정
   - 다음과 같이 아주 간단하게 문제를 해결할 수 있음
```sql
SELECT
     category,
     COUNT(*)
FROM
     order_stat
GROUP BY
     category;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/694b583a-4a4a-4da6-a764-ccdbf8d5efa0">
</div>

   - 그런데 이 요구 사항에 추가로 "각 카테고리에 속한 상품명도 하나 보고 싶다"는 요구를 받았다고 가정
   - 아주 자연스러운 요구처럼 들림 : 그래서 다음과 같이 product_name 을 추가로 조회하는 쿼리를 작성했다고 가정
```sql
-- 잘못된 쿼리의 예시
SELECT
     category,
     product_name, -- 바로 이 컬럼이 문제
     COUNT(*)
FROM
     order_stat
GROUP BY
     category;
```
   - 실행 결과
```
Error Code: 1055. Expression #2 of SELECT list is not in GROUP BY clause and
contains nonaggregated column 'my_shop.order_stat.product_name' which is not
functionally dependent on columns in GROUP BY clause; this is incompatible
with sql_mode=only_full_group_by
```
   - product_name이 GROUP BY 절에 없다는 오류
   - 이 쿼리는 대부분의 데이터베이스에서 오류를 발생
   - '전자기기' 카테고리 그룹의 내부
      + GROUP BY category 명령을 내리면, 전자기기, 도서, 가구, 문구, NULL 각각의 카테고리를 그룹으로 묶음
      + 여기서 '전자기기' 카테고리의 데이터만 자세히 살펴보기
<div align="center">
<img src="https://github.com/user-attachments/assets/5374a3ad-f821-4ead-86d8-668cd2f88f8d">
</div>

   - 데이터베이스는 '전자기기' 카테고리에 속한 주문 5건을 하나의 그룹으로 묶어야 함
   - 쉽게 이야기해서 5행의 데이터를 단 한 행으로 만들어야 함
   - SELECT에서 category, product_name, COUNT(```*```)를 선택했기 때문에 각각의 대표를 하나만 선택해야 함
      + category :그룹의 기준이니 당연히 '전자기기'이다. '전자기기'라는 하나의 명확한 값을 구할 수 있음
      + category 전자기기 그룹에 속한 모든 값은 같은 '전자기기'
      + COUNT(*) : 그룹 내 주문이 5건이니, 5라는 명확한 하나의 값을 구할 수 있음
      + 💡 그런데 product_name의 경우 : '프리미엄 기계식 키보드', '고성능 게이밍 마우스', '4K 모니터', '노이즈캔슬링 블루투스 이어폰', '보조배터리 20000mAh' 이 다섯 가지 값 중에 어떤 것을 선택해야 하는지 데이터베이스는 이 질문에 대한 답을 알 수 없음
      + 즉, 요청 자체가 모호함

3. 올바른 해결책
   - 데이터베이스는 이처럼 모호한 명령을 처리할 수 없도록 설계
   - 그래서 "GROUP BY로 묶었다면, SELECT 절에는 그룹의 '대표 선수'인 GROUP BY 컬럼이나, 그룹의 '요약 정보'인 집계 함수만 올 수 있다"는 엄격한 규칙을 적용
   - 따라서 '전자기기' 그룹에서는 그룹을 대표하는 category 인 '전자기기'와, 그룹 전체를 요약한 COUNT(*), SUM(price * quantity) 등과 같은 값만 함께 조회할 수 있음
   - 💡 그룹 전체를 대표할 수 없는 개별 행의 정보(product_name , order_id , customer_name 등)는 SELECT 절에 단독으로 올 수 없음
   - 💡 정리하면 GROUP BY를 사용하는 경우 GROUP BY에 사용된 컬럼이나 집계 함수만 사용할 수 있음
```sql
SELECT
     category, -- GROUP BY에 사용된 컬럼
     product_name, -- GROUP BY에 사용된 컬럼이 아님 X
     quantity, -- GROUP BY에 사용된 컬럼이 아님 X
     COUNT(*), -- 집계 함수
     SUM(quantity) -- 집계 함수
FROM
     order_stat
GROUP BY
     category;
```
   - product_name, quantity는 GROUP BY에 사용된 컬럼이 아니고, 집계 함수를 사용한 것도 아니므로 오류가 발생
