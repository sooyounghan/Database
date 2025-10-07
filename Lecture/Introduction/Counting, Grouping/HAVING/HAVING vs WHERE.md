-----
### HAVING - 그룹 필터링 2
-----
1. WHERE vs HAVING
    - 둘 다 '필터링'을 하지만, 작동하는 시점과 대상이 완전히 다르다는 것을 이제 명확히 구분해야 함
<div align="center">
<img src="https://github.com/user-attachments/assets/fd8bc0e2-1bca-4d38-b6da-d2bdc9502463">
</div>

2. 💡 WHERE는 그룹화 이전에 개개인을 걸러내는 조건이고, HAVING은 그룹화 이후에 그룹 자체를 걸러내는 조건
3. 종합 예제) WHERE 와 HAVING 함께 사용하기
    - "가격이 10만 원 이상인 고가 상품들 중에서만, 카테고리별로 묶었을 때, 그 고가 상품이 2건 이상 팔린 카테고리 구하기"
      + WHERE : 가격이 10만 원 이상인 고가 상품들
      + GROUP BY : 카테고리별로 묶었을 때
      + HAVING : 그 고가 상품이 2건 이상 팔린 카테고리
    - 이 질문에는 두 가지 필터링 조건이 있음
      + 개별 주문 건에 대한 필터링 : 가격 >= 100000
      + 그룹화된 결과에 대한 필터링: 주문 건수 >= 2
    - 이것이 바로 WHERE 와 HAVING 을 함께 사용해야 하는 시나리오
```sql
SELECT
     category,
     COUNT(*) AS premium_order_count
FROM
     order_stat
WHERE
     price >= 100000 -- 1. 먼저 개별 행을 가격으로 필터링
GROUP BY
     category -- 2. 필터링된 행들을 카테고리별로 그룹화
HAVING
     COUNT(*) >= 2; -- 3. 그룹화된 결과 중, 건수가 2개 이상인 그룹만 필터링
```
   - 쿼리 실행 과정 추적
   - 1단계 : WHERE price >= 100000
     + WHERE 절이 개별 행들을 필터링한 : price 가 100,000원 미만인 행들은 이 단계에서 모두 탈락
```sql
SELECT *
FROM order_stat
WHERE price >= 100000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/7e1f0136-b87c-46f0-9548-32bdf5bfc45c">
</div>

   - 전체 11개 중 총 6개의 행만 다음 단계로 넘어감
   - order_id : 2, 4, 6, 10, 11 주문이 이 단계에서 제외

   - 2단계: GROUP BY category
    + 2단계를 통과한 6개의 행을 category 기준으로 그룹으로 묶음
    + 전자기기, 가구, 문구 그룹으로 묶임
<div align="center">
<img src="https://github.com/user-attachments/assets/70c3d878-0bf9-4526-a3a4-fbc2534d3e39">
<img src="https://github.com/user-attachments/assets/1c88b975-efef-44c9-b144-bb07fc0cf8ad">
</div>

   - 3단계: HAVING COUNT(*) >= 2
      + GROUP BY로 만들어진 그룹 결과에 대해 HAVING 조건절을 적용하여 필터링
      + 각 그룹의 COUNT(```*```) 값이 2 이상인 그룹만 살아남음
      + '문구' 그룹은 COUNT(```*```) 가 1이므로 이 조건을 만족하지 못해 여기서 제외
<div align="center">
<img src="https://github.com/user-attachments/assets/f90ddb92-7274-42d1-b19a-899d0f1974d2">
</div>

   - 최종 단계 : SELECT ...
     + 최종적으로 3단계를 통과한 그룹들에 대해 SELECT 절에 명시된 컬럼(category , COUNT(*))을 선택하여 최종 결과를 화면에 보여줌
     + 최종 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/0085b418-7f4a-433b-88b0-3d49532ce926">
</div>

4. 이처럼 WHERE는 그룹화할 대상을 미리 선별하는 역할을, HAVING 은 그룹화된 결과물을 최종적으로 필터링하는 역할을 수행
