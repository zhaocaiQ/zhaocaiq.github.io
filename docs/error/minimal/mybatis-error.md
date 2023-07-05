---
title: Mybatis error
layout: default
parent: Spring Boot
grand_parent: Error
nav_order: 3
---

{: .warning }
cannot invoke because this.mapper is null

JPA를 이용하여 구현했었는데 sql문을 직접 사용하여 구현해야한다고 해서 Mybatis(마이바티스)로 코드를 변경했다. 그리고 DB 테이블도 추가되어 전체적으로 다시 구현하게 되었는데, 문제는 코드 구현하고 실행하려고 하니 자꾸 mapper is null이라는 오류가 발생하는 것이었다.  
찾아보니 service에서 불러온 mapper에 [@Autowired 어노테이션을 해주지 않아] 발생한 거였다.

```
//** UserServiceImpl.java */
@Service
public class UserServiceImpl implements UserService{

    @Autowired
    private UserMapper userMapper;

    ...
}
```


실행을 했지만 또 다른 오류가 발생하였다..

{: .warning }
Invalid bound statement (not found)

Mapper 인터페이스와 Mapper.xml에 id가 달라서 혹은 오타가 있어서 application.yml파일에 오타가 있어서 등 [다양한 이유로 오류가 발생]한다고 되어 있는데 아무리 봐도 id를 다르게 입력하지도 않았고 오타도 없었다. Mapper.xml에 입력한 코드가 문제였나 싶어서 sql문도 여러번 확인해 봤지만 원인을 찾을 수 없었다.
하지만 역시 구글링을 하다보니 답을 찾을 수 있었다. application.yml파일에서 [들여쓰기를 잘못]해서 mybatis를 spring 안에 쓴거였다.

```
//** application.yml */
spring:
  jpa:
    defer-datasource-initialization: true

  ...

  # model 프로퍼티 camel case 설정
  mybatis:
    configuration:
      map-underscore-to-camel-case: true
  
    # xml파일 result type에 패키지명을 생략할 수 있도록 alias 설정
    type-aliases-package: com.education.education2
  
    # Mybatis mapper 위치 설정
    mapper-locations: mapper/*.xml
```

위의 코드를 아래와 같이 바꾸니 오류없이 실행이 되었다.

```
//** application.yml */
spring:
  jpa:
    defer-datasource-initialization: true

  ...

# model 프로퍼티 camel case 설정
mybatis:
configuration:
    map-underscore-to-camel-case: true

# xml파일 result type에 패키지명을 생략할 수 있도록 alias 설정
type-aliases-package: com.education.education2

# Mybatis mapper 위치 설정
mapper-locations: mapper/*.xml
```

[@Autowired 어노테이션을 해주지 않아]: https://wakestand.tistory.com/701 "this.mapper is null 오류해결"
[다양한 이유로 오류가 발생]: https://madplay.github.io/post/mybatis-invalid-bound-statement-not-found-error "Invalid bound statement (not found) 발생하는 이유들"
[들여쓰기를 잘못]: https://dong-co.tistory.com/117 "Invalid bound statement (not found) 오류해결"