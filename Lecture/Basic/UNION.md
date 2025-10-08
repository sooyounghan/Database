-----
### UNION
-----
1. JOIN, 서브쿼리 : 이 기술들의 공통점은 기존 테이블의 정보를 조합하거나 필터링해서, 우리가 원하는 형태의 '하나의 결과 집합(Result Set)'을 만들어내는 것
2. JOIN이 여러 테이블을 옆으로(수평으로) 붙여서 더 많은 정보를 가진 컬럼들을 만드는 기술이었다면, UNION은 여러 개의 결과 집합을 아래로(수직으로) 이어 붙여서 더 많은 행을 가진 하나의 집합으로 만드는 기술
3. JOIN이 정보를 풍성하게 만드는 기술이라면, UNION은 흩어진 집합들을 하나로 모으는 기술
4. 상황 : 쇼핑몰은 현재 활동 중인 고객을 users 테이블에, 과거에 탈퇴한 고객을 retired_users라는 별도의 테이블에 보관
   - 연말을 맞아 모든 고객(활동 + 탈퇴)에게 감사 이메일을 보내기 위해, 두 테이블에 흩어져 있는 이름과 이메일을 합쳐서 하나의 전체 목록을 만들어야 함
   - 이 업무는 JOIN 으로는 해결할 수 없는데, 두 테이블은 서로 연결된 관계가 아니라, 구조는 비슷하지만 분리된 별개의 집합이기 때문임

5. 실습 준비 : retired_users 테이블 생성
   - '탈퇴한 고객' 정보를 담을 retired_users 테이블을 임시로 생성
   - 일부러 현재 활동 고객인 '션'을 탈퇴 고객 명단에도 포함시킬 것 : '션'은 users 테이블에도 포함되고 retired_users 테이블에도 포함

```sql
-- 본 실습을 위한 탈퇴 고객 테이블 생성
DROP TABLE IF EXISTS retired_users;
CREATE TABLE retired_users (
   id BIGINT PRIMARY KEY,
   name VARCHAR(255) NOT NULL,
   email VARCHAR(255) NOT NULL,
   retired_date DATE NOT NULL
);

-- 탈퇴 고객 데이터 입력
INSERT INTO retired_users (id, name, email, retired_date) VALUES
(1, '션', 'sean@example.com', '2024-12-31'),
(7, '아이작 뉴턴', 'newton@example.com', '2025-01-10');
```

6. UNION의 개념과 사용법
   - 활동 고객 명단 (users 테이블)
```sql
SELECT name, email
FROM users;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/825745c0-f705-497b-ad22-fbc2705b9261">
</div>

   - 탈퇴 고객 명단 (retired_users 테이블)
```sql
SELECT name, email
FROM retired_users;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/0e61121f-bb8f-441e-9d75-57cc8ff3eb77">
</div>

   - UNION 연산자는 이 두 개의 SELECT 문의 결과를 하나로 합쳐줌
   - 두 SELECT 문 사이에 UNION 키워드를 넣어주기만 하면 됨
```sql
SELECT name, email FROM users
UNION
SELECT name, email FROM retired_users;
```

7. UNION 사용의 핵심 규칙
   - UNION으로 연결되는 모든 SELECT 문은 컬럼의 개수가 동일해야 함
   - 각 SELECT 문의 같은 위치에 있는 컬럼들은 서로 호환 가능한 데이터 타입이어야 함 (예) 숫자 타입은 숫자 타입끼리, 문자 타입은 문자 타입끼리)
   - 최종 결과의 컬럼 이름은 첫 번째 SELECT 문의 컬럼 이름을 따름
<div align="center">
<img src="https://github.com/user-attachments/assets/5fde1e25-6bd8-4ca7-9105-e1c614507a29">
</div>

   - '션'은 users 테이블에도 있고 retired_users 테이블에도 있었지만, 최종 결과 목록에는 단 한 번만 나타남
   - 💡 UNION의 특징 : UNION 은 기본적으로 두 결과 집합을 합친 뒤, 완전히 중복되는 행은 자동으로 제거하여 고유한 값만 남김
      + 이메일 목록을 만들 때 중복된 주소로 두 번 발송하는 실수를 원천적으로 방지해주니 매우 유용한 기능
   - 하지만 의도적으로 중복을 허용해야 하거나, 데이터에 중복이 없다는 것을 이미 알아서 굳이 중복 검사를 할 필요가 없는 경우도 있는데, 바로 이럴 때 UNION ALL을 사용
