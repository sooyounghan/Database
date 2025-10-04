-----
### 정리
-----
1. DDL - 테이블 생성
    - customers, products, orders 3개 테이블을 만들어 쇼핑몰 핵심 데이터를 저장
    - 기본 키는 AUTO_INCREMENT로 자동 번호를 부여
    - 날짜 시간 컬럼은 DEFAULT CURRENT_TIMESTAMP (ON UPDATE)로 자동 관리
    - 외래 키(FK)로 orders.customer_id → customers.customer_id, orders.product_id → products.product_id 관계 설정

2. DDL - 테이블 변경, 제거
    - 구조 변경은 ALTER TABLE ADD / MODIFY / DROP COLUMN 으로 수행
    - 대용량 테이블은 변경 시 잠금, 속도를 고려해 새벽에 작업
    - DROP TABLE은 테이블 자체를, TRUNCATE TABLE은 데이터만 삭제
    - FK가 걸려 있으면 두 명령 모두 차단
    - FK를 잠시 무시하려면 SET FOREIGN_KEY_CHECKS=0 후 작업하고 즉시 1로 돌릴 것

3. DML - 등록
    - INSERT는 (1) 모든 컬럼, (2) 필요 컬럼만, (3) 다중 VALUES 세 가지 패턴
    - 열 목록을 명시해서 스키마 변경 시 오류를 막을 것
    - AUTO_INCREMENT, DEFAULT가 있는 컬럼은 생략 가능
    - 물론 NULL을 입력할 수 있는 컬럼도 생략할 수 있음

4. DML - 수정
    - UPDATE … SET … WHERE 형태로 수정 (WHERE를 반드시 쓸 것)
    - MySQL Workbench는 기본으로 안전 업데이트 모드(SQL_SAFE_UPDATES=1)를 켜둠
    - 변경 전 동일 WHERE로 SELECT 해 확인할 것
    - 대량 수정이 필요하면 SET SQL_SAFE_UPDATES=0 로 잠시 끄고 끝나면 다시 킬 것

5. DML - 삭제
    - DELETE FROM … WHERE 로 행을 삭제 (WHERE를 반드시 쓸 것)
    - DELETE는 느리지만 조건 삭제, 트랜잭션 롤백 가능, AUTO_INCREMENT 유지
    - TRUNCATE는 빠르지만 조건 불가, 롤백 불가, AUTO_INCREMENT 초기화
    - 실행 전 반드시 SELECT 로 대상 행을 확인

6. 제약 조건 활용
    - NOT NULL : 필수값을 비워 두면 오류 발생
    - UNIQUE 키 : 중복 입력 시 오류 발생
    - FK : 부모에 없는 값을 넣으면 오류 발생
    -  INSERT, UPDATE, DELETE 모두 제약조건을 우선 검증하므로 데이터 무결성이 보호
