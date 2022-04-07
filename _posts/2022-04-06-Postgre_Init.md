---
title: PostgreSQL Docker DB 초기화
author: Jandari
date: 2022-04-06 22:00:00 -0500
categories: [Database, PostgreSQL]
tags: [Docker, PostgreSQL]
math: true
mermaid: true
---

## 소개

요즘은 간단한 프로젝트나 테스트를 하기 위해 Docker를 사용하여 Database를 띄우는 경우가 많습니다.

그럴때 마다 백업 된 데이터를 다시 넣어주거나 데이터를 일일이 다시 넣어주는 일은 매우 귀찮은 일입니다.

Database를 가장 맨 처음 딱 한번만 데이터를 넣고 싶을 때 사용하는 방법에 대해 알아보도록 하겠습니다.

#### Init Data 준비

```sql

CREATE TABLE USERS(
    index SERIAL PRIMARY KEY,
    id VARCHAR NOT NULL,
    name VARCHAR NOT NULL,
    email VARCHAR NOT NULL 
);

INSERT INTO USERS(index, id, name, email)
VALUES(DEFAULT, 'hgsp', '박혁거세', 'ysp@google.com');
INSERT INTO USERS(index, id, name, email)
VALUES(DEFAULT, 'ssl', '이순신', 'shl@google.com');
INSERT INTO USERS(index, id, name, email)
VALUES(DEFAULT, 'skl', '이성계', 'gwl@google.com');
INSERT INTO USERS(index, id, name, email)
VALUES(DEFAULT, 'jj', '조조', 'bhj@google.com');
INSERT INTO USERS(index, id, name, email)
VALUES(DEFAULT, 'ska', '안성기', 'sya@google.com');
INSERT INTO USERS(index, id, name, email)
VALUES(DEFAULT, 'jjk', '김좌진', 'syk@google.com');
```

위 데이터는 `USERS`라는 테이블을 만들고 사람들의 id, name, email을 넣어주는 sql 문입니다.


#### docker-compose 파일 준비

```yml
services:
  postgres:
    image: postgres:13
    container_name: postgres
    environment:
      - POSTGRES_PASSWORD=passwd
    ports:
      - '5432:5432'
    volumes:
      - ./init_data/createdb.sql:/docker-entrypoint-initdb.d/createdb.sql
  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4
    ports:
      - 8088:80
    environment:
      - PGADMIN_DEFAULT_EMAIL=my@mail.com
      - PGADMIN_DEFAULT_PASSWORD=passwd
```

`POSTGRES_PASSWOR`는 postgresql에서 사용 될 암호입니다.

`volumes`에 보시면 컨테이너의 `docker-entrypoint-inidb.d`라는 곳에 우리가 작성한 sql파일을 넣게 되는데 postgresql은 생성 되는 최초 한번만 저기의 파일을 읽어 사용하게 됩니다.

`pgadmin`의 경우 web으로 된 postgresql의 editor 입니다.

## 실행

`docker-compose` 파일이 있는 곳에서 아래와 같이 명령어를 입력하여줍니다.

```sql
$ docker-compose up

postgres  | The files belonging to this database system will be owned by user "postgres".
postgres  | This user must also own the server process.
postgres  |
postgres  | The database cluster will be initialized with locale "en_US.utf8".
postgres  | The default database encoding has accordingly been set to "UTF8".
postgres  | The default text search configuration will be set to "english".
postgres  |
postgres  | Data page checksums are disabled.
postgres  |
postgres  | fixing permissions on existing directory /var/lib/postgresql/data ... ok
postgres  | creating subdirectories ... ok
postgres  | selecting dynamic shared memory implementation ... posix
postgres  | selecting default max_connections ... 100
postgres  | selecting default shared_buffers ... 128MB
postgres  | selecting default time zone ... Etc/UTC
postgres  | creating configuration files ... ok
postgres  | running bootstrap script ... ok
postgres  | performing post-bootstrap initialization ... ok
postgres  | syncing data to disk ... initdb: warning: enabling "trust" authentication for local connections
postgres  | You can change this by editing pg_hba.conf or using the option -A, or
postgres  | --auth-local and --auth-host, the next time you run initdb

........
pgadmin   | NOTE: Configuring authentication for SERVER mode.
pgadmin   |
pgadmin   | [2022-04-06 12:44:52 +0000] [1] [INFO] Starting gunicorn 20.1.0
pgadmin   | [2022-04-06 12:44:52 +0000] [1] [INFO] Listening at: http://[::]:80 (1)
pgadmin   | [2022-04-06 12:44:52 +0000] [1] [INFO] Using worker: gthread
pgadmin   | [2022-04-06 12:44:52 +0000] [88] [INFO] Booting worker with pid: 88
```

이미지가 없다면 자동으로 다운받게 되고 컨테이너가 실행되게 됩니다.

위와 같이 `postgres`와 `pgadmin`이 정상적으로 실행 된것을 볼 수 있습니다.

#### Postgre 콘솔 접속

```
$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                           NAMES
4ef0ee78dad7   dpage/pgadmin4   "/entrypoint.sh"         2 minutes ago   Up 2 minutes   443/tcp, 0.0.0.0:8088->80/tcp   pgadmin
c96f8fa62108   postgres:13      "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:5432->5432/tcp          postgres
```

저는 postgre의 `CONTAINER ID`가 `c96f8fa62108`이기 때문에 아래와 같이 명령어를 입력하여 접속하도록 하겠습니다.

```
$  docker exec -it c96 psql -U postgres
postgres=# SELECT * FROM USERS;

 index |  id  |   name   |     email
-------+------+----------+----------------
     1 | hgsp | 박혁거세 | ysp@google.com
     2 | ssl  | 이순신   | shl@google.com
     3 | skl  | 이성계   | gwl@google.com
     4 | jj   | 조조     | bhj@google.com
     5 | ska  | 안성기   | sya@google.com
     6 | jjk  | 김좌진   | syk@google.com
```

#### pgAdmin 접속

```
$ docker ps

CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                           NAMES
4ef0ee78dad7   dpage/pgadmin4   "/entrypoint.sh"         7 minutes ago   Up 7 minutes   443/tcp, 0.0.0.0:8088->80/tcp   pgadmin
c96f8fa62108   postgres:13      "docker-entrypoint.s…"   7 minutes ago   Up 7 minutes   0.0.0.0:5432->5432/tcp          postgres
```

`pgadmin`의 포트가 `8088`이 포워딩 되어 있는 것을 볼 수 있습니다.

[http://localhost:8088](http://localhost:8088)

위 주소로 접속하게 되면 아래와 같은 `Login`화면을 볼 수 있습니다.

`docker-compose`에서 설정한 email과 password로 접속하면 됩니다.

![image](/assets/img/post/2022-04-06-Postgre_Init/1.jpg)

`Add New Server`를 통해 `postgre`의 서버를 설정하도록 하겠습니다.

![image](/assets/img/post/2022-04-06-Postgre_Init/2.jpg)

![image](/assets/img/post/2022-04-06-Postgre_Init/3.jpg)

`Connection`에서 `Host name/address` 부분은 `docker-compose`에 적혀있는 서비스 명을 적으면 됩니다.

`docker-compose`를 사용하여 띄웠기 때문에 Docker 내부의 서비스명으로 도메인이 자동으로 생성됩니다.

![image](/assets/img/post/2022-04-06-Postgre_Init/4.jpg)


