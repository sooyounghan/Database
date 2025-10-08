-----
### 정리
-----
1. INNER JOIN
<div align="center">
<img src="https://github.com/user-attachments/assets/22caa470-3cfc-41c3-af65-ae2c13ecaa1b">
</div>

2. LEFT OUTER JOIN
<div align="center">
<img src="https://github.com/user-attachments/assets/65b12ae9-235a-4e0a-bcbf-ce93570a9ad8">
</div>

3. RIGHT OUTER JOIN
<div align="center">
<img src="https://github.com/user-attachments/assets/5cc00f24-21e1-4c2f-aafe-60183ddb3333">
</div>

4. FULL OUTER JOIN (MySQL 지원 X)
<div align="center">
<img src="https://github.com/user-attachments/assets/614e845b-92f3-479b-af16-173071426f6e">
</div>

5. 외부 조인 1
   - 내부 조인은 양쪽 테이블에 모두 데이터가 존재하는 교집합 영역만 조회
   - 한쪽 테이블에만 데이터가 존재하는 경우까지 포함해서 조회하려면 외부 조인을 사용
   - 외부 조인은 기준 테이블의 모든 데이터를 결과에 포함
   - LEFT JOIN은 왼쪽 테이블을 기준으로 하고 RIGHT JOIN은 오른쪽 테이블을 기준으로 함
   - 기준 테이블과 짝이 맞는 데이터가 없는 경우 해당 필드는 NULL 값으로 채워짐
   - NULL 값을 비교할 때는 반드시 IS NULL 또는 IS NOT NULL을 사용해야 함

6. 외부 조인 2
   - LEFT JOIN과 RIGHT JOIN은 테이블의 위치를 바꾸면 서로 동일한 결과를 만들 수 있음
   - FROM A LEFT JOIN B는 FROM B RIGHT JOIN A와 같은 결과를 반환
   - 실무에서는 분석 기준이 되는 테이블을 FROM 절에 먼저 쓰고 LEFT JOIN으로 다른 테이블을 붙여나가는 방식이 더 직관적이므로 LEFT JOIN이 주로 사용

7. 조인의 특징
   - 조인 시 행의 개수가 변하는지 여부는 테이블 간의 관계(PK-FK)와 조인 방향에 따라 결정
   - 자식 테이블에서 부모 테이블로 조인(FK → PK)하면, 자식 테이블의 각 행은 부모의 단 하나의 행과 연결되므로 행 개수가 유지
   - 부모 테이블에서 자식 테이블로 조인(PK → FK)하면, 부모 테이블의 한 행이 자식의 여러 행과 연결될 수 있으므로 행 개수가 증가할 수 있음
   - 조인으로 인한 행 수 변화를 이해해야 SUM, COUNT 같은 집계 함수를 정확하게 사용할 수 있음

8. 셀프 조인
   - 셀프 조인은 하나의 테이블을 자기 자신과 조인하는 기법
   - 한 테이블 내에 계층적인 관계나 자기 참조 관계가 있을 때 사용(예) 직원의 상사 정보)
   - 테이블 별칭(Alias)을 사용해 하나의 테이블을 두 개의 다른 테이블처럼 보이게 만드는 것이 핵심
   - 상사가 없는 최상위 직원을 포함하려면 INNER JOIN 대신 LEFT JOIN을 사용

9. CROSS 조인
   - CROSS JOIN은 ON 조건 없이 한 테이블의 모든 행을 다른 테이블의 모든 행과 각각 연결
   - 두 테이블의 모든 경우의 수를 조합한 카테시안 곱(Cartesian Product) 결과
   - 상품의 사이즈 색상 재질 등 모든 옵션 조합을 생성할 때 유용
   - INSERT INTO ... SELECT 구문과 함께 사용하면 마스터 데이터를 쉽게 생성할 수 있음
   - 데이터가 많은 테이블에 사용하면 결과 행이 기하급수적으로 늘어나 서버에 심각한 부하를 줄 수 있으므로 주의

10. 조인 종합 실습
    - 실무 문제는 여러 테이블을 동시에 조인하고 여러 조건을 적용해야 해결할 수 있음
    - 문제를 작은 단위로 나누어 필요한 정보 테이블 조건과 정렬 순서를 파악하는 전략이 유용
    - JOIN 구문은 체인처럼 계속 이어서 여러 테이블을 연결할 수 있음
    - 여러 필터링 조건은 WHERE 절에 AND로 연결하고 최종 결과는 ORDER BY로 정렬
