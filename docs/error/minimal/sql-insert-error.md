---
title: sql insert문 오류
layout: default
parent: Spring Boot
grand_parent: Error
nav_order: 3
---

{: .warning }
Caused by: org.h2.jdbc.JdbcSQLSyntaxErrorException: Column "userid" not found; SQL statement:


더미데이터를 만들기 위해 data.sql에 INSERT문으로 데이터를 입력하였는데 위와 같은 에러가 떴다. 처음에는 sql문을 잘 못 쓴 줄 알고 고쳐봤는데 딱히 틀린 곳이 없었다.  
구글링하다가 나와 [같은 오류]인 글을 봤는데 쌍따옴표가 아닌 홑따옴표로 감싸야 한다는 거였다.. 대체 왜 이렇게 만든건지 모르겠다만, 덕분에 잘 해결하였다.


```
//** 오류발생 sql문 */
INSERT INTO TM_USR_USR(usr_id,
                       usr_nm,
                       ...
                      )
       VALUES("userid",
              "username",
              ...
            );
```

```
//** 오류해결 sql문 */
INSERT INTO TM_USR_USR(usr_id,
                       usr_nm,
                       ...
                      )
       VALUES('userid',
              'username',
              ...
            );
```

{: .warning }
H2 조회시 한글 깨짐 오류

데이터가 잘 들어갔나 확인을 했는데, 한글로 넣었더니 조회시 깨짐 현상이 발생하였다.
![sql-insert-error1](/assets/images/sql-insert-error1.png)

해결 방법은 application.yml파일에 ['utf-8']을 넣어주면 된다.
![sql-insert-error2](/assets/images/sql-insert-error2.png)

```
//** application.yml */
spring:
  sql:
    init:
      encoding: utf-8
```

[같은 오류]: https://www.inflearn.com/questions/492997/h2%EC%BD%98%EC%86%94%EC%B0%BD%EC%97%90%EC%84%9C-insert-into%EA%B5%AC%EB%AC%B8-%EC%98%A4%EB%A5%98%EB%82%A9%EB%8B%88%EB%8B%A4 "H2 INSERT오류"
['utf-8']: https://blog.karsei.pe.kr/120 "H2 한글깨짐 오류"
