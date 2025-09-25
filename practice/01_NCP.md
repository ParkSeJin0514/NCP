# 📙 09.24 NCP
## 🔨 Make Server
### Server

- local host

```bash
curl localhost
curl localhost -I # 헤더 확인
```

- Apache Install

```bash
yum clean all # 쓸모없는 패키지 삭제
yum install httpd -y # 설치
systemctl start httpd
systemctl enable httpd # 재부팅시에도 작동
systemctl status httpd # 상태 확인
```

- httpd Access log 확인

```bash
cat /var/log/httpd/access_log
```

- 블록 장치 리스트 출력

```bash
lsblk # 리눅스 시스템에 연결된 블록 장치(Block Device) 정보를 트리 형태로 보여줌
```

- 파일시스템 디스크 사용량 확인

```bash
df -hT # 파일시스템 단위의 디스크 사용량, 남은 공간, 마운트 지점 확인
```

- init script
```bash
#!/bin/bash
wget -O /etc/yum.repos.d/Rocky-Extras.repo http://init.ncloud.com/server/linux/repo/rocky8/Rocky-Extras.repo
yum clean all
yum -y install httpd php mariadb-server php-mysqlnd
cd /var/www/html
wget https://ncp-largeobjects.s3.ap-northeast-2.amazonaws.com/NCP-Essentials/lab-app-php7.zip
unzip lab-app-php7.zip -d /var/www/html/
chown apache:root /var/www/html/rds.conf.php
systemctl enable mariadb
systemctl enable httpd
systemctl enable php-fpm
systemctl start mariadb
systemctl start httpd
systemctl start php-fpm
```