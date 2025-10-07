-----
### 윈도우 - MySQL 다운로드와 설치
-----
1. 윈도우를 사용하면 MySQL Installer 설치 프로그램 하나만 다운로드
   - MySQL 커뮤니티 서버 다운로드 방법 : ```https://dev.mysql.com/downloads/```
   - 다운로드 사이트 이미지
<div align="center">
<img src="https://github.com/user-attachments/assets/67a7eca4-8bf4-40e3-8dcc-1acb93ab4ab9">
</div>

  - MySQL 다운로드 사이트에서 MySQL Installer for Windows를 선택
  - 또는 다음 링크로 직접 이동 : ```https://dev.mysql.com/downloads/installer/```

2. MySQL Installer for Windows 다운로드 사이트
<div align="center">
<img src="https://github.com/user-attachments/assets/dd7be216-101f-457b-abd9-6238c1a3dd1e">
</div>

  - Select Version : 8.0.x (8.0.42 ~ 8.0.99)
  - Select Operating System : Microsoft Windows
  - 중요 : 버전은 꼭 8.0.x 버전을 사용하자.
    + 더 최신 버전들이 있지만, 이 버전을 사용하는 이유는 함께 설치할 MySQL 워크벤치가 아직 8.0.x(8.0.42 ~ 8.0.99) 버전에 머물러 있기 때문임
    + 따라서 더 최신의 버전인 8.4.x, 9.x 버전은 MySQL 워크벤치와 호환성에 문제가 발생할 수 있기 때문에 사용하지 않음
  - 항목을 선택하고 나면 스크린샷의 빨간 네모에 있는 Download 버튼을 선택
  - 둘 중에 용량이 큰 것을 선택하면 된다. (mysql-installer-community-x.msi)

<div align="center">
<img src="https://github.com/user-attachments/assets/1d9c6cb1-3d40-4031-9573-9f8fc9adc1e1">
</div>

   - 이 화면에서 스크린샷의 빨간 네모에 있는 문구를 선택 : 그러면 회원 가입 없이 다운로드가 가능

3. MySQL Installer for Windows 설치 방법
<div align="center">
<img src="https://github.com/user-attachments/assets/bf39087e-292f-48f3-8d8a-3a7a5837d5db">
</div>

  - mysql-installer-community-8.0.x.dmg 와 같은 파일이 다운로드 되었을 것 : 해당 파일을 실행
  - 이 앱이 디바이스를 변경할 수 있도록 허용하시겠어요? 설치 파일을 실행하면 이런 문구가 나올 수 있음
    + mysql-installer-community-8.0.x.msi가 화면에 나오는데 예 버튼을 선택
    + 추가로 MySQLInstallerLauncher.exe도 화면에 나올 수 있는데, 예 버튼을 선택해야 함

<div align="center">
<img src="https://github.com/user-attachments/assets/0df81821-96f8-4209-9e76-76cbf52ac8dc">
</div>

   - 화면과 같이 Full 버튼을 선택
   - Next 버튼을 선택

<div align="center">
<img src="https://github.com/user-attachments/assets/65d135ae-07c6-4e73-905c-45ec03b9d3df">
</div>

   - Execute 버튼을 선택

<div align="center">
<img src="https://github.com/user-attachments/assets/db5f94c4-02c3-409d-8aea-d87a1850df98">
</div>

   - 설치가 완료되면 Next 버튼을 선택

<div align="center">
<img src="https://github.com/user-attachments/assets/56c83158-506b-4f51-9da9-260ee9c7640b">
</div>

   - Next 버튼을 선택
     
<div align="center">
<img src="https://github.com/user-attachments/assets/a45edb15-871b-458d-9574-be12cc268edb">
</div>

   - 여기서도 별도의 설정을 선택하지 말고 그대로 Next 버튼을 선택
      + Config Type은 Development Computer가 선택되어 있어야 함 (기본으로 선택되어 있음)
      + 다른 값은 변경하지 말고 그대로 유지

