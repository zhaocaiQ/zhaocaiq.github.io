---
title: API프로젝트-1 회원 CRUD
layout: default
parent: Spring-Boot
grand_parent: Programming
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

***


## MVC를 위한 폴더구조   

![api-project1-1](/assets/images/api-project1-1.png)    

controller: 요청과 응답이 이루어짐
dto: entity와 비슷함. 요청/응답에 필요한 객체를 생성하여 이용
entity: 데이터베이스 테이블에 필요한 컬럼 설정
mapper: interface로 필요한 함수 선언
service:
   - service interface: 필요한 함수 선언 
   - service Impl: controller에서 들어온 요청이 실행되는 곳    

***

## 구현 코드   

#### Entity   
- UserEntity   

```
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Builder
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserEntity {
    private Long usrNo;            //사용자번호
    private String usrId;          //사용자아이디
    private String usrNm;          //사용자이름
    private String usrPw;          //사용자비밀번호
    private String isDlt;          //사용여부
    private String regNo;          //등록자번호
    private LocalDateTime regDtm;  //등록날짜
    private String udtNo;          //수정자번호
    private LocalDateTime udtDtm;  //수정날짜
}
```

- UserHistoryEntity   

```
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Builder
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserHistoryEntity {
    private Long usrNo;            //사용자번호
    private String usrId;          //사용자아이디
    private String usrNm;          //사용자이름
    private String usrPw;          //사용자비밀번호
    private String isDlt;          //사용여부
    private String regNo;          //등록자번호
    private LocalDateTime regDtm;  //등록날짜
    private String udtNo;          //수정자번호
    private LocalDateTime udtDtm;  //수정날짜
}
```

#### Service      
- UserService    
```
import com.education3.education3.entity.user.UserEntity;

import java.util.List;

public interface UserService {
    List<UserEntity> findAllUSer();
    UserEntity findUserByUserNo(Long usrNo);

    String createUser(UserEntity userEntity);
    String updateUser(Long id, String udtNo);
    String deleteUser(Long id);

}
```

- UserServiceImpl    
```
import com.education3.education3.dto.UpdateUserDto;
import com.education3.education3.entity.user.UserEntity;
import com.education3.education3.entity.user.UserHistoryEntity;
import com.education3.education3.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Service
public class UserServiceImpl implements UserService{

    @Autowired
    private UserMapper userMapper;
    @Override
    public List<UserEntity> findAllUSer() {
        List<UserEntity> userList = userMapper.selectAllUser();
        if (userList.isEmpty()){
            List<UserEntity> emptyList = new ArrayList<>();
            return emptyList;
        }
        return userList;
    }

    @Override
    public UserEntity findUserByUserNo(Long usrNo) {
        Optional<UserEntity> user = userMapper.selectUserByUserNo(usrNo);
        if (user.isPresent()){
            return user.get();
        }
        return null;
    }

    @Override
    public String createUser(UserEntity userEntity) {
        try{
            LocalDateTime timeNow = LocalDateTime.now();
            if (userEntity.getRegNo() == null){
                userEntity.setRegNo("001");
            }
            if (userEntity.getUdtNo() == null) {
                userEntity.setUdtNo("001");
            }
            UserEntity user = UserEntity.builder()
                    .usrId(userEntity.getUsrId())
                    .usrNm(userEntity.getUsrNm())
                    .usrPw(userEntity.getUsrPw())
                    .isDlt("N")
                    .regNo(userEntity.getRegNo())
                    .regDtm(timeNow)
                    .udtNo(userEntity.getUdtNo())
                    .udtDtm(timeNow)
                    .build();
            UserHistoryEntity userHst = UserHistoryEntity.builder()
                    .usrId(userEntity.getUsrId())
                    .usrNm(userEntity.getUsrNm())
                    .usrPw(userEntity.getUsrPw())
                    .isDlt("N")
                    .regNo(userEntity.getRegNo())
                    .regDtm(timeNow)
                    .udtNo(userEntity.getUdtNo())
                    .udtDtm(timeNow)
                    .build();
            userMapper.insertUser(user);
            userMapper.insertUserHst(userHst);
            return "Success create user";
        } catch (Exception err){
            System.out.println(err);
            return "Not success creat user";
        }
    }

    @Override
    public String updateUser(Long id, String udtNo) {
        Optional<UserEntity> existUser = userMapper.selectUserByUserNo(id);
        if(existUser.isPresent() && existUser.get().getIsDlt().equals("N")){
            UserEntity user = existUser.get();
            LocalDateTime timeNow = LocalDateTime.now();
            if (udtNo == null) {
                udtNo = "001";
            }
            UpdateUserDto updateUserDto = UpdateUserDto.builder()
                    .usrNo(id)
                    .udtNo(udtNo)
                    .isDlt("Y")
                    .udtDtm(timeNow)
                    .build();
            userMapper.updateUser(updateUserDto);
            UserHistoryEntity userHistory = UserHistoryEntity.builder()
                                    .usrId(user.getUsrId())
                                    .usrNm(user.getUsrNm())
                                    .usrPw(user.getUsrPw())
                                    .isDlt("Y")
                                    .regNo(udtNo)
                                    .regDtm(timeNow)
                                    .udtNo(udtNo)
                                    .udtDtm(timeNow)
                                    .build();
            userMapper.insertUserHst(userHistory);
            return "Success update user";
        }
        return "Not exist user";
    }

    @Override
    public String deleteUser(Long id) {
        Optional<UserEntity> user = userMapper.selectUserByUserNo(id);
        if(user.isPresent() && user.get().getIsDlt().equals("Y")) {
            userMapper.deleteUser(id);
            return "Success delete user";
        }
        return "Not exist user";
    }
}
```

