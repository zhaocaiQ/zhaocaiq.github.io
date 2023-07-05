---
title: schema.sql error
layout: default
parent: Spring Boot
grand_parent: Error
nav_order: 2
---

{: .warning }
Failed to execute SQL script statement #5 of class path resource


spring boot 실행과 동시에 h2 데이터베이스에 테이블을 생성하는 코드를 작성했는데 위와 같은 오류가 발생했다.
처음에는 application.yml코드에 init을 설정하지 않아서 sql문을 읽지 못해서 테이블이 생성 안되었다 생각했다.   

```
//** application.yml */
server:
  port: 8083
spring:
  # H2 Setting Info (H2 Console에 접속하기 위한 설정정보 입력)
  h2:
    console:
      enabled: true  # H2 Console을 사용할지 여부 (H2 Console은 H2 Database를 UI로 제공해주는 기능)
      path: /h2-console  # H2 Console의 Path
  # Database Setting Info (Database를 H2로 사용하기 위해 H2연결 정보 입력)
  datasource:
    driver-class-name: org.h2.Driver  # Database를 H2로 사용하겠다.
    url: jdbc:h2:./test  # H2 접속 정보
    username: sa  # H2 접속 시 입력할 username 정보 (원하는 것으로 입력)
    password:  # H2 접속 시 입력할 password 정보 (원하는 것으로 입력)
  sql:
    init:
      # 생성 설정 always : 서버 재기동 마다 schema, data 새로   생성
      mode: always
      schema-locations: classpath:schema.sql

  jpa:
    defer-datasource-initialization: true
```

하지만 init을 추가해도 계속해서 오류가 발생하였다.  
오류를 검색해도 마땅한 해결책이 안나와서 sql문이 혹시 잘 못 되었나하고 sql문도 계속 살펴 봤는데 크게 문제는 없었다.

```
//** schema.sql */
DROP TABLE IF EXISTS tm_usr_usr CASCADE;
DROP TABLE IF EXISTS th_usr_usr CASCADE;
DROP TABLE IF EXISTS tm_usr_cnt CASCADE;
DROP TABLE IF EXISTS th_usr_cnt CASCADE;

CREATE TABLE tm_usr_usr (
        usr_no bigint AUTO_INCREMENT,
        usr_id varchar(100),
        usr_nm varchar(100),
        usr_pw varchar(100),
        is_dlt varchar(20),
        reg_no bigint,
        reg_dtm timestamp,
        udt_no bigint,
        udt_dtm timestamp,
        primary key (usr_no)
    )

...

CREATE TABLE th_usr_cnt (
        cnt_no bigint AUTO_INCREMENT,
        usr_no bigint,
        comt_cd varchar(255),
        comt_nm varchar(255),
        zip_cd int,
        adr varchar(255),
        is_dlt varchar(20),
        reg_no bigint,
        reg_dtm timestamp,
        udt_no bigint,
        udt_dtm timestamp,
        primary key (usr_no)
    )
```

그래서 직접 h2 DB에 위의 명령어를 입력해봤는데 아래와 같이 아주 잘 생성되었다.
![sql-error1](/assets/images/sql-error1.png)

그런데 한번에 테이블을 생성하기 위해 create문 여러개를 한번에 입력했더니 오류가 발생했다.
![sql-error2](/assets/images/sql-error2.png)

위의 오류를 찾아보면 다 예약어를 사용했다고 나온다. 하나씩 생성했을 때 오류가 발생하지 않는 걸 보면 예약어 때문은
아니어서 또 고민하고 있는데 보인 **콜론(;)**.. DROP문에는 끝에 콜론을 입력했는데 create문은 끝에 콜론을 입력하지 않았고,
콜론을 추가하니 오류가 발생하지 않고 테이블도 잘 생성되었다.

```
DROP TABLE IF EXISTS tm_usr_usr CASCADE;
DROP TABLE IF EXISTS th_usr_usr CASCADE;
DROP TABLE IF EXISTS tm_usr_cnt CASCADE;
DROP TABLE IF EXISTS th_usr_cnt CASCADE;

CREATE TABLE tm_usr_usr (
        usr_no bigint AUTO_INCREMENT,
        usr_id varchar(100),
        usr_nm varchar(100),
        usr_pw varchar(100),
        is_dlt varchar(20),
        reg_no bigint,
        reg_dtm timestamp,
        udt_no bigint,
        udt_dtm timestamp,
        primary key (usr_no)
    );

CREATE TABLE th_usr_usr (
        usr_no bigint AUTO_INCREMENT,
        usr_id varchar(100),
        usr_nm varchar(100),
        usr_pw varchar(100),
        is_dlt varchar(20),
        reg_no bigint,
        reg_dtm timestamp,
        udt_no bigint,
        udt_dtm timestamp,
        primary key (usr_no)
    );
CREATE TABLE tm_usr_cnt (
        cnt_no bigint AUTO_INCREMENT,
        usr_no bigint,
        comt_cd varchar(255),
        comt_nm varchar(255),
        zip_cd int,
        adr varchar(255),
        is_dlt varchar(20),
        reg_no bigint,
        reg_dtm timestamp,
        udt_no bigint,
        udt_dtm timestamp,
        primary key (usr_no)
    );

CREATE TABLE th_usr_cnt (
        cnt_no bigint AUTO_INCREMENT,
        usr_no bigint,
        comt_cd varchar(255),
        comt_nm varchar(255),
        zip_cd int,
        adr varchar(255),
        is_dlt varchar(20),
        reg_no bigint,
        reg_dtm timestamp,
        udt_no bigint,
        udt_dtm timestamp,
        primary key (usr_no)
    );
```

코드를 입력할 때 잘 살펴보며 입력해야겠다.