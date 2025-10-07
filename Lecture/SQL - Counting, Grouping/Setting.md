-----
### 집계와 그룹핑 실습 데이터 준비
-----
1. order_stat (주문 통계)라는 이름의 테이블을 만들고, 여기에 분석에 충분한 샘플 데이터를 채워 넣을 것
2. 테이블 생성 (CREATE TABLE)
```sql
CREATE TABLE order_stat (
     order_id INT PRIMARY KEY AUTO_INCREMENT,
     customer_name VARCHAR(50),
     category VARCHAR(50),
     product_name VARCHAR(100),
     price INT,
     quantity INT,
     order_date DATE
);
```

3. 샘플 데이터 삽입 (INSERT INTO)
```sql
INSERT INTO order_stat (customer_name, category, product_name, price, quantity, order_date)
VALUES
    ('이순신', '전자기기', '프리미엄 기계식 키보드', 150000, 1, '2025-05-10'),
    ('세종대왕', '도서', 'SQL 마스터링', 35000, 2, '2025-05-10'),
    ('신사임당', '가구', '인체공학 사무용 의자', 250000, 1, '2025-05-11'),
    ('이순신', '전자기기', '고성능 게이밍 마우스', 80000, 1, '2025-05-12'),
    ('세종대왕', '전자기기', '4K 모니터', 450000, 1, '2025-05-12'),
    ('장영실', '도서', '파이썬 데이터 분석', 40000, 3, '2025-05-13'),
    ('이순신', '문구', '고급 만년필 세트', 200000, 1, '2025-05-14'),
    ('세종대왕', '가구', '높이조절 스탠딩 데스크', 320000, 1, '2025-05-15'),
    ('신사임당', '전자기기', '노이즈캔슬링 블루투스 이어폰', 180000, 1, '2025-05-15'),
    ('장영실', '전자기기', '보조배터리 20000mAh', 50000, 2, '2025-05-16'),
    ('홍길동', NULL, 'USB-C 허브', 65000, 1, '2025-05-17'); -- 카테고리가 NULL인 데이터 추가
```

3. 실무에서 데이터는 종종 누락, 오류가 섞인다. 때로는 시스템 오류나 사람의 실수로 특정 값이 누락된 채 저장되기도 함
4. 물론 정상적으로 의도한 결과일 수 도 있으나, 이런 '누락된 데이터(NULL)'가 집계에 어떤 영향을 미치는지 직접 확인해야 함
5. 그래서 일부러 마지막에 category 가 NULL 인 데이터를 하나 추가
6. 결과 확인
```sql
SELECT *
FROM order_stat;
```
  - order_stat - 데이터
<div align="center">
<img src="https://github.com/user-attachments/assets/b482cba8-1211-47a1-8ece-43e59ee7c689">
</div>

7. JOIN : 주문 통계(order_stat) 테이블은 앞서 설계한 customers, orders, products 테이블을 모두 합쳐(JOIN)놓은 것 같아 보ㅇㅁ
