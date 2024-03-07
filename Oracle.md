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
