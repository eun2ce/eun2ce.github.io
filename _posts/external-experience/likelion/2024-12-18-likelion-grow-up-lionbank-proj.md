---
title: "[ 멋쟁이사자처럼 백엔드 스쿨 ] docker-compose 로 MySQL 활용하기: 은행 사용자 정보 저장"
date: 2024-12-18 09:39:00 +0900
categories: [ "external-experience", "likelion" ]
tags: [ "bootcamp", "java", "멋쟁이사자처럼", "부트캠프", "프로그래밍" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/external-experience/likelion/2024-12-18-likelion-grow-up-lionbank-proj/2024-12-18-09-37-11.png
  alt: "docker-compose 로 MySQL 활용하기: 은행 사용자 정보 저장"
---

> 최근 프로젝트에서 `Docker-Compose`를 활용해 `MySQL`을 띄우고, 은행 사용자와 계좌 정보를 저장하는 시스템을 구축해보았습니다.
> 초기 설정부터 문제 해결 과정까지 경험한 내용을 공유합니다.
> 특히, 초기화 `SQL 스크립트(init.sql)`가 제대로 실행되지 않았던 문제와 그 해결 과정을 중점적으로 다룹니다.
> {: .prompt-info }

## 개요

이 프로젝트는 은행 사용자 정보를 데이터베이스에 저장하고 관리하는 것을 목표로 합니다. 주요 요구사항은 다음과 같습니다.

1. `Docker-Compose`로 `MySQL` 환경 구성: 손쉬운 배포와 테스트를 위해 컨테이너화된 데이터베이스 환경 구축.
2. 사용자 및 계좌 테이블 설계: 간단한 스키마로 사용자와 계좌 데이터를 저장.
3. 초기 데이터 삽입: init.sql 스크립트를 통해 초기 테이블과 샘플 데이터를 자동 생성.

## 환경설정

### `docker-compose` 파일

`docker-compose.yaml`파일을 작성하여 `MySQL` 컨테이너를 정의합니다.

```yaml
services:
  db:
    image: mysql:8.3.0
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: "root"
      MYSQL_DATABASE: "lionbank_db"
      MYSQL_USER: "lionbank"
      MYSQL_PASSWORD: "lionbank"
    volumes:
      - ./database/datadir:/var/lib/mysql
      - ./database/init/:/docker-entrypoint-initdb.d/
    ports:
      - 3306:3306
    command: >
      --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci
```

### 초기화 스크립트 (`init.sql`)

```sql
USE lionbank_db;

-- 고객 테이블 생성
CREATE TABLE IF NOT EXISTS customers (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

-- 계좌 테이블 생성
CREATE TABLE IF NOT EXISTS accounts (
    account_id VARCHAR(50) PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL,
    balance DECIMAL(15, 2) DEFAULT 0.00,
    FOREIGN KEY (customer_id) REFERENCES customers(id) ON DELETE CASCADE
);

-- 샘플 데이터 삽입
INSERT INTO customers (id, name) VALUES ('cust_1', 'Alice');
INSERT INTO accounts (account_id, customer_id, balance) VALUES ('acc_1', 'cust_1', 1000.00);
```

## 초기화 스크립트가 실행되지 않는 문제 해결

MySQL 컨테이너는 정상적으로 실행되었지만, 테이블이 생성되지 않는 문제가 발생했습니다.   
로그를 확인해 보니 초기화 스크립트가 실행되지 않았음을 알 수 있었습니다.

### 원인 분석

* `docker-entrypoint-initdb.d` 디렉토리에 파일이 잘못된 위치에 있거나 파일명이 정확하지 않을 가능성.
* 데이터베이스가 이미 초기화된 상태에서 스크립트가 무시되는 현상.

### 해결

#### 컨테이너 로그 파일 확인

```bash
2024-12-16 16:28:57+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.3.0-1.el8 started.
2024-12-16 16:28:57+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2024-12-16 16:28:57+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.3.0-1.el8 started.
'/var/lib/mysql/mysql.sock' -> '/var/run/mysqld/mysqld.sock'
2024-12-16T16:28:58.061140Z 0 [System] [MY-015015] [Server] MySQL Server - start.
2024-12-16T16:28:58.248922Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.3.0) starting as process 1
2024-12-16T16:28:58.253094Z 0 [Warning] [MY-010159] [Server] Setting lower_case_table_names=2 because file system for /var/lib/mysql/ is case insensitive
2024-12-16T16:28:58.261965Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-12-16T16:28:58.483499Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-12-16T16:28:58.716122Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2024-12-16T16:28:58.716420Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2024-12-16T16:28:58.722427Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2024-12-16T16:28:58.750494Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
2024-12-16T16:28:58.750682Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.3.0'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
```

위에 보이는 것 처럼 `init.sql`이 실행 된 내역이 남지 않았습니다.

#### docker-entrypoint-initdb.d 파일 확인

```bash
$ docker ps                      
CONTAINER ID   IMAGE         COMMAND                  CREATED        STATUS         PORTS                               NAMES
e200fed51a81   lionbank-db   "docker-entrypoint.s…"   24 hours ago   Up 7 minutes   0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
$ docker exec -it mysql /bin/bash                               
bash-4.4# ls /docker-entrypoint-initdb.d/
init.sql/
```

`init.sql` 은 파일인데 디렉토리 경로로 잡혀있어 실행되지 않았다는 것을 알게 되었습니다.

#### `docker-compose.yaml` 수정

```yaml
    volumes:
      - ./database/init/init.sql:/docker-entrypoint-initdb.d/init.sql # 파일을 지정하여 컨테이너 내부로 마운트 한다.
```

#### 컨테이너 재 시작

컨테이너 재시작을 그냥 하게 되면 `database` 디렉토리 안에있는 `MySQL`이 생성한 파일 때문에, 컨테이너가 깨끗하게 작동되지 않을 수 있습니다.
따라서 아래와 같은 순서로 진행합니다.

* 컨테이너 중지

```bash
$ docker-compose down
[+] Running 2/2
 ✔ Container mysql           Removed                                       2.7s
 ✔ Network lionbank_default  Removed
 ```

* 디렉토리 초기화

> `init.sql` 파일을 삭제하지 않도록 주의
> {: .prompt-danger }

```bash
$ tree database/.
database/.
├── datadir
└── init
    └── init.sql
```

* 컨테이너 재시작

```bash
docker-compose up -d
[+] Running 2/2
 ✔ Network lionbank_default  Created                                                                          0.1s
 ✔ Container mysql           Started
```

#### 확인

* `init.sql` 생성 확인

```bash
$ docker ps                      
CONTAINER ID   IMAGE         COMMAND                  CREATED        STATUS         PORTS                               NAMES
e200fed51a81   lionbank-db   "docker-entrypoint.s…"   24 hours ago   Up 43 minutes   0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
$ docker exec -it mysql /bin/bash                               
bash-4.4# ls /docker-entrypoint-initdb.d/
init.sql // 생성 되어 있음
```

* 컨테이너 로그 확인

```bash
$ docker logs mysql
```

![docker_container_logs](/assets/img/posts/external-experience/likelion/2024-12-18-likelion-grow-up-lionbank-proj/2024-12-18-09-36-12.png)

정상 실행 된 것을 볼 수 있습니다.

* 테이블 조회

```bash
$ docker exec -it mysql mysql -u lionbank -plionbank lionbank_db

mysql: [Warning] Using a password on the command line interface can be insecure.
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.3.0 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show tables;
```

![docker_container_logs](/assets/img/posts/external-experience/likelion/2024-12-18-likelion-grow-up-lionbank-proj/2024-12-18-09-37-11.png)

> git repo [https://github.com/eun2ce/likelion/tree/main/lionbank](https://github.com/eun2ce/likelion/tree/main/lionbank)
> {: .prompt-info }
