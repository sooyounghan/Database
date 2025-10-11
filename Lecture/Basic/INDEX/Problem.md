-----
### 문제와 풀이
-----
1. 문제와 풀이를 위한 인덱스 초기화
   - 기존에 존재하는 idx 로 시작하는 모든 인덱스를 제거
```sql
SHOW INDEX FROM items;
```
   - 실행 결과 idx 로 시작하는 모든 인덱스를 제거 (인덱스는 아쉽게도 IF EXISTS 구문이 없으므로, 직접 제거)
```sql
DROP INDEX idx_items_item_name ON items;
DROP INDEX idx_items_price_name ON items;
DROP INDEX idx_items_price ON items;
DROP INDEX idx_items_price_category_temp ON items;
DROP INDEX idx_items_category_price ON items;
```

2. 문제 : 인덱스들을 만들어서 다음 쿼리 성능을 개선
   - 최근 쇼핑몰의 items 테이블에 데이터가 많아지면서, 사용자들이 특정 조건으로 상품을 조회할 때 시스템이 점점 느려진다는 불만이 접수
   - 원인 파악 결과, 자주 사용되는 조회 쿼리에 인덱스가 걸려있지 않아 전체 데이터를 스캔(Full Table Scan) 중
   - 다음은 느리다고 보고된 주요 쿼리
```sql
SELECT * FROM items
WHERE category = '전자기기' AND is_active = TRUE;

SELECT * FROM items
WHERE category = '전자기기' AND is_active = TRUE
ORDER BY stock_quantity DESC;

SELECT * FROM items
WHERE stock_quantity < 90 AND category = '전자기기' AND is_active = TRUE;

SELECT * FROM items
WHERE stock_quantity < 90 AND category = '전자기기' AND is_active = TRUE
ORDER BY stock_quantity DESC;
```

3. 정답
```sql
CREATE INDEX idx_items_category_actvie_stock
ON items (category, is_active, stock_quantity DESC);
```
