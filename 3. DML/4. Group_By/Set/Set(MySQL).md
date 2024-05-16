-----
### SET (집합 연산자)
-----
1. 두 SELECT 문을 통해 얻어온 결과를 집합 연산을 통해 하나의 결과로 만드는 것
2. 합집합, 교집합, 차집합 등 집합 연산을 할 수 있음
3. 집합 연산을 하기 위해서는 두 SELECT 문을 통해 가져오는 COLUMN이 같아야 함

-----
### 합집합 - UNION, UNION ALL
-----
1. 두 SELECT 문의 결과를 모두 포함하는 최종 결과를 반환
2. UNION : 중복되는 데이터를 하나만 가져옴
3. UNION ALL : 중복되는 데이터 모두 가져옴
4. 예제
```sql
SELECT EMP_NO
FROM TITLES
WHERE TITLE = 'Senior Staff'

UNION

SELECT EMP_NO
FROM TITLES
WHERE TITLE = 'Staff';
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/41c806a1-e708-418e-8694-6db5a6c2b4e6">
</div>

```sql
SELECT EMP_NO
FROM TITLES
WHERE TITLE = 'Senior Staff'

UNION ALL

SELECT EMP_NO
FROM TITLES
WHERE TITLE = 'Staff';
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/d184396f-12d6-4110-b9d7-188726cf42d8">
</div>

-----
### 교집합
-----
1. 두 SELECT문 결과 중 중복되는 부분만 가져옴
2. 💡 교집함은 JOIN문을 이용 (MySQL에서는 교집합 연산자 미제공)
3. 예제
```sql
SELECT T1.EMP_NO
FROM TITLES T1, TITLES T2
WHERE T1.EMP_NO = T2.EMP_NO  AND T1.TITLE = 'Senior Staff' AND T2.TITLE = 'Staff';
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/e0398580-bfeb-4bbf-a2d6-934ced58a8f6">
</div>

-----
### 차집합
-----
1. 두 SELECT 문 중에서 중복되는 부분을 제거하고, 첫 번째 SELECT 문 결과만 가져옴
2. 💡 차집합은 Sub-Query를 이용 (MySQL에서는 차집합 연산자 미제공)
3. 예제
```sql
SELECT EMP_NO
FROM TITLES
WHERE TITLE = 'Staff' AND 
	  EMP_NO NOT IN (SELECT EMP_NO
					 FROM TITLES
					 WHERE TITLE = 'Senior Staff');
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/a7b52a71-b231-4571-95d0-745ff7a37add">
</div>

