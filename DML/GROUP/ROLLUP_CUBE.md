-----
### ROLLUP (expr1, expr2)
-----
1. expr로 명시한 표현식 기준으로 집계한 결과, 즉 추가적인 집계 정보를 보여줌
2. ROLLUP 절에는 그룹핑 대상, 즉 SELECT 리스트에서 집계 함수를 제외한 컬럼 등의 표현식
3. 명시한 표현식 수의 순서(오른쪽에서 왼쪽 순)에 따라 레벨 별로 집계한 결과 반환
4. 표현식의 개수가 n개이면, n + 1레벨까지, 하위 레벨에서 상위 레벨 순으로 데이터 집계

<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/2508ce8e-399c-40e4-b283-2a7a8be886b7">
</div>

  - ROULLUP 절에 period와 gubun을 명시하였으므로, 총 레벨 수는 3
  - 레벨 3 : 월과 대출종류(period, gubun)
  - 레벨 2 : 월(period)
  - 레벨 1 : 전체 합계가 집계

-----
### 분할 ROLLUP
-----
1. GROUP BY expr1, ROLLUP(expr2, expr3)
2. 레벨은 2 + 1레벨로 총 3레벨
3. 결과 : (expr1, expr2, expr3), (expr1, expr2), (exp1) 순으로 집계
4. 전체 합계는 집계되지 않음
   
<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/69d4f6ae-3d7e-4011-8770-d30eaf2e717d">
</div>

  - period, GROUP BY(gubun) : 전체 레벨은 1 + 1 = 2레벨
  - 레벨 2 : 월과 대출 종류 (period, gubun)
  - 레벨 1 : 월 (period)

<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/5bcfebd0-0c09-4e48-a9f1-b7d87481679f">
</div>

  - period, GROUP BY(gubun) : 전체 레벨은 1 + 1 = 2레벨
  - 레벨 2 : 월과 대출 종류 (period, gubun)
  - 레벨 1 : 대출 종류 (gubun)

-----
### CUBE (expr1, expr2)
-----
1. 명시한 표현식 개수에 따라 가능한 모든 조합별로 집계
2. 레벨은 2의 (expr 수) 제곱 만큼 종류별 집계

<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/a92a5811-b980-407c-9c5b-0e4f1c8aa96d">
</div>

  - expr의 수가 2개 이므로, 총 4개의 유형이 집계
  - 레벨 1 : 전체 집계
  - 레벨 2 : 대출 종류별 집계 (gubun)
  - 레벨 3 : 월별 집계 (period)
  - 레벨 4 : 월별 대출 종류별 집계 (gubun + period)

-----
### 분할 CUBE
-----
1. GROUP BY expr1, CUBE(expr2, expr3)
   - (expr1, expr2, expr3), (expr1, expr2), (expr1, expr3), (expr1)로 집계
<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/e64d8d2c-84d1-4a05-bd6d-2a3fceb3fd90">
</div>

  - 2가지 유형 집계
  - 레벨 1 : (period, gubun) 
  - 레벨 2 : (gubun)

-----
### 정리
-----
<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/43849a18-a142-43e5-8be9-37de7872f2b1">
</div>
