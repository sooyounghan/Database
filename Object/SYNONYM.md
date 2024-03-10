-----
### 시노님(Synonym)
-----
1. 데이터베이스 객체는 각자 고유한 이름이 있는데, 이 객체들에 대해 동의어를 만드는 데이터베이스 객체
2. PUBLIC Synonym과 PRIVATE Synonym이 존재
   - PUBLIC : 데이터베이스 모든 사용자가 접근 가능 (DBA 권한이 있는 사용자만 생성, 삭제 가능)
   - PRIVATE : 특정 사용자에게만 참조되는 시노님
     
3. 형식
```sql
CREATE [OR REPLACE] [PUBLIC] SYNONYM [스키마명.]시노님명
FOR [스키마명.]객체명;
```

  - PUBLIC을 생략하면, PRIVATE 시노님이 생성됨
  - FOR절 이하 객체에는 테이블, 뷰, 프로시저, 함수, 패키지, 시퀀스 등 가능

4. 시노님 삭제
```sql
DROP [PUBLIC] SYNONYM [스키마명.]시노님명;
```

-----
### 시노님(Synonym) 예제
-----
1. CHANNELS 테이블이 있다고 하자. 이 테이블 객체에 대해 SYN_CHANNEL이라는 시노님을 붙인다.
```sql
CREATE [ON REPLACE] SYNONYM SYN_CHANNEL
FOR CHANNELS;
```
  - 어느 사용자나 SYN_CHANNEL로 CHANNELS 테이블 조회 가능
  - 하지만, 다른 사용자에게서는 이를 접근할 권한이 없으므로, 다른 사용자에게 SELECT 권한을 부여
    
```sql
GRANT SELECT ON SYN_CHANNEL TO 유저명;
```
```sql
SELECT COUNT(*)
FROM 소유자명.SYN_CHANNEL;
```

2. CHANNELS 테이블이 있다고 하자. 이 테이블 객체에 대해 SYN_CHANNEL이라는 PUBLIC 시노님을 붙인다.4
```sql
CREATE [ON REPLACE] PUBLIC SYNONYM SYN_CHANNEL
FOR CHANNELS;
```
  - 하지만, 다른 사용자에게서는 이를 접근할 권한이 없으므로, 다른 사용자에게 SELECT 권한을 부여
    
```sql
GRANT SELECT ON SYN_CHANNEL TO 유저명;
```
```sql
SELECT COUNT(*)
FROM SYN_CHANNEL;
```

3. PUBLIC과 PRIVATE 설정 시 차이점
   - PRIVATE로 설정한다면, 소유자명.시노님명
   - PUBLIC로 설정한다면, 시노님명 (만든 주체가 PUBLIC이므로)


-----
### 시노님(Synonym) 사용 이유
-----
1. 데이터베이스의 투명성 제공 (즉, 다른 사용자의 객체를 참조할 때 많이 사용)
2. 객체의 이름이 바뀌더라도 이전에 작성한 SQL문을 수정할 필요가 없음
3. 원 객체를 숨길 수 있어 보안 측면에서 유리 (PRIVATE은 소유자명을 명시해야하지만, PUBLIC은 숨길 수 있음)
