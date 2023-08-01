---
title: nullpointerexception
layout: default
parent: Spring Boot
grand_parent: Error
---

{: .warning }
java.lang.nullpointerexception: cannot invoke "string.length()" because "s" is null



spring boot는 JPA 컬럼명을 생성할 때, 스네이크 표기법이 아닌 카멜 표기법을 이용한다고 한다.  
user id로 조회하는 jpa를 생성하고 싶은데 스네이크표기법으로 Entity를 생성했어서 아래와 같이 코드를 작성했는데

```
@Repository
public interface UserRepository extends JpaRepository<UserEntity, Long> {
    boolean existsByUsr_Id(String usr_id);
}
```


그렇게하다보니 함수명에 언더바가 들어가게 되었고 그 결과 오류가 발생했다.  
언더바를 없애도 해당 컬럼이 없다 그랬었나 여하튼 그런 오류가 발생하여 Entity를 모두 카멜표기법으로 바꿨다.


```
public class UserEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long usrNo;
    private String usrId;
    private String usrNm;
    private String usrPw;
    private String isDlt;
}
```


userEntity의 컬럼명을 수정하고 실행을 하니 nullpointerexception오류가 발생하였다.  
구글링해서 원인이 뭔지 몰라서 한참 찾다가 API를 요청하는 json에 키값을 안 바꾼게 눈에 보여서 수정을 하니
오류가 바로 해결되었다.


[변경전]    
```
{
    "usr_id": "asddf",
    "usr_pw": "aaaa",
    "usr_nm": "asdf"
}
```

[변경후]    
```
{
    "usrId": "asddf",
    "usrPw": "aaaa",
    "usrNm": "asdf"
}
```