#### Mapper    

- UserMapper   
```
import com.education3.education3.dto.UpdateUserDto;
import com.education3.education3.entity.user.UserEntity;
import com.education3.education3.entity.user.UserHistoryEntity;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;
import java.util.Optional;

@Mapper
public interface UserMapper {

    List<UserEntity> selectAllUser();
    Optional<UserEntity> selectUserByUserId(String usrId);
    Optional<UserEntity> selectUserByUserNo(Long usrNo);
    int insertUser(UserEntity userEntity);
    int insertUserHst(UserHistoryEntity userHistoryEntity);
    int updateUser(UpdateUserDto updateUserDto);
    int deleteUser(Long usrNo);
}
```

- UserMapper.xml    
```
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.education3.education3.mapper.UserMapper">
    <!-- 모든 사용자 조회 -->
    <select id="selectAllUser" resultType="UserEntity">
        SELECT *
        FROM   TM_USR_USR
    </select>

    <!-- 사용자아이디로 사용자 조회 -->
    <select id="selectUserByUserId" resultType="UserEntity">
        SELECT *
        FROM   TM_USR_USR
        WHERE  USR_ID = #{usrId}
    </select>

    <!-- 사용자번호로 사용자 조회 -->
    <select id="selectUserByUserNo" resultType="UserEntity">
        SELECT *
        FROM   TM_USR_USR
        WHERE  USR_NO = #{usrNo}
    </select>

    <!-- 사용자등록 -->
    <insert id="insertUser">
        INSERT INTO TM_USR_USR(USR_ID,  /* 유저아이디 */
                               USR_NM,  /* 유저이름 */
                               USR_PW,  /* 유저비밀번호 */
                               IS_DLT,  /* 사용여부 */
                               REG_NO,  /* 등록자번호 */
                               REG_DTM, /* 등록날짜 */
                               UDT_NO,  /* 수정자번호 */
                               UDT_DTM) /* 수정날짜 */
               VALUES(#{usrId},
                      #{usrNm},
                      #{usrPw},
                      #{isDlt},
                      #{regNo},
                      #{regDtm},
                      #{udtNo},
                      #{udtDtm})
    </insert>

    <!-- 사용자등록 로그 생성 -->
    <insert id="insertUserHst">
        INSERT INTO TH_USR_USR(USR_ID,  /* 유저아이디 */
                               USR_NM,  /* 유저이름 */
                               USR_PW,  /* 유저비밀번호 */
                               IS_DLT,  /* 사용여부 */
                               REG_NO,  /* 등록자번호 */
                               REG_DTM, /* 등록날짜 */
                               UDT_NO,  /* 수정자번호 */
                               UDT_DTM) /* 수정날짜 */
               VALUES(#{usrId},
                      #{usrNm},
                      #{usrPw},
                      #{isDlt},
                      #{regNo},
                      #{regDtm},
                      #{udtNo},
                      #{udtDtm})
    </insert>

    <!-- 사용자삭제 시, 사용여부 변경("N" -> "Y") -->
    <update id="updateUser">
        UPDATE TM_USR_USR
        SET    IS_DLT = #{isDlt},   /* 사용여부 */
               UDT_NO = #{udtNo} ,  /* 수정자번호 */
               UDT_DTM = #{udtDtm} /* 수정날짜 */
        WHERE  USR_NO = #{usrNo}    /* 유저번호 */
    </update>

    <!-- 사용자 삭제 -->
    <delete id="deleteUser">
        DELETE
        FROM  TM_USR_USR
        WHERE USR_NO = #{usrNo}
    </delete>
</mapper>
```

#### Controller   

- UserController       
```
import com.education3.education3.entity.user.UserEntity;
import com.education3.education3.service.User.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping("/v1/user")
    public String signUpUser(@RequestBody UserEntity userEntity){
        String result = userService.createUser(userEntity);
        return result;
    }

    @GetMapping("/v1/user/all")
    public List<UserEntity> getAllUsers(){
        List<UserEntity> userEntityList = userService.findAllUSer();
        return userEntityList;
    }

    @GetMapping("/v1/user/{id}")
    public UserEntity getUser(@PathVariable Long id){
        UserEntity user = userService.findUserByUserNo(id);
        return user;
    }

    @PutMapping("/v1/user/{id}")
    public String updateUser(@PathVariable Long id,
                             @RequestBody String udtNo){
        String result = userService.updateUser(id, udtNo);
        return result;
    }

    @DeleteMapping("/v1/user/{id}")
    public String deleteUser(@PathVariable Long id){
        String result = userService.deleteUser(id);
        return result;
    }
}
```

