-----
### Database
----
1. 여러 사람이 공유하고 운영할 목적으로 관리되는 통합적 정보의 집합
2. 데이터베이스 관리시스템 (DBMS, DataBase Management System)에 의해 관리
3. 논리적으로 연관된 하나 이상의 자료를 모음으로 그 내용을 고도로 구조화하여 검색과 갱신을 효율적으로 하는 것
4. 몇 개의 자료를 조직적으로 통합하여 자료 항목의 중복을 없애고, 자료를 구조화하여 기억시켜 놓은 자료의 집합체
5. 공동 자료로서 각 사용자는 같은 데이터라 할지라도 각자의 응용 목적에 따라 다르게 사용 가능
6. 특징
   - 실시간 접근성
   - 지속적인 변화
   - 동시 공유
   - 내용에 의한 참조
   - 데이터 논리적 독립성 = 데이터에 종속적이지 않음
7. 장점
   - 데이터 중복 최소화
   - 데이터 공유
   - 일관성, 무결성, 보안성 유지
   - 최신 데이터 유지
   - 데이터 표준화 가능
   - 데이터의 논리적, 물리적 독립성
   - 용이한 데이터 접근
   - 데이터 공간 절약
   
-----
### RDBMS (Relational DataBase Management System)
----
1. 관계형 데이터 모델을 기반으로 한 데이터베이스
2. 모든 데이터를 테이블 단위로 저장
3. 관계 (Relationship) : 두 테이블 간 서로 연결되는 방식 / 테이블의 종속성 표현

-----
### DataBase Management System(DBMS)
----
1. 최종 사용자 또는 응용 프로그램이 데이터베이스에 접근할 수 있도록 도와주는 인터페이스 역할
2. 저장된 데이터를 보다 체계적으로 관리하고 이용할 수 있도록 해주는 소프트웨어
3. 이름, 값의 관계로 관리하는 데이터베이스 (TABLE의 집합)

         root 계정 : 기본적으로 제공되는 관리 계정
         Database 조회 : show databases;
         사용할 Database 지정 : use Database_name;
         Table 구조 조회 : desc[ribe] table_name;
<div align = "center">
<img src="https://github.com/sooyounghan/Web/assets/34672301/d44f70a5-aec5-47cb-a35e-0c8b98e6e5bb">
</div>   

      <기본 Schema>
      - infomation_schema : mysql 서버의 테이블, 열, 제약조건, 권한 등 정보를 담고 있는 테이블들의 집합
      - mysql : 사용자, 권한 등에 대한 정보를 가진 테이블들이 존재
      - performance_schema : mysql 서버의 성능가 관련된 정보를 가진 테이블들이 존재
      - sys : MYSQL 5.7 부터 추가 (information_schema와 performance_schema를 조합)
      
-----
### Structed Data
----
1. 미리 정해진 구조에 따라 저장된 데이터
2. 정해진 형식과 구조를 바탕으로 함
3. 손쉽게 데이터 검색 및 삽입 / 삭제 / 수정 / 삭제 연산 수행 가능

-----
### Unstructed Data
----
1. 정해진 구조나 규칙이 없고, 연산에 사용할 수 없는 형태의 데이터
2. 소셜 데이터 및 워드나 PDF 문서, 이미지, 영상 등

-----
### Semi-Structed Data
----
1. 정형 데이터와는 달리 구조화되어있지 않아 연산 불가능
2. 데이터 내부에 데이터 구조에 대한 메타 정보를 함께 저장하는 파일 형식 데이터
3. HTML, XML, JSON 파일이나 웹 로그, 센서 데이터 등을 의미

<div align = "center">
<img src="https://github.com/sooyounghan/Web/assets/34672301/abb38eae-ac68-4e1b-9ea8-7acaed72282e">
</div>   

<SQL = RDBMS>

-----
### Table = Relationship
----
1. 동일 주제의 데이터 집합
2. 1개 이상의 열과 0개 이상의 행으로 이루어진 데이터 집합
3. 데이터베이스 내 유일한 이름을 가져야 함

-----
### Column = Attribute, Field
----
1. 의미가 더 이상 분리되지 않는 최소 데이터 단위
2. 한 테이블 내 유일한 열 이름을 가져야함
3. 문자, 숫자, 날짜 등 자신만의 데이터 타입 가짐

-----
### Row = Tuple, Record
----
1. 관련 있는 열의 묶음
2. 한 테이블에 저장된 모든 행은 동일한 수의 열을 가짐

-----
### Primary Key (PK)
----
1. 주된 식별자
2. 한 테이블 내 행을 구별할 수 있는 열 또는 열의 묶음

-----
### SQL (Structed Query Language)
----
1. 데이터에 접근하기 위한 가장 보편적 수단
2. 역할 : RDBMS의 데이터를 관리하고, 다양한 데이터 동작을 수행하는데 사용되는 표준화된 프로그래밍 언어
3. 데이터베이스 및 테이블 생성, 검색 및 추가, 수정, 삭제 및 트랜잭션 처리 / 보안 및 권한 제어 등 모든 측면 관리

-----
### DDL (Data Definition Language)
----
1. 데이터베이스의 구조를 정의할 때 사용하는 언어
2. 데이터베이스나 테이블, 인덱스와 같은 데이터베이스 객체를 생성, 변경, 삭제할 때 사용
<div align = "center">
<img src="https://github.com/sooyounghan/Web/assets/34672301/bcc38aa6-0a91-4ccf-bd0f-c33be0d4abe9">
</div>   

-----
### DML (Data Manipulation Language)
----
1. 데이터베이스를 관리하는 데 사용하는 언어
2. 레코드 추가 및 삭제, 데이터를 변경할 때 사용
3. SELECT문 : 데이터 검색에 사용되나 DML 또는 질의어(SQL)로 분리되기도 함

<div align = "center">
<img src="https://github.com/sooyounghan/Web/assets/34672301/6b69e0cc-c00c-4a08-af0d-40b41542f6e6">
</div>   

-----
### DCL (Data Control Language)
----
1. 데이터에 대한 액세스를 제어하기 위한 언어
2. 데이터베이스 객체나 데이터에 대한 권한 관리에 사용

-----
### TCL (Transaction Contorl Language)
----
: INSERT, UPDATE, DELETE 문에 의해 수행된 변경 사항을 관리하는 데 사용 (ROLLBACK, COMMIT)

-----
### Data Dictionary = System Catalog
-----
1. 사용 가능한 데이터베이스 및 테이블의 정보를 가지고 있는 시스템 테이블
2. DBMS만이 추가, 수정, 삭제만 가능
3. 사용자는 조회만 가능

-----
### 주석문 : --, /* */ 
-----

-----
### 현재 접속한 데이터베이스 내의 Table 조회
-----
: SELECT * FROM tab;

-----
### 원하는 테이블의 구조를 조회
-----
: desc[ribe] table_name;

-----
### DataBase 생성
-----
- CREATE DATABASE 데이터베이스명 default character set UTF8;
