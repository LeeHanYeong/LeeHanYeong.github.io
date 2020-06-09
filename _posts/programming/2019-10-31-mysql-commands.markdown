---
layout: post
title:  "ln 옵션"
categories: ['기타']
---

## MySQL 정리

### 명령어들

```sql
## User
CREATE USER 'username'@'localhost' identified by 'password';
DROP USER 'username'@'localhost';

GRANT ALL PRIVILEGES ON *.* to 'username'@'localhost';
GRANT ALL PRIVILEGES ON dbname.* to 'username'@'localhost';


## DB
SHOW DATABASES;
CREATE DATABASE dbname;
USE dbname;
DROP DATABASE [IF EXISTS] dbname;

## 테이블
SHOW TABLES;
EXPLAIN tablename;
DESCRIBE tablename;
RENAME TABLE tablename1 TO tablename2
DROP TABLE [IF EXISTS] tablename;

# Rows
INSERT INTO tablename VALUES (val1, val2...);
INSERT INTO tablename (col1, col2...) VALUES (val1, val2...);
SELECT col1, col2... FROM tablename;
UPDATE tablename SET col1=val WHERE condition;
```





### 그 외 사용법

**특정 버전의 MySQL설치**

```shell
brew install mysql@5.7
```



**MySQL Root password 설정**  
brew로 MySQL을 설치 한 경우, Root 비밀번호가 설정되지 않은 상태이므로 아래 명령어를 실행

```
mysql_secure_installation
```



**MySQL Validate Password Plugin 삭제**  

```sql
> mysql -uroot
mysql> uninstall plugin validate_password;
Query OK, 0 rows affected (0.05 sec)
```



**모든 로그 확인**

```sql
mysql> show variables like "general_log%";
+------------------+--------------------------------+
| Variable_name    | Value                          |
+------------------+--------------------------------+
| general_log      | OFF                            |
| general_log_file | /usr/local/var/mysql/lhy-2.log |
+------------------+--------------------------------+

mysql> SET GLOBAL general_log = 'ON';
```

```shell
tail -f /usr/local/var/mysql/lhy-2.log
```

로그 확인이 끝난 후 `SET GLOBAL general_log = 'OFF';` 설정