##### 테이블, 레코드 생성 코드

- schema.sql   
```
DROP TABLE IF EXISTS tm_usr_usr CASCADE;
DROP TABLE IF EXISTS th_usr_usr CASCADE;

--유저정보 테이블
CREATE TABLE tm_usr_usr (
        usr_no bigint AUTO_INCREMENT,
        usr_id varchar(100), --사용자아이디
        usr_nm varchar(100), --사용자 이름
        usr_pw varchar(100), --사용자 비밀번호
        is_dlt varchar(20),  --사용여부(Y,N)
        reg_no varchar(20),  --등록자 번호
        reg_dtm timestamp,   --등록날짜
        udt_no varchar(20),  --수정자 번호
        udt_dtm timestamp,   --수정날짜
        primary key (usr_no)
    );

CREATE TABLE th_usr_usr (
        usr_no bigint AUTO_INCREMENT,
        usr_id varchar(100), --사용자아이디
        usr_nm varchar(100), --사용자 이름
        usr_pw varchar(100), --사용자 비밀번호
        is_dlt varchar(20),  --사용여부(Y,N)
        reg_no varchar(20),  --등록자 번호
        reg_dtm timestamp,   --등록날짜
        udt_no varchar(20),  --수정자 번호
        udt_dtm timestamp,   --수정날짜
        primary key (usr_no)
    );

```

- data.sql    
```
--유저정보
INSERT INTO TM_USR_USR(usr_id,
                       usr_nm,
                       usr_pw,
                       is_dlt,
                       reg_no,
                       reg_dtm,
                       udt_no,
                       udt_dtm)
       VALUES('userid2',
              'username2',
              'user123',
              'N',
              '001',
              '2021-07-07 17:03:22.822812',
              '001',
              '2021-07-07 17:03:22.822812');

INSERT INTO TH_USR_USR(USR_ID,
                       USR_NM,
                       USR_PW,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('userid2',
              'username2',
              'user123',
              'N',
              '001',
              '2023-07-07 17:03:22.822812',
              '001',
              '2023-07-07 17:03:22.822812');
```


***

## 발생 오류      


{: .warning }
Cannot invoke "com.education3.education3.service.User.UserService.createUser(com.education3.education3.entity.user.UserEntity)" because "this.userService" is null

- Controller에서 UserService에 @Autowired 어노테이션을 설정하지 않아서 생긴 오류   


[오류 코드]   

```
@RestController
public class UserController {
    
    private UserService userService;
    ...
}
```

[해결 코드]   

```
@RestController
public class UserController {
    
    @Autowired
    private UserService userService;
    ...
}
```

{: .warning }
Invalid bound statement (not found)

- 이전에 [오류 게시글]에도 올렸었는데 다른 이유로 또 발생하였다.    
이번엔 Mapper.xml에서 패키지 경로를 잘못 입력해서 였다.(education3로 입력해야 했는데 education으로 입력)    

[오류 코드]   

```
<mapper namespace="com.education.education3.mapper.UserMapper">
 ...
</mapper>
```

[해결 코드]   

```
<mapper namespace="com.education3.education3.mapper.UserMapper">
 ...
</mapper>
```


{: .warning }
Required URI template variable 'usrNo' for method parameter type Long is not present

- 변수명이 서로 달라서 생긴 오류   
url 경로에는 {id}로 입력하고선 @PathVariable에서는 usrNo으로 입력함   

[오류 코드]   

```
@GetMapping("/v1/user/{id}")
public UserEntity getUser(@PathVariable Long usrNo){
    UserEntity user = userService.findUserByUserNo(usrNo);
    return user;
}
```

[해결 코드]   

```
@GetMapping("/v1/user/{id}")
public UserEntity getUser(@PathVariable Long id){
    UserEntity user = userService.findUserByUserNo(id);
    return user;
}
```

## 실행 결과

- 유저 생성(method: post, url: localhost:8085/v1/user)

[기존 테이블]   
![api-project1-2](/assets/images/api-project1-2.png)   

[실행 화면]    
![api-project1-3](/assets/images/api-project1-3.png)   

[결과]   
![api-project1-4](/assets/images/api-project1-4.png)    

- 유저 조회   
[모든유저 조회 결과]    
![api-project1-5](/assets/images/api-project1-5.png)   

[특정유저 조회 결과]   
![api-project1-6](/assets/images/api-project1-6.png)    

- 유저 업데이트(유저이용X)   
[실행 화면]    
![api-project1-7](/assets/images/api-project1-7.png)    

[결과]   
- master테이블   
![api-project1-8](/assets/images/api-project1-8.png)   
- history테이블   
![api-project1-9](/assets/images/api-project1-9.png)   

- 유저 삭제   
[실행 화면]    
![api-project1-10](/assets/images/api-project1-10.png)   

[결과]   
![api-project1-11](/assets/images/api-project1-11.png)   

[오류 게시글]: https://zhaocaiq.github.io/docs/error/minimal/mybatis-error/