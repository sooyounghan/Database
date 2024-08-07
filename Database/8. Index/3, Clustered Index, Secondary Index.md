-----
### 클러스터형 인덱스와 보조 인덱스의 구조
-----
1. 인덱스가 없이 테이블 생성하고, 데이터 입력
```sql
CREATE DATABASE IF NOT EXISTS testdb;
USER testdb;
DROP TABLE IF EXISTS clustertbl;

CREATE TABLE clusterTBL
( userID      char(8) ,
  userName    varchar(10) 
);

INSERT INTO clusterTBL VALUES('YJS', '유재석');
INSERT INTO clusterTBL VALUES('KHD', '강호동');
INSERT INTO clusterTBL VALUES('KKJ', '김국진');
INSERT INTO clusterTBL VALUES('KYM', '김용만');
INSERT INTO clusterTBL VALUES('KJD', '김제동');
INSERT INTO clusterTBL VALUES('NHS', '남희석');
INSERT INTO clusterTBL VALUES('SDY', '신동엽');
INSERT INTO clusterTBL VALUES('LHJ', '이휘재');
INSERT INTO clusterTBL VALUES('LKK', '이경규');
INSERT INTO clusterTBL VALUES('PSH', '박수홍');
```

2. 만약, 페이지 당 4개의 행이 입력된다고 가정하면, 다음과 같이 구성
<div align="center">
<img src="https://github.com/user-attachments/assets/56dcce98-a930-469e-ac55-a5b8e9ca1438">
</div>

   - MySQL의 기본 페이지의 크기는 16KB (=16384 Byte)이므로 훨씬 더 많은 행 데이터가 들어가지만, 이해를 돕기 위해 가정한 것
   - MySQL 페이지 크기는 SHOW VARAIBLES LIKE 'innodb_page_size'; 문으로 확인 가능하며, 필요하다면, 4KB, 8KB, 16KB, 32KB, 64KB로 변경 가능

3. 정렬된 순서만 확인해보면, 입력된 것과 동일한 순서
```sql
SELECT *
FROM clustertbl;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c637ef63-1b27-45b8-9b12-8348b6db0d2c">
</div>

4. 이 테이블의 userID에 클러스터형 인덱스를 구성해보자
   - userID를 Primary Key로 지정하면 클러스터형 인덱스로 구성
```sql
ALTER TABLE clustertbl
ADD CONSTRAINT PK_clustertbl_userID
PRIMARY KEY (userID);
```
  - 테이블 다시 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/7441cb2a-af3b-4f6a-b331-d84587f98553">
</div>  

  - userID는 오름차순 정렬이 되어있음 (Primary Key로 지정했으니 클러스터형 인덱스가 생성되었기 때문임)
  - 실제 데이터는 아래와 같이 데이터 페이지가 정렬되고, B-Tree 형태의 인덱스가 형성
<div align="center">
<img src="https://github.com/user-attachments/assets/65227a26-9e9e-4ea3-94f4-b20c2f58b9da">
</div>

  - 클러스터형 인덱스를 구성하기 위해 행 데이터를 해당 열로 정렬한 후 루트 페이지를 다시 만들게 됨
  - 클러스터형 인덱스는 루트 페이지와 리프 페이지 (중간 페이지가 있다면 중간 페이지도 포함)로 인덱스가 구성
    + 동시에, 인덱스 페이지의 리프 페이지는 데이터 그 자체임

5. 클러스터형 인덱스는 데이터의 검색 속도가 보조 인덱스보다 더 빠름

6. 보조 인덱스 생성
```sql
CREATE DATABASE IF NOT EXISTS testdb;
USE testdb;

DROP TABLE IF EXISTS secondarytbl;
CREATE TABLE secondarytbl - Secondary Table 약자
( userID  CHAR(8),
  name    VARCHAR(10)
);

