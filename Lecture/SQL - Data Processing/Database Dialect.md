-----
### 데이터베이스 방언
-----
1. SQL은 표준이 있지만, 각 데이터베이스 제조사(Oracle, Microsoft, MySQL 등)는 자신들만의 고유한 기능과 문법을 추가
   - 이를 '데이터베이스 방언(Dialect)'이라고 부름
   - 방언은 사투리로, 방언은 특정 데이터베이스에서만 동작하는 강력하고 편리한 기능을 제공하지만, 다른 데이터베이스와 호환되지 않는다는 단점이 존재
   - IFNULL()도 사실 MySQL(과 일부 DB)의 방언이고, SQL 표준에 더 가까운 것은 COALESCE()

2. MySQL 전용 상세 함수 : MySQL은 개발 편의성과 강력한 기능을 위해 SQL 표준 외에 다양한 함수들을 제공

-----
### 문자열 함수 (String Functions)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/d26584a6-dbf0-4e9b-948b-14636c2ca5df">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/f5ed7bff-2383-4dd3-9932-a0ac354b9a04">
</div>

-----
### 날짜 및 시간 함수 (Date and Time Functions)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/d2506225-caa5-4104-b518-5ea9e9f1888e">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/a4f98ea7-7ff9-4f17-94b0-4013bec97225">
</div>

-----
### 숫자 함수 (Numeric Functions)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/aa193cf9-a50b-43e5-91c2-051e107b5bb1">
</div>

-----
### 흐름 제어 함수 (Flow Control Functions)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/4150d348-0096-471b-b227-34f69f3bce67">
</div>

-----
### 윈도우 함수 (Window Functions) - MySQL 8.0+
-----
: OVER() 절과 함께 사용되어 행의 집합(window) 내에서 연산을 수행
<div align="center">
<img src="https://github.com/user-attachments/assets/fd114e06-34ce-4ef2-84e1-7411283cd0b5">
</div>

-----
### JSON 함수 - MySQL 5.7+
-----
: JSON 데이터를 생성, 파싱, 수정하는 강력한 함수
<div align="center">
<img src="https://github.com/user-attachments/assets/1705aa46-a26f-4459-b414-fec9fbaa5052">
</div>

-----
### 기타 함수
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/73600e2c-ec37-4d7c-acc6-bcba34e42d19">
</div>

-----
### MySQL 공식 문서 - 함수
-----
1. ```https://dev.mysql.com/doc/refman/8.0/en/functions.html```
2. 실제로는 더 많은 특수 목적의 함수들이 존재하며, 최신 버전의 MySQL이 출시될 때마다 새로운 함수들이 추가될 수 있음
3. 가장 정확하고 완전한 정보는 MySQL 공식 문서를 참고