<div align="center">
<img src="https://github.com/user-attachments/assets/cb491424-3a13-475e-a9dd-102d6a582169">
</div>

   - Use Strong Password Encryption for Authentication (RECOMMENDED)를 선택
   - Next 버튼을 선택

<div align="center">
<img src="https://github.com/user-attachments/assets/20ecb91a-af8c-4e1c-a113-f6ae2ff2c629">
</div>

  - 서버의 root 계정 비밀번호를 입력해야 함
  - 최소 8자는 입력해야 함 : 외우기 쉽게 test1234로 입력
  - Next 버튼을 선택
  - 실제 서비스를 운영한다면 당연히 어려운 비밀번호를 사용해야 함

<div align="center">
<img src="https://github.com/user-attachments/assets/0dcd0392-9bf2-4425-926c-0b387390abb2">
</div>

  - 설정을 변경하지 말고 Next 버튼을 선택
  - "Start the MySQL Server at System Startup"이 체크되어 있으면 컴퓨터가 재부팅 되어도 MySQL 서버가 자동으로 실행 : 이 버튼은 체크 상태로 유지 (기본으로 체크)

<div align="center">
<img src="https://github.com/user-attachments/assets/edcb5fad-7147-47cd-97c9-9320735f18d4">
</div>

   - Next 버튼을 선택
     
<div align="center">
<img src="https://github.com/user-attachments/assets/8cf17f98-c133-4383-8746-85d853996de9">
</div>

  - Execute 버튼을 선택
    
<div align="center">
<img src="https://github.com/user-attachments/assets/0a94c3af-fc79-49aa-b6f5-2cc2e9a3a6be">
</div>

  - Finish 버튼을 선택
    
<div align="center">
<img src="https://github.com/user-attachments/assets/4aaf49ee-4e6d-4637-ad9f-a2572b5c4e8a">
</div>

   - Next 버튼을 선택
     
<div align="center">
<img src="https://github.com/user-attachments/assets/b35937f6-2915-41ed-a6d6-c71115027363">
</div>

   - Finish 버튼을 선택 : 설정은 화면과 같이 처음 상태 그대로 유지

<div align="center">
<img src="https://github.com/user-attachments/assets/6e2f2b76-2d53-4394-95b5-f5e1e3f03c5a">
</div>

   - Next 버튼을 선택

<div align="center">
<img src="https://github.com/user-attachments/assets/5d039b84-5f49-417c-8991-c1e90b6ab47a">
</div>

   - Password : test1234 를 입력
   - Check 버튼을 선택

<div align="center">
<img src="https://github.com/user-attachments/assets/7d630360-72fb-4076-92fc-f3d6fb6e9cec">
</div>

  - 초록색 체크가 나옴
  - Next 버튼을 선택

<div align="center">
<img src="https://github.com/user-attachments/assets/73a87bdb-abae-4645-92e9-4cd93dea2c57">
</div>

  - Execute 버튼을 선택
    
<div align="center">
<img src="https://github.com/user-attachments/assets/acc4ae25-4e17-438a-a7d3-c1ebd8066193">
</div>

   - Finish 버튼을 선택

<div align="center">
<img src="https://github.com/user-attachments/assets/ef028ffe-ccc1-46e3-be7d-c16495ddee99">
</div>

   - Next 버튼을 선택

<div align="center">
<img src="https://github.com/user-attachments/assets/690fe08e-b7aa-48be-a2cf-2d4e36baca9d">
</div>

  - Finish 버튼을 선택
  - 설치가 완료되면서 다음 두 프로그램이 실행 : 우선 둘다 종료

<div align="center">
<img src="https://github.com/user-attachments/assets/a0f20918-a083-4926-9c3f-65260b40beea">
</div>

  - MySQL 워크벤치 프로그램 : 우선 종료

<div align="center">
<img src="https://github.com/user-attachments/assets/8931c949-9ac6-470a-b992-66570df34a54">
</div>

  - MySQL 쉘 프로그램 : 우선 종료

