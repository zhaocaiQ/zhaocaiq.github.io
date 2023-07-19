---
title: API프로젝트-1 회원 CRUD
layout: default
parent: Spring
grand_parent: Programming
nav_exclude: true
---


## application.yml에 필요한 설정입력   


```
//** application.yml */

server:
  port: 8085

spring:
  #H2 환경설정
  h2:
    console:
      enabled: true #H2 console을 사용할지 여부
      path: /h2-console #H2 console 접근 path

  #H2 연결을 위한 정보입력
  datasource:
    driver-class-name: org.h2.Driver #데이터베이스로 H2사용 선언
    url: jdbc:h2:~/education
    username: sa
    password:

  sql:
    init:
      mode: always #생성 설정 always: 서버 재가동마다 schema, data 새로생성
      schema-locations: classpath:schema.sql #스키마생성 sql문 기입 파일위치
      data: classpath:data.sql #데이터 삽입 sql문 기입 파일위치
      encoding: utf-8 #H2 한글깨짐 현상 해결

  mvc:
    converters:
      preferred-json-mapper: gson #gson 사용 선언

mybatis:
  configuration:
    map-underscore-to-camel-case: true #카멜표기법으로 설정

  #xml파일 result type에 패키지명 생략할 수 있도록 설정
  type-aliases-package: com.education3.education3

  #Mybatis mapper 위치 설정
  mapper-locations: mapper/*.xml
```

## application.yml에 필요한 설정입력  

