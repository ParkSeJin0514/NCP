# 📙 09.24 NCP
## 🔌 DB/Peering
### DB
```bash
netstat -antp # -a : 모든 연결과 리스닝 포트 표시
							# -n : 호스트명/서비스명 해석을 하지 않고 숫자(IP, 포트)로 표시
							# -t : TCP 연결만 표시
							# -p : 각 소켓을 소유한 프로세스 ID와 이름 표시
```

- mysql 연결

```sql
mysql -u root -h 'ip' -p          # MySQL 클라이언트 프로그램을 실행해서 원격 DB 서버에 접속하는 명령어

-u root                           # -u 옵션 : 사용할 DB 사용자 계정 지정 (여기서는 root 계정)
                                  # 주의 : MySQL은 '사용자' + '호스트' 조합으로 계정을 구분
                                  # 예) 'root'@'localhost' 와 'root'@'%' 는 서로 다른 계정

-h 'ip'                           # -h 옵션 : 접속할 MySQL 서버의 호스트(IP 또는 도메인) 지정
                                  # 예) -h 192.168.1.10 → 192.168.1.10 서버의 MySQL에 접속
                                  # 생략 시 기본은 localhost (Unix socket 접속)

-p                                # -p 옵션 : 패스워드를 입력하겠다는 의미
                                  # 실행하면 비밀번호 입력 프롬프트가 뜸
                                  # -pYourPass 형태로 바로 붙여 쓸 수도 있지만 보안상 권장하지 않음
```

- [localhos](http://localhost)t → mysql 연동

```sql
SELECT user, host FROM mysql.user;                              # MySQL 서버에 등록된 사용자(user)와 접속 가능한 호스트(host) 정보를 조회
                                                                # mysql.user 테이블은 MySQL의 계정 및 인증 정보를 저장하는 시스템 테이블

ALTER USER 'root'@'localhost' IDENTIFIED BY 'lab-password';     # root 계정의 비밀번호를 'lab-password'로 변경
                                                                # root'@'localhost' 는 root 계정이 오직 로컬에서만 접근 가능함을 의미
                                                                # 비밀번호 변경 시 보안상 반드시 강력한 패스워드를 설정하는 것이 권장됨

FLUSH PRIVILEGES;                                               # MySQL 권한 테이블(mysql.*)의 변경 사항을 즉시 반영
                                                                # 사용자 계정 변경, 권한 부여/회수 작업 후 적용하기 위해 실행
```

- 모든 호스트(%)에서 접속 허용
- Web Server → DB 연동

```sql
CREATE USER 'root'@'%' IDENTIFIED BY 'lab-password';                 # 계정 생성 : 사용자명 'root', 호스트 '%' (어느 호스트에서나 접속 허용)
                                                                     # 주의 : MySQL/MariaDB 는 '사용자' + '호스트' 조합별로 별도 계정으로 취급
                                                                     # 즉, 'root'@'localhost' 와 'root'@'%' 는 서로 다른 계정
                                                                     # 'IDENTIFIED BY' : 해당 계정의 비밀번호를 'lab-password' 로 설정
                                                                     # 이미 동일 조합 계정이 있으면 에러 → IF NOT EXISTS 를 쓰거나 ALTER USER 사용

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;         # 권한 부여 : 모든 DB와 모든 객체(*.*)에 대해 모든 권한 부여
                                                                     # WITH GRANT OPTION : 이 계정이 다른 사용자에게 권한을 부여/회수할 수 있게 허용
                                                                     # 사실상 전체 관리자 권한을 원격에서 허용하는 설정

FLUSH PRIVILEGES;                                                    # 권한 즉시 반영 : 메모리에 로드된 권한 정보를 즉시 다시 읽도록 지시
                                                                     # 참고 : 현대 MySQL/MariaDB 에서는 CREATE USER / GRANT 시 자동 반영되어 생략해도 됨
DROP USER 'root'@'%';                                                # 특정 유저 삭제                     
```

- 기존 [localhost](http://localhost) 계정의 호스트 정보 변경

```sql
UPDATE mysql.user SET Host='%' WHERE User='root' AND Host='localhost';   # mysql.user 테이블에서 root 계정의 Host 값을 수정
                                                                         # 원래 'root'@'localhost' 계정은 로컬(127.0.0.1, ::1)에서만 접속 가능
                                                                         # Host='%' 로 바꾸면 'root'@'%' 형태가 되어 "어느 IP에서든 root로 접속 가능"하게 됨
                                                                         # 즉, root 계정의 접근 제한이 사라져 원격 접속도 허용됨 (매우 위험한 설정)

FLUSH PRIVILEGES;                                                        # 권한 테이블(mysql.user)의 변경 사항을 즉시 반영
                                                                         # UPDATE, INSERT, DELETE 로 직접 수정했을 때 반드시 실행해야 함
```