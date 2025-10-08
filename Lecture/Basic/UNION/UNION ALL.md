-----
### UNION ALL
-----
1. 상황 : 마케팅팀에서 두 종류의 고객에게 이벤트 안내 메일을 보내려고 함
   - 첫 번째 그룹은 '전자기기' 카테고리의 상품을 구매한 이력이 있는 고객
   - 두 번째 그룹은 '서울'에 거주하는 고객
   - 두 그룹의 명단을 합쳐서 전체 발송 목록을 만들고 싶음

2. UNION 과 UNION ALL 의 차이 : 유일한 차이점은 '중복 처리' 여부
   - UNION : 두 결과 집합을 합친 후, 중복된 행을 제거
   - UNION ALL : 중복 제거 과정 없이, 두 결과 집합을 그대로 모두 합침

3. UNION 사용 시 (중복 제거) : 먼저, 두 고객 그룹을 UNION 으로 합쳐보기
```sql
-- 전자기기 구매 고객
SELECT u.name, u.email
FROM users u
INNER JOIN orders o ON u.user_id = o.user_id
INNER JOIN products p ON o.product_id = p.product_id
WHERE p.category = '전자기기'

UNION

-- 서울 거주 고객
SELECT name, email
FROM users
WHERE address LIKE '서울%';
```
   - 전자기기 구매 고객 : 션, 마리 퀴리, 네이트, 네이트(중복), 이순신
   - 서울 거주 고객 : 션, 세종대왕, 마리 퀴리
   - 중복 고객 : 션, 네이트(중복), 마리 퀴리

   - UNION은 이 중복을 제거하고 고유한 목록만 보여줌
<div align="center">
<img src="https://github.com/user-attachments/assets/c956b268-7b3a-4d0b-9dfe-a33b94cfe0e4">
</div>

   - 총 5명의 고유한 고객 목록이 생성 : 중복된 '션'과 '네이트', '마리 퀴리'는 한 번씩만 포함되었음

4. UNION ALL 사용 시 (중복 허용)
```sql
-- 전자기기 구매 고객
SELECT u.name, u.email
FROM users u
INNER JOIN orders o ON u.user_id = o.user_id
INNER JOIN products p ON o.product_id = p.product_id
WHERE p.category = '전자기기'

UNION ALL

-- 서울 거주 고객
SELECT name, email
FROM users
WHERE address LIKE '서울%';
```
   - UNION ALL은 각 SELECT 문의 결과를 그대로 모두 이어 붙임
   - '전자기기' 구매 내역은 총 5건(네이트 2건 포함)이고, '서울' 거주 고객은 3명이므로 총 8개의 행이 반환
<div align="center">
<img src="https://github.com/user-attachments/assets/58ae746d-b06e-424f-9cff-cda625c8738f">
</div>

   - 결과를 보면 '션', '네이트', '마리 퀴리'가 중복되어 나타나는 것을 확인할 수 있음

5. 실무 가이드 : 성능이 핵심
   - UNION ALL이 UNION 보다 훨씬 빠름
   - UNION : 두 결과를 합친 뒤, 중복을 제거하기 위해 데이터베이스는 보이지 않는 곳에서 추가 작업을 해야 함
     + 보통 전체 결과를 정렬(Sort)한 다음, 서로 인접한 행들을 비교하여 중복을 찾아내는 과정을 거침
     + 데이터의 양이 수십만, 수백만 건이라면 이 정렬과 비교 작업은 엄청난 비용과 시간을 소모
   - UNION ALL : 이런 추가 작업이 전혀 없이 그냥 첫 번째 SELECT 결과 아래에 두 번째 SELECT 결과를 가져다 붙이기만 하면 됨
   - 따라서 실무에서는 다음의 가이드라인을 따르는 것이 좋음
     + 중복을 제거해야만 하는 명확한 요구사항이 있을 때만 UNION을 사용 (예) 고유한 이메일 주소 목록, 고유한 고객 ID 목록 등)
     + 그 외의 모든 경우에는 UNION ALL 을 우선적으로 사용
        * 두 결과 집합에 중복이 발생할 수 없다는 것을 명확히 아는 경우
        * 중복이 발생해도 비즈니스 로직상 상관없는 경우
   - 중복을 제거할 필요가 없다면, 항상 UNION ALL 을 사용하는 것이 중요 : 불필요한 UNION 사용은 쿼리를 느리게 만드는 주범이 될 수 있음