INSERT INTO mixedtbl VALUES('LSG', '이승기');
INSERT INTO mixedtbl VALUES('KBS', '김범수');
INSERT INTO mixedtbl VALUES('KKH', '김경호';
INSERT INTO mixedtbl VALUES('JYP', '조용필');
INSERT INTO mixedtbl VALUES('SSK', '성시경');
INSERT INTO mixedtbl VALUES('LJB', '임재범');
INSERT INTO mixedtbl VALUES('YJS', '윤종신');
INSERT INTO mixedtbl VALUES('EJW', '은지원');
INSERT INTO mixedtbl VALUES('JKW', '조관우');
INSERT INTO mixedtbl VALUES('BBK', '바비킴');
```
<div align="center">
<img src="https://github.com/user-attachments/assets/56dcce98-a930-469e-ac55-a5b8e9ca1438">
</div>

  - 위 그림과 동일한 구조 형성
  - Unique 제약 조건은 보조 인덱스를 생성하는 것을 확인했으므로, userID열에 Unique 제약 조건 지정`
```sql
ALTER TABLE secondarytbl
ADD CONSTRAINT UK_secondarytbl_userID
UNIQUE (userID);
```

  - 데이터 순서 확인
```sql
SELECT *
FROM clustertbl;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/37a802e9-f979-450c-907a-a7e52837148f">
</div>

  - 입력한 것과 순서의 변화가 없으며, 내부적으로 아래와 같이 구성
<div align="center">
<img src="https://github.com/user-attachments/assets/55fdd48f-6b12-47aa-bc50-806f0d44ce32">
</div>

  - 💡 보조 인덱스는 데이터 페이지를 건드리지 않고, 별도의 장소에 인덱스 페이지를 생성
  - 💡 우선, 인덱스 페이지의 리프 페이지에 인덱스로 구성한 열(userID)을 정렬
  - 💡 그리고 데이터 위치 포인터를 생성
    + 데이터 위치 포인터는 클러스터형 인덱스와 달리 주소값(페이지 번호 + #오프셋)이 기록되어 바로 데이터 위치 가리킴
    + BBK를 예를 들면, 1002번 페이지에 두 번쨰(#2)에 데이터가 있다고 기록
    + 따라서, 이 데이터의 위치 포인터는 데이터가 위치한 고유한 값
  - 💡 물론, 한 테이블에 클러스터형 인덱스와 보조 인덱스가 함께 존재하면, 구성이 조금 달라짐

7. 이 상황에서 데이터를 검색(SELECT 문)
   - 💡 클러스터형 인덱스에서 검색(예) JKW(조관우)를 검색한다면, 루트 페이지(100번)와 리프 페이지(=데이터 페이지, 1000번) 한 개씩만 검색하므로, 총 2개 페이지를 읽음
   - 💡 보조 인덱스에서 검색하면, JKW를 검색할 시, 인덱스 페이지의 루트 페이지(10번), 리프 페이지(100번), 그리고 데이터 페이지 (1002번)를 읽게 되어 3개 페이지를 읽음
   - 만약, 범위를 검색하는 것을 고려한다면? (예) userID가 'A' ~ 'J'인 사용자를 모두 검색)
     + 💡 클러스터형 인덱스의 경우 루트 페이지(100번)와 1000번 페이지 2개 페이지만 읽으면 원하는 데이터가 모두 들어있음 (리프 페이지가 곧 데이터 페이지이므로, 클러스터형 인덱스는 범위로 검색 시 아주 우수한 성능을 보임)
     + 💡 보조 인덱스의 경우, 우선 루트 페이지와 리프 페이지 중 100번 페이지를 읽으면 됨
     + 그러나 데이터를 검색하려하니 범위에 해당하는 BBK와 EJW, JYP가 서로 다른 페이지에 존재하므로, 1000페이지, 1001페이지, 1002페이지를 읽어야 함
     + 즉, 보조 인덱스는 10번, 100번, 1000번, 1001번, 1002번 페이지를 읽어 총 5개 페이지를 읽음
   - 즉, 결론적으로 클러스터형 인덱스가 보조 인덱스보다 검색이 빠르다고 볼 수 있음

8. 이번에는 클러스터형 인덱스에 새로운 데이터를 입력
```sql
INSERT INTO clustertbl VALUES('FNT', '푸니타');
INSERT INTO clustertbl VALUES('KAI', '카아이');
```
<div align="center">
<img src="https://github.com/user-attachments/assets/216091a2-379e-40e8-9bc4-8ac5f329bec7">
</div>

  - 첫 번쨰 리프 페이지(=데이터 페이지)가 페이지 분할 발생
  - 그리고 데이터를 공평한 후, 루트 페이지에 등록
  - 💡 루트 페이지의 순서가 약간 변경이 되었으나, 페이지 분할에 비해서 같은 페이지 내에서의 순서 변경은 시스템에 미치는 영향은 미미
  - 💡 오히려 페이지 분할이 시스템에 많은 부하를 줌

9. 이번에는 보조 인덱스에 동일한 입력
```sql
INSERT INTO secondatytbl VALUES('FNT', '푸니타');
INSERT INTO secondarytbl VALUES('KAI', '카아이');
```
<div align="center">
<img src="https://github.com/user-attachments/assets/bd2d5823-b8a4-433a-91ad-4081c90632d2">
</div>

  - 💡 보조 인덱스는 데이터 페이지를 정렬하는 것이 아니므로, 데이터 페이지의 뒤쪽 부분에 삽입
  - 💡 그리고 인덱스의 리프 페이지에도 약간의 위치가 조정된 것일 뿐, 페이지 분할은 일어나지 않음
  - 💡 결국, 클러스터형 인덱스보다 데이터 입력에서는 성능에 주는 부하가 더 적음
  - 실제로, 대용량 테이블일 경우에는 INSERT 작업이 대개는 클러스터형 인덱스가 더 시스템 부하가 많이 생김

10. 클러스터형 인덱스의 특징
    - 클러스터형 인덱스가 생성 시에는 데이터 페이지 전체가 다시 정렬
    - 이미 대용량의 데이터가 입력된 상태라면, 업무 시간에 클러스터형 인덱스를 생성하는 것은 심각한 시스템 부하를 줄 수 있으므로 신중해야함
    - 💡 클러스터형 인덱스의 인덱스 자체의 리프 페이지가 곧 데이터 (즉, 인덱스 자체에 데이터가 포함되어 있다고 볼 수 있음)
    - 💡 클러스터형 인덱스는 보조 인덱스보다 검색 속도가 빠르지만, 데이터의 입력 / 수정 / 삭제는 더 느림
    - 클러스터형 인덱스는 성능은 좋지만 테이블에 한 개만 생성 가능
    - 그러므로, 어느 열에 클러스터형 열을 생성하는지에 따라 시스템 성능이 달라질 수 있음

11. 보조 인덱스의 특징
    - 보조 인덱스의 생성 시 데이터 페이지는 그냥 둔 상태에서 별도의 페이지에 인덱스를 구성
    - 💡 보조 인덱스는 인덱스 자체의 리프 페이지는 데이터가 아니라 데이터가 위치하는 주소값(RID)
    - 💡 클러스터형 인덱스보다 검색 속도가 더 느리지만, 데이터 입력 / 수정 / 삭제는 덜 느림
    - 보조 인덱스는 여러 개 생성 가능
    - 하지만, 함부로 남용할 경우 오히려 시스템 성능을 떨어뜨리는 결과를 초래하므로 필요한 열에만 생성하는 것이 좋음

-----
### OLTP와 OLAP에 인덱스 생성
-----
1. OLTP(On-Line Transaction Processing)는 INSERT / UPDATE / DELETE가 자주 발생되므로, 꼭 필요한 인덱스만 최소로 생성하는 것이 바람직
2. 하지만, OLAP (On-Line Analytical Processing)는 INSERT / UPDATE / DELETE가 별로 사용될 일이 없으므로 인덱스를 많이 만들어도 별 문제가 되지 않음
3. 하나의 DB가 OLTP / OLAP 겸용으로 사용한다면, 두 개를 분리하는 방법을 고려하는 것이 전반적인 시스템 성능에 도움이 됨

-----
### 클러스터형 인덱스와 보조 인덱스가 혼합되어 있을 경우
-----
1. 현실적으로는 하나의 테이블에 클러스터형 인덱스와 보조 인덱스가 혼합되어 있는 경우가 많음
2. 이번에는 열이 3개인 테이블 사용
   - userID열에는 클러스터형 인덱스를, name열에는 보조 인덱스를 생성
   - addr열은 참조용으로 사용
```sql
CREATE DATABASE IF NOT EXISTS testdb;
USE testdb;

DROP TABLE IF EXISTS mixedtbl;
CREATE TABLE mixedtbl
( userID  CHAR(8) NOT NULL,
  name    VARCHAR(10) NOT NULL,
  addr    char(2)
);

INSERT INTO mixedtbl VALUES('LSG', '이승기', '서울');
INSERT INTO mixedtbl VALUES('KBS', '김범수', '경남');
INSERT INTO mixedtbl VALUES('KKH', '김경호', '전남');
INSERT INTO mixedtbl VALUES('JYP', '조용필', '경기');
INSERT INTO mixedtbl VALUES('SSK', '성시경', '서울');
INSERT INTO mixedtbl VALUES('LJB', '임재범', '서울');
INSERT INTO mixedtbl VALUES('YJS', '윤종신', '경남');
INSERT INTO mixedtbl VALUES('EJW', '은지원', '경북');
INSERT INTO mixedtbl VALUES('JKW', '조관우', '경기');
INSERT INTO mixedtbl VALUES('BBK', '바비킴', '서울');
```

  - 만약, 페이지당 4개의 행이 입력된다고 가정하면 다음과 같이 구성
<div align="center">
<img src="https://github.com/user-attachments/assets/e0a622e5-f3fd-4c7f-af33-0176442d2b14">
</div>

3. 이 테이블의 userID열에 클러스터형 인덱스를 생성 (Primary Key로 지정하면, Default로 클러스터형 생성)
```sql
ALTER TABLE mixedtbl
ADD CONSTRAINT PK_mixedtbl_userID
PRIMARY KEY (userID);
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f8880e4f-a92a-4c0f-8e15-7c30d308960b">
</div>

4. 이번에는 UNIQUE 제약 조건 추가
```sql
ALTER TABLE mixedtbl
ADD CONSTRAINT UK_mixedtbl_name
UNIQUE (name);
```

5. 클러스터형 인덱스와 보조 인덱스 생성 여부 확인
```sql
SHOW INDEX FROM mixedtbl;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/efe4cc8f-1f73-43d7-bdea-ef90abf36076">
</div>

6. 두 인덱스가 혼합된 내부 구성
<div align="center">
<img src="https://github.com/user-attachments/assets/4b01911c-d4d2-4a74-9cb2-fa0151154d8b">
</div>

  - 클러스터형 인덱스의 경우에는 그대로 변함이 없음
  - 💡 보조 인덱스의 경우, 보조 인덱스의 루트 페이지와 리프 페이지의 키 값(여기서는 name 열)이 이름으로 구성되어있으므로 일단 이름으로 정렬
  - 💡 특히, 보조 인덱스의 리프 페이지는 클러스터형 인덱스가 없었다면, 아마도 '데이터 페이지의 주소값'으로 구성되었을 것이지만, 클러스터형 인덱스의 키 값(여기서는 userID)을 가지게 됨
  - 💡 또, 보조 인덱스를 검색한 후, 모두 다시 클러스터형 인덱스의 루트 페이지부터 검색한다는 점

7. 예를 들어, '임재범'이란 사람의 주소를 알고 싶다면, 다음과 같은 쿼리 실행
  - 💡 인덱스는 단지 검색을 빨리 검색해주는 역할을 할 뿐, 결과의 내용과 관련 없음
```sql
SELECT addr FROM mixedtbl WHERE name = '임재범';
```

  - 💡 다음과 같은 순서로 검색
    + (페이지번호 10번 읽음) 보조 인덱스의 루트 페이지에서 '은지원'볻 큰 값이므로 200번 페이지에 있다는 것 확인
    + 💡 (페이지번호 200번 읽음) '임재범'은 클러스터형 인덱스의 키 값이 LJB임을 확인한 후, 무조건 클러스터 인덱스의 루트 페이지로 가서 찾음
    + (페이지번호 20번 읽음) 'LJB'는 'KBS'보다 크고, 'SSK'보다 작으므로 1001번 페이지에 있는 것 확인
    + (페이지번호 1001번 읽음) 'LJB' 값을 찾고, 그 주소인 '서울을 알아냄'

8. 왜 이렇게 구성한 이유?
   - 보조 인덱스의 리프 페이지에 기존처럼 '데이터페이지의 번호 + #오프셋'으로 하면 더 검색이 빠르고, 효율적일 것
   - 클러스터형 인덱스와 보조 인덱스를 분리해서 서로 관련 없이 구성하면 검색에서는 더 우수한 성능을 보일 것
   - 만약, 'MMI 멍멍이 서울'이라는 행이 추가되면, 클러스터형 인덱스는 페이지 분할 등의 작업 발생 (기존 방식과 동일)
   - 보조 인덱스에서도 100번 페이지만 '멍멍이 MMI'이 추가되면서 약간의 데이터가 변경될 뿐 그렇게 큰 변화가 발생하지 않음
   - 💡 하지만, 만약, 보조 인덱스의 리프 페이지가 '데이터 페이지의 주소값'으로 되어있다면?
     + 💡 데이터의 삽입으로 클러스터형 인덱스의 리프 페이지(=데이터 페이지)가 재구성
     + 💡 따라서, '데이터 페이지의 번호' 및 '#오프셋'이 대폭 변경 되어 보조 인덱스 역시 많이 부분이 재구성되어야 하므로, 엄청난 시스템 부하를 발생시킬 소지 존재
   - 따라서, 보조 인덱스와 클러스터형 인덱스가 하나의 테이블에 존재하면 위와 같이 구성
   - 💡 그렇게 되면, 이름으로 검색 시 보조 인덱스를 검색한 후 다시 클러스터형 인덱스를 검색해야 하므로 약간의 손해를 볼 수 있지만, 데이터 삽입 때문에 보조 인덱스를 대폭 재구성하게 되는 큰 부하는 걸리지 않음

9. 고려사항
    - 클러스터형 인덱스의 키(KKH, KBS 등)를 보조 인덱스가 저장하는 것 확인 가능
    - 그러므로 클러스터형 인덱스를 지정한 열(여기서는 userID)의 자릿수가 크다면 보조 인덱스에 저장되어야 할 양도 더불어 많아짐
    - 그러면, 차지하는 공간이 커질 수 밖에 없음
    - 💡 결국, 보조 인덱스와 혼합되어 사용되는 경우에는 되도록이면 클러스터형 인덱스로 설정한 열은 적은 자릿수의 열을 선택하는 것이 바람직

10. 💡 인덱스를 검색하기 위한 일차 조건은 WHERE 절에 해당 인덱스를 생성한 열의 이름이 나와야 함
    - 💡 물론, WHERE 절에 해당 인덱스를 생성한 열 이름이 나와도 인덱스를 사용하지 않는 경우 많음
