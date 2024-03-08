-----
### 집합 연산자
-----
1. 데이터 집합을 대상으로 연산을 수행하는 연산자
2. 데이터의 집합의 수는 한 개 이상 사용 가능
3. 여러 개의 SELECT문을 연결해 또 다른 하나의 쿼리를 만들 수 있음

-----
### UNION : 합집합
-----
1. 두 개의 데이터 집합이 있으면, 각 집합 원소(SELECT 결과)를 모두 포함한 결과 반환 (중복된 데이터는 한 번만 조회)
```sql
SELECT GOODS
FROM EXP_GOODS_ASIA
WHERE COUNTRY = '한국'

UNION

SELECT GOODS
FROM EXP_GOODS_ASIA
WHERE COUNTRY = '중국';
```
: 한국과 중국 모두 두 국가 모두 수출품을 조회(단, 중복은 1번만 출력)


2. UNION ALL : UNION과 비슷하지만, 중복된 항목도 모두 조회
```sql
SELECT GOODS
FROM EXP_GOODS_ASIA
WHERE COUNTRY = '한국'

UNION ALL

SELECT GOODS
FROM EXP_GOODS_ASIA
WHERE COUNTRY = '중국';
```

-----
### INTERSECT : 교집합
-----
: 데이터 집합에서 공통된 항목만 추출
 ```sql
SELECT GOODS
FROM EXP_GOODS_ASIA
WHERE COUNTRY = '한국'

INTERSECT

SELECT GOODS
FROM EXP_GOODS_ASIA
WHERE COUNTRY = '중국';
```
: 두 국가 중에 겹치는 수출품만 조회

-----
### MINUS : 차집합
-----
: 한 데이터 집합을 기준으로 다른 데이터 집합과 공통된 항목을 제외한 결과만 추출

 ```sql
SELECT GOODS
FROM EXP_GOODS_ASIA
WHERE COUNTRY = '한국'

MINUS

SELECT GOODS
FROM EXP_GOODS_ASIA
WHERE COUNTRY = '중국';
```
: 한국 기준, 중국과 겹치는 수출품을 제외하고 한국만 수출하는 수출품 조회

-----
### 집합 연산자의 제한사항
-----
1. 집합 연산자로 연결되는 각 SELECT문의 SELECT 리스트의 개수와 데이터 타입은 일치
2. 집합 연산자로 SELECT 문을 연결할 때, ORDER BY 절은 맨 마지막 문장에서만 사용 가능
3. BLOB, CLOB, BFILE 타입의 컬럼에 대해서는 집합 연산자 사용 불가
4. UNION, INTERSECT, MINUS 연산자는 LONG형 컬럼에 사용할 수 없음