4. 윈도우 - 2. MySQL 워크벤치 실행
   - MySQL 워크벤치 실행 방법 : MySQL Installer를 통해 MySQL 서버와 MySQL 워크벤치가 한 번에 모두 설치
      + MySQL 워크벤치를 통해서 앞서 설치한 MySQL 서버에 접근할 수 있음
      + MySQL 워크벤치를 설치하면 다음 아이콘을 확인할 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/822468a7-3d9b-4813-a4a7-7348e52cd958">
</div>

   - 윈도우의 검색창에서 MySQL Workbench라고 검색하면 이 프로그램이 보일 것
   - 이 프로그램을 실행 : MySQL 워크벤치를 실행하면 다음 화면이 나옴

<div align="center">
<img src="https://github.com/user-attachments/assets/2d49baed-ad06-4ede-a52f-aacabcb9e072">
</div>

   - MySQL 워크벤치의 실행 화면은 스크린샷과 다를 수 있음
   - 찾아보면 MySQL Connections 항목이 보일 것
   - 해당 항목에 있는 Local instance 3306 이라는 부분을 찾아서 마우스로 클릭

<div align="center">
<img src="https://github.com/user-attachments/assets/d3589690-c543-4ab9-a9f1-30004ec32222">
</div>

   - Password는 앞서 설정한 test1234 입력
   - Save password in vault 앞에 체크표시 : 그러면 이후에 접근할 때는 비밀번호를 입력하지 않아도 됨
   - OK 버튼을 선택
   - 다음 화명니 나오면 성공적

<div align="center">
<img src="https://github.com/user-attachments/assets/52e14cfa-40d4-4b61-bdce-d4e15645e886">
</div>

   - 이제 MySQL 워크벤치를 통해서 앞서 설치한 MySQL 서버에 접근할 수 있음
   - 실습을 위한 모든 설치가 끝남
   - 만약 다음 화면이 나온다면 MySQL 데이터베이스 서버가 꺼져있는 것

<div align="center">
<img src="https://github.com/user-attachments/assets/13836877-4745-49e1-b1e8-4d2edea8a811">
</div>

   - 왼쪽 하단에 No connection established라는 빨간색 경고 메시지가 보임
   - 하단의 Action Output을 보면 Could not connect. server may not be running. 메시지가 보임
   - MySQL 서버에 문제가 있어서 MySQL 워크벤치가 접근할 수 없는 것
   - 이런 경우 다음 MySQL 서버 실행 확인 방법을 참고해서 MySQL 서버를 먼저 실행한 다음에 MySQL 워크벤치를 실행

5. 윈도우 - 3. MySQL 서버 실행 확인
   - MySQL 서버 실행 확인 방법 : 윈도우의 서비스를 실행
   - 윈도우 검색창에서 서비스를 검색
   - 서비스 실행 화면
<div align="center">
<img src="https://github.com/user-attachments/assets/29b0afa9-85f7-4866-b95a-84c7f9553476">
</div>

  - 오른쪽 항목에서 MySQL80 을 찾아서 더블 클릭

<div align="center">
<img src="https://github.com/user-attachments/assets/8a0da117-dd8c-45ab-805f-3edd9db1fb70">
</div>

   - 시작 버튼을 선택하면 MySQL 서버가 실행되고, 중지 버튼을 선택하면 MySQL 서버가 중지
   - 현재는 실행 중 상태

<div align="center">
<img src="https://github.com/user-attachments/assets/07c3797f-592f-4244-bb3c-f61c8df1f7d9">
</div>

  - 시작 버튼을 선택하면 MySQL 서버가 실행되고, 중지 버튼을 선택하면 MySQL 서버가 중지
  - 현재는 중지됨 상태
  - 시작 버튼을 선택해서 MySQL 서버를 실행 중 상태로 변경
  - 실행 중 상태를 유지 : 실행 중 상태여야 MySQL 워크벤치도 정상 작동
