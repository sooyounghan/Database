-----
### 식별 관계 vs 비식별 관계: 다대다 관계에서의 확장성 문제
-----
1. 소프트웨어는 끊임없이 변하는 비즈니스 요구에 대응해야 함
   - 처음엔 단순했던 M:N 관계가 더 복잡한 규칙을 수용해야 할 때, 두 설계 방식의 유연성은 큰 차이를 보임
   - 새로운 요구사항 : "고객이 장바구니에 같은 티셔츠를 담더라도, 'L 사이즈, 흰색'과 'M 사이즈, 검은색'처럼 옵션이 다르면 별개의 상품으로 주문할 수 있게 할 것"
   - 이 요구사항은 기존의 (주문, 상품) 관계에 '옵션'이라는 새로운 차원을 추가해야 함을 의미 : 이제 중복을 판단하는 기준이 (order_id, product_id) 에서 (order_id, product_id, option_id)로 변경되어야 함

2. 비식별 관계의 경우 - 유연한 확장
   - 비식별 관계에선 order_item 테이블의 정체성(PK)이 order_item_id라는 독립적인 값에 의해 결정
   - 따라서 새로운 비즈니스 규칙을 추가하는 작업이 훨씬 간단하고 안전
   - 변경 작업
     + order_item 테이블에 option_id 컬럼을 추가
     + 기존의 (order_id, product_id) 조합에 대한 UNIQUE 제약조건을 삭제
     + 새로운 비즈니스 규칙에 맞는 (order_id, product_id, option_id) 조합에 대한 UNIQUE 제약조건을 추가
```sql
-- 1. 새로운 컬럼 추가 (상품에 옵션이 없는 경우도 있으므로 NULL 허용)
ALTER TABLE order_item_non_identifying
ADD COLUMN option_id BIGINT NULL;

-- 2. 기존 UNIQUE 제약조건 삭제
ALTER TABLE order_item_non_identifying
DROP KEY uq_order_item;

-- 3. 새로운 UNIQUE 제약조건 추가
ALTER TABLE order_item_non_identifying
ADD UNIQUE KEY uq_order_item_option (order_id, product_id, option_id);
```

   - 참고 : MySQL은 NULL을 서로 다른 값으로 취급하므로 동일 주문, 상품, NULL이 중복 허용된다는 점을 주의
   - MySQL의 경우 다음과 같이 외래 키 제약조건을 먼저 삭제해야 함
```sql
-- 외래 키(Foreign Key) 제약조건 삭제 (MySQL에서는 인덱스 변경 전 FK를 먼저 삭제해야 함)
ALTER TABLE order_item_non_identifying
DROP FOREIGN KEY fk_oi_non_orders;
ALTER TABLE order_item_non_identifying
DROP FOREIGN KEY fk_oi_non_product;

-- 1. 새로운 컬럼 추가 (상품에 옵션이 없는 경우도 있으므로 NULL 허용)
ALTER TABLE order_item_non_identifying
ADD COLUMN option_id BIGINT NULL;

-- 2. 기존 UNIQUE 제약조건 삭제
ALTER TABLE order_item_non_identifying
DROP INDEX uq_order_item;

-- 3. 새로운 UNIQUE 제약조건 추가, option_id 추가
ALTER TABLE order_item_non_identifying
ADD UNIQUE KEY uq_order_item_option (order_id, product_id, option_id);

-- 외래 키 제약조건 다시 추가
ALTER TABLE order_item_non_identifying ADD CONSTRAINT fk_oi_non_orders
FOREIGN KEY (order_id) REFERENCES orders_non_identifying(order_id);

ALTER TABLE order_item_non_identifying ADD CONSTRAINT fk_oi_non_product
FOREIGN KEY (product_id) REFERENCES product_non_identifying(product_id);
```

   - 중요한 점은 테이블의 PK인 order_item_id 는 전혀 변경되지 않았다는 것
   - 테이블의 정체성은 그대로 유지한채, 비즈니스 규칙을 담당하는 UNIQUE 제약조건만 유연하게 교체
   - 요구사항 반영 : 이제 동일한 주문과 상품에 대해 옵션만 다르게 하여 여러 데이터를 추가할 수 있음
