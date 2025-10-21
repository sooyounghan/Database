-----
### 다대다(M:N) 관계 - 속성
-----
1. 관계의 속성(Attribute)과 연결 테이블
   - 다대다 관계를 표현하는 연결 테이블은 단순히 두 테이블을 이어주는 기술적인 역할만 하는 것이 아님
   - 대부분의 경우, 이 연결 테이블은 관계 자체에 대한 추가적인 속성을 가지게 됨
     + 예를 들어, 회원이 상품을 주문하는 행위
     + 주문 당시의 상품 가격은 얼마였는가? (order_price) : 상품의 현재 가격은 할인이나 정책에 따라 계속 변할 수 있으므로, 주문이 일어난 시점의 가격을 반드시 기록해야 함
     + 몇 개를 주문했는가? (order_quantity) : 주문 수량은 주문과 상품의 관계에서만 의미가 있는 매우 중요한 정보
     + 이러한 order_price나 order_quantity 같은 정보들은 orders 테이블에 속할 수 없음 : 하나의 주문에는 여러 상품이 있으니 어떤 상품의 가격인지 알 수 없음
     + 그럼 product 테이블에 속할 수 없음 : 상품 자체의 정보가 아니라 '주문될 때'의 정보이기 때문임

   - 이 정보들은 orders와 product가 만나는 관계 그 자체에 속하는 속성들 : 그리고 이 속성들을 저장할 완벽한 장소가 바로 연결 테이블
   
2. 이처럼 관계 자체에 속성이 존재하는 경우가 대부분
   - 따라서 개념적 모델링 단계에서부터 다대다 관계가 보이면, 그 관계가 어떤 속성을 가질지 고민해야 함
   - 만약 속성이 존재한다면, 그 연결 테이블은 단순히 관계를 풀어내기 위한 기술적 장치가 아니라, '주문항목'이라는 의미를 가지는 하나의 독립적인 연관 엔티티(Associative Entity)로 설계해야 함
   - 사실상 이론적으로는 속성이 없는 순수한 다대다 관계도 존재할 수 있지만, 실무에서 마주하는 대부분의 다대다 관계는 이처럼 속성을 가지는 경우가 대부분
   - 그리고 이는 개념적 모델링 단계에서부터 이미 엔티티로 도출되는 것이 일반적
   - 💡 즉, 연결 테이블은 단순히 기술적인 해결책이 아니라, 그 자체로 중요한 비즈니스 의미를 갖는 핵심 엔티티인 경우가 많음

3. 정리하면 이론적으로 개념적 모델의 다대다 관계가 논리적 모델링 단계에서 연결 테이블로 풀어진다고 설명하지만, 실무에서는 대부분의 다대다 관계가 이런 '관계 속성'을 가지고 있음
   - 따라서 개념적 모델링 단계에서부터 이미 '주문 항목(Order Item)'이라는 별도의 엔티티로 식별되는 경우가 대부분임

4. 참고 : 연관 엔티티 추천 이름 및 특징
   - 주문 항목(order_item)은 주문(order)과 상품(product)의 관계에서 나온 연관 엔티티 : 연관 엔티티의 이름을 지을 때는 크게 2가지 방법
   - 연결 강조 : 주문 상품(order_product)
      + '주문'과 '상품' 테이블을 직접적으로 연결한다는 관계를 이름에 명시적으로 보여줌
      + 매우 직관적이고 단순하여 관계를 파악하기 쉽다는 장점
   - 의미 있는 이름 : 주문 항목(order_item)
      + '주문'에 포함된 '항목'이라는 의미를 가장 명확하게 전달
      + 실제 쇼핑몰 비즈니스 로직(예) 장바구니 항목, 주문 항목)과 용어가 일치하여 이해하기 쉬움
   - 의미 있는 이름 : 주문 상세(order_detail)
      + '주문의 상세 내역'이라는 의미로, order_item과 거의 동일한 의미로 널리 사용
      + 주문에 속한 각 상품 정보를 상세히 나타낸다는 점을 강조

   - 💡 실무에서는 의미 있는 이름을 사용하는 것이 좋음
     + 데이터베이스는 단순히 데이터를 저장하는 창고가 아니라 비즈니스 모델을 반영하는 설계도이기 때문임
     + 만약 관계가 단순하고 의미 있는 이름을 짓기 어렵다면 양쪽 엔티티의 이름을 사용하는 연결을 강조하는 방식을 선택할 수 있음

   - 앞서 연결 테이블의 이름을 order_product로 지은 것은 이 테이블이 정말로 단순히 연결만을 강조했기 때문임
   - 그리고 그 안에는 연결 정보를 제외한 어떤 정보도 없었음 : 예를 들면 주문 당시의 상품 가격(order_price), 주문 갯수(order_quantity) 같은 정보가 없었음

5. 다대다 관계 구현 : order_item 테이블
   - 연결 테이블이 단순한 연결을 넘어서 상품 가격(order_price), 주문 갯수(order_quantity)같은 비즈니스 속성을 가지도록 다시 정의해보자. 이제 단순히 관계만 풀어내는 테이블이 아니라, 비즈니스 의미가 있는 테이블이 됨
   - 이렇게 정의하면 비즈니스 모델을 반영하는 의미있는 이름을 사용하는 것이 더 나은 선택 : 주문 항목(order_item)이라는 이름을 사용
      + order_product : 기술적으로 다대다 관계를 풀어내는 것에 초점
      + order_item : 비즈니스 속성과 의미를 가짐, 기술적으로 다대다 관계도 풀어냄
```sql
DROP TABLE IF EXISTS order_item;

CREATE TABLE order_item (
   order_item_id BIGINT NOT NULL AUTO_INCREMENT, -- 주문 상품 ID (PK)
   order_id BIGINT NOT NULL, -- 주문 ID (FK from orders)
   product_id BIGINT NOT NULL, -- 상품 ID (FK from product)
   order_price INT NOT NULL, -- 주문 당시 가격 (관계 속성)
   order_quantity INT NOT NULL, -- 주문 수량 (관계 속성)
  
   PRIMARY KEY (order_item_id),
   -- 한 주문에 동일한 상품이 중복으로 들어가는 것을 방지
   CONSTRAINT uq_order_item UNIQUE (order_id, product_id),
   CONSTRAINT fk_order_item_orders FOREIGN KEY (order_id)
   REFERENCES orders (order_id),
   CONSTRAINT fk_order_item_product FOREIGN KEY (product_id)
   REFERENCES product (product_id)
);
```
   - order_item_id는 이 테이블 고유의 대리 키 PK
   - (order_id, product_id) 유니크 제약조건을 둬서 하나의 주문에서 같은 상품을 주문하는 중복 주문행을 차단
   - order_id는 orders 테이블의 PK를 참조하는 FK
   - product_id는 product 테이블의 PK를 참조하는 FK
   - order_price와 order_quantity는 관계 자체의 속성

   - order_item 테이블은 order_id와 product_id를 외래 키로 받아 두 테이블을 연결하고, order_price와 order_quantity라는 관계의 속성을 저장
   - 이 order_item 테이블을 통해, 우리는 이제 어떤 주문에 어떤 상품들이 얼마의 가격으로 몇 개씩 포함되었는지 정확하고 명확하게 저장하고 조회할 수 있게 되었음

6. 식별, 비식별 관계
   - order_item 테이블은 고유의 order_item_id를 기본 키로 사용하는 비식별 관계 모델
   - 부모 테이블의 기본 키를 물려받아 (order_id, product_id) 를 복합 기본 키로 사용하는 식별 관계 모델도 존재
