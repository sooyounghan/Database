-----
### Oracle 명령어
-----
1. SQLPLUS :  SQL문을 실행시키고 그 결과를 볼수 있도록 오라클에서 제공하는 툴
  - SQL PLUS 명령어
      
        + 1. SET LINSIZE 크기 (너비 조정)
        + 2. SET PAGESIZE 크기 (페이지 조정)
	        예) SET LINESIZE 150 / SET PAGESIZE 500

2. USER-NAME : /as sysdba (데이터베이스 관리자로 연결)
3. CONN [/AS SYSDBA] : 데이터베이스 관리자로 접근
4. CONN [SYSTEM] : [시스템] 유저로 접근
5. SHOW USER(= SELECT * FROM ALL_USERS/DBA_USERS) : 사용 유저 보기 
6. DBA_USERS : 모든 사용자 계정의 정보가 담긴 테이블
------
7. hr계정 : 오라클이 교육용으로 만든 계정으로 기본적으로 잠겨있으므로, 계정 잠금 해제 필요
  - 계정해제 : alter user hr acoount unlock;
  - 계정비밀번호 지정 : alter user hr idetified by 비밀번호;
  - conn hr/hr(비밀번호) : hr 계정에 접근
-----
8. SPOOL
  - 시작 : SPOOL 경로/생성할 파일명
  - 종료 : SPOOL OFF (해당 지정파일에 저장)
  - BUFFER 내 / 의미 : 쿼리문의 ;와 동일하며, 버퍼에서는 오직 1개의 쿼리문만 실행가능
  - 버퍼 창 불러오기 : ed
  - 저장 후 실행 : 커멘드 창 : /

-----
### 테이블 스페이스 (Table Space)
-----
1. 데이터 저장 단위는 물리적, 논리적 단위로 나눌 수 있음
2. 물리적 단위는 파일이며, 논리적 단위는 데이터 블록 → 익스텐트 → 세그먼트 → 데이터 스페이스
3. 데이터 블록이 여러 개 모여 익스텐트를, 여러 개의 익스텐트가 모여 세그먼트를 구성
4. 테이블 스페이스는 테이블들을 담을 수 있는 커다란 공간이며, 실제 물리적으로 파일 존재
5. 테이블 스페이스 생성 방법
   - SQLPLUS 실행(사용자는 system와 비밀번호)
   - 테이블 스페이스를 myts라는 이름으로 100MB 크기로 생성할 것이며, 데이터 늘어나 꽉 찰 것을 대비해 5MB씩 자동 증가 옵션 추가

      		CREATE TABLESPACE myts DATAFILE '생성위치\파일명' SIZE 100M AUTOEXTEND ON NEXT 5M;

6. 기본 테이블스페이스 : 해당 사용자로 로그인한 뒤 테이블과 같은 각종 데이터베이스 객체가 저장되는 테이블 스페이스
7. 임시 테이블 스페이스 : 해당 사용자가 사용하는 디폴트 임시 테이블 스페이스

----
### 사용자 생성
----
1. 사용자 생성하기
   
           CREATE USER 유저명 IDENTIFIED BY 비밀번호
   	   DEFAULT TABLESPACE 테이블스페이스명
   	   TEMPORARY TABLESPACE 테이블스페이스명;

2, 권한 부여하기   

   - 사용자 생성을 완료하면, 해당 사용자에게 권한 부여
   - 데이터베이스 접속을 위해 CONNECT 라는 권한 부여해야 데이터베이스 접속 가능
   - GRANT DBA TO 유저명;

3. 사용자 계정 DB 접속 확인
   
   - CONNECT 유저명/암호;
   - SELECT USER FROM DUAL;