```sql
-- order_id=100, product_id=1 (티셔츠)에 대해 옵션만 다르게 주문
INSERT INTO order_item_non_identifying(order_id, product_id, option_id, order_price, order_quantity)
VALUES (100, 1, 10, 15000, 2); -- option_id 10: L 사이즈, 흰색

INSERT INTO order_item_non_identifying(order_id, product_id, option_id, order_price, order_quantity)
VALUES (100, 1, 15, 15000, 1); -- option_id 15: M 사이즈, 검은색
```
   - 실행 결과 : 성공적으로 2개의 행이 삽입
     + (order_id, product_id) 는 (100, 1) 로 동일하지만, option_id가 다르기 때문에 새로운 UNIQUE 제약조건을 위반하지 않음
<div align="center">
<img src="https://github.com/user-attachments/assets/bdfec21a-48d5-455d-b4b7-388d9e2974ef">
</div>

3. 식별 관계의 경우 - 깨지기 쉬운 구조
   - 식별 관계에선 (order_id, product_id) 자체가 PK이므로, '하나의 주문에 동일한 상품은 하나만'이라는 규칙이 PK 레벨에 딱 붙어 있음
   - 새로운 요구사항은 이 PK 제약조건을 정면으로 위반
   - 변경 작업 : 이 문제를 해결하려면 테이블의 PK, 즉 정체성 자체를 변경해야 함
     + 테이블에 option_id 컬럼을 추가한다.
     + 기존의 (order_id, product_id) 복합 키(PK)를 삭제
     + option_id까지 포함된 새로운 3컬럼 복합 키(PK) (order_id, product_id, option_id)를 생성

   - 문제점
     + 이것은 단순히 컬럼 하나를 추가하거나 제약조건을 바꾸는 수준이 아님 : 테이블의 근간이자 식별자인 PRIMARY KEY를 변경하는 매우 위험하고 복잡한 작업
     + 만약 다른 테이블에서 이 복합키를 참조하고 있었다면, 그곳까지 변경의 여파가 미침
     + 서비스가 운영 중인 상황에서 PK를 변경하는 것은 상당한 부담과 위험을 동반

   - 이 예시는 M:N 관계에서도 비식별 관계가 비즈니스 변화에 훨씬 유연하고 확장성 있게 대응할 수 있음을 명확하게 보여줌

-----
### 다대다 정리
-----
1. 결론적으로, 다대다 관계를 해결하는 연결 테이블은 비식별 관계로 설계하는 것이 훨씬 더 유연하고 확장성이 높음
   - 비식별 관계 : 독립적인 대리 키를 사용하므로, 비즈니스 규칙이 변경되어도 테이블의 PK를 건드릴 필요가 없음
     + 새로운 정보가 필요하면 컬럼만 추가하고, 비즈니스 규칙의 변경은 UNIQUE 제약조건을 수정하여 대응하면 됨
     + 이는 기존 테이블을 참조하는 다른 관계에 영향을 주지 않아 훨씬 안전

   - 식별 관계 : 복합키 자체가 비즈니스 규칙을 강제하기 때문에, 해당 규칙이 변경되면 PK 구조 자체를 바꿔야 함
     + 이는 매우 위험하고 번거로운 작업이며, 테이블의 확장성을 심각하게 저해  

2. 실무에서는 M:N 관계를 해소하는 연결 테이블에 비식별 관계를 사용하는 것이 최고의 선택
   - PK는 별도의 대리 키를 사용하고 외래키 (FK1, FK2, ...) 조합에 UNIQUE 제약조건을 걸어 비즈니스 중복을 제거

3. 대리 키 + 비식별 관계
   - 현대 애플리케이션 개발에서는 '기본적으로 모든 관계는 비식별 관계로 설계하고, PK는 비즈니스와 무관한 대리 키를 사용한다' 는 원칙을 따르는 것이 현명한 선택
   - 이것이 변화에 유연하게 대처하고 유지보수하기 좋은 시스템을 만드는 현대적인 설계 방법

4. 대리 키-PK, 자연 키-UNIQUE, 관계-비식별 : 현대적인 데이터베이스 설계는 대리 키-PK, 자연 키-UNIQUE, 관계-비식별 방식이 사실상 표준
