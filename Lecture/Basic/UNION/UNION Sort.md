-----
### UNION 정렬
-----
1. UNION 또는 UNION ALL을 사용하여 여러 SELECT 문의 결과를 합칠 때, 최종 결과 집합에 대해 정렬(Sorting)을 적용할 수 있음
   - 이 때 ORDER BY 절의 위치가 중요
   - ORDER BY 절은 전체 UNION 연산의 가장 마지막에 한 번만 사용해야 함
   - 만약 각 SELECT 문 안에 ORDER BY를 사용하면 에러가 발생하거나, 예상과 다른 결과가 나올 수 있음 : UNION 은 각 SELECT 문의 개별적인 정렬 순서가 아니라, 합쳐진 최종 결과 전체에 대한 순서를 결정해야 하기 때문임
   - 예를 들어, 활동 고객과 탈퇴 고객의 명단을 합친 후, 이름(name)을 기준으로 오름차순 정렬하고 싶다고 가정
```sql
SELECT name, email FROM users

UNION

SELECT name, email FROM retired_users

ORDER BY name; -- 최종 결과에 대한 정렬
```
<div align="center">
<img src="https://github.com/user-attachments/assets/0d8b3c75-d52f-4464-b0df-de10ec9af2e4">
</div>

   - ORDER BY 절을 UNION 연산자의 마지막에 배치함으로써, 합쳐진 모든 고객의 이름이 글자 순서대로 깔끔하게 정렬됨

2. UNION에 나오지 않는 필드를 사용한다면?
   - UNION 또는 UNION ALL 연산의 ORDER BY 절에서는 첫 번째 SELECT 문의 컬럼 이름이나 해당 컬럼의 별칭(Alias)만 사용할 수 있음
   - 이는 UNION 결과 집합의 컬럼 이름이 첫 번째 SELECT 문을 따르기 때문임
```sql
SELECT name, email, created_at FROM users -- created_at

UNION ALL

SELECT name, email, retired_date FROM retired_users -- retired_date
```
<div align="center">
<img src="https://github.com/user-attachments/assets/911cff0c-d7ba-4550-9e8c-cf0c24e5b577">
</div>

   - 참고 : 정렬 조건을 주지 않았기 때문에 정렬 결과는 달라질 수 있음
   - 첫 번째 SELECT 문은 created_at, 두 번째 SELECT 문은 retired_date를 사용
      + 실행 결과를 보면 컬럼 이름에 created_at이 들어가 있는 것을 확인할 수 있음
   - 만약 첫 번째 SELECT 문에 없는 컬럼을 ORDER BY 절에서 사용하려고 하면 에러가 발생
```sql
SELECT name, email, created_at FROM users

UNION ALL
SELECT name, email, retired_date FROM retired_users

ORDER BY retired_date; -- 첫 번째 SELECT 문에 없는 컬럼
```
   - 실행 결과
```
Error Code: 1054. Unknown column 'retired_date' in 'order clause
```

3. 별칭을 사용하기
   - 다음의 created_at, retired_date과 같이 이름이 다른 컬럼이 있다면 별칭을 사용하는 것을 권장
   - 나중에 쿼리를 다시 보거나 다른 개발자가 볼 때, created_at이라는 이름만으로는 retired_date도 포함하여 정렬된다는 사실을 즉시 파악하기 어려울 수 있기 때문임
   - event_date와 같은 중립적인 별칭은 해당 컬럼이 여러 종류의 날짜 정보를 담고 있음을 명확하게 보여줌
```sql
SELECT name, email, created_at AS event_date FROM users

UNION ALL

SELECT name, email, retired_date AS event_date FROM retired_users

ORDER BY event_date DESC; -- 별칭을 사용하여 정렬
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5d035eda-30f8-4aef-b101-07b32b433098">
</div>

   - 참고 : 데이터를 생성한 날짜가 다를 수 있기 때문에 정렬 결과를 달라질 수 있음
