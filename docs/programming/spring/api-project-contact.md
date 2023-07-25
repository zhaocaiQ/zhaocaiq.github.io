---
title: API프로젝트-1 회원정보 CRUD
layout: default
parent: Spring
grand_parent: Programming
---

### **목차**
  
- [구현코드](#구현코드)    
- [실행결과](#실행결과)   

***

# 테이블확장

기존 회원 CRUD에서 회원정보 테이블을 확장하여 가입과 탈퇴 및 삭제하는 API를 만들어보았다.
{: .fs-6 .fw-300 }

## 구현코드    

### **Entity**   

- UserContactEntity    

```
import lombok.Builder;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Builder
public class UserContactEntity {
    private Long cntNo;           //유저정보번호
    private Long usrNo;           //유저번호
    private String comtCd;        //법정동코드
    private String comtNm;        //법정동명
    private String zipCd;         //우편번호
    private String adr;           //주소
    private String adrDtl;        //상세주소
    private String isDlt;          //사용여부
    private String regNo;          //등록자번호
    private LocalDateTime regDtm;  //등록날짜
    private String udtNo;          //수정자번호
    private LocalDateTime udtDtm;  //수정날짜
}
```

- UserContactHistoryEntity    

```
import lombok.Builder;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Builder
public class UserContactHistoryEntity {
    private Long cntNo;           //유저정보번호
    private Long usrNo;           //유저번호
    private String comtCd;        //법정동코드
    private String comtNm;        //법정동명
    private String zipCd;         //우편번호
    private String adr;           //주소
    private String adrDtl;        //상세주소
    private String isDlt;          //사용여부
    private String regNo;          //등록자번호
    private LocalDateTime regDtm;  //등록날짜
    private String udtNo;          //수정자번호
    private LocalDateTime udtDtm;  //수정날짜
}
```

## **Dto**    

이전 게시글에서는 회원정보만 가지고 회원가입을 했기 때문에 UserEntity를 이용하여 정보를 받았지만, 회원정보 테이블이 추가 되어 따로 정보를 받는 Dto를 생성하였다.   

- SignUpUserDto    

```
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class SignUpUserDto {
    private String usrId;          //사용자아이디
    private String usrNm;          //사용자이름
    private String usrPw;          //사용자비밀번호
    private String comtCd;         //법정동코드
    private String comtNm;         //법정동명
    private String zipCd;          //우편번호
    private String adr;            //주소
    private String adrDtl;         //상세주소
    private String isDlt;          //사용여부
    private String regNo;          //등록자번호
    private LocalDateTime regDtm;  //등록날짜
    private String udtNo;          //수정자번호
    private LocalDateTime udtDtm;  //수정날짜
}

```


### **Service**  

- UserService    

```
import com.education3.education3.dto.SignUpUserDto;
import com.education3.education3.entity.user.UserEntity;

import java.util.List;

public interface UserService {
    List<UserEntity> findAllUSer();
    UserEntity findUserByUserNo(Long usrNo);
    Boolean createUser(SignUpUserDto signUpUserDto);
    Boolean updateUser(Long id, String udtNo);
    Boolean deleteUser(Long id);

    //추가코드
    Boolean createUserContact(SignUpUserDto signUpUserDto);
    Boolean updateUserContact(Long id, String udtNo);
    Boolean deleteUserContact(Long id);
}
```
- UserServiceImpl   

기존 UserServiceImpl.java에 추가로 구현   

```
import com.education3.education3.dto.SignUpUserDto;
import com.education3.education3.dto.UpdateUserDto;
import com.education3.education3.entity.user.UserContactEntity;
import com.education3.education3.entity.user.UserContactHistoryEntity;
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

    ...
    //추가코드
    @Override
    public Boolean createUserContact(SignUpUserDto signUpUserDto) {
        Optional<UserEntity> existUser = userMapper.selectUserByUserId(signUpUserDto.getUsrId());
        if (existUser.isPresent()){
            UserEntity user = existUser.get();
            try {
                UserContactEntity userContact = UserContactEntity.builder()
                        .usrNo(user.getUsrNo())
                        .comtCd(signUpUserDto.getComtCd())
                        .comtNm(signUpUserDto.getComtNm())
                        .zipCd(signUpUserDto.getZipCd())
                        .adr(signUpUserDto.getAdr())
                        .adrDtl(signUpUserDto.getAdrDtl())
                        .isDlt(user.getIsDlt())
                        .regNo(signUpUserDto.getRegNo())
                        .regDtm(user.getRegDtm())
                        .udtNo(user.getUdtNo())
                        .udtDtm(user.getUdtDtm())
                        .build();
                userMapper.insertUserContact(userContact);
                UserContactHistoryEntity userContactHistory = UserContactHistoryEntity.builder()
                        .usrNo(user.getUsrNo())
                        .comtCd(signUpUserDto.getComtCd())
                        .comtNm(signUpUserDto.getComtNm())
                        .zipCd(signUpUserDto.getZipCd())
                        .adr(signUpUserDto.getAdr())
                        .adrDtl(signUpUserDto.getAdrDtl())
                        .isDlt(user.getIsDlt())
                        .regNo(signUpUserDto.getRegNo())
                        .regDtm(user.getRegDtm())
                        .udtNo(user.getUdtNo())
                        .udtDtm(user.getUdtDtm())
                        .build();
                userMapper.insertUserContactHistory(userContactHistory);
                return true;
            } catch (Exception err) {
                System.out.println(err);
                return false;
            }
        }
        return false;
    }

    @Override
    public Boolean updateUserContact(Long id, String udtNo) {
        Optional<UserContactEntity> existUserCont = userMapper.selectUserContactByUserNo(id);
        if (existUserCont.isPresent()) {
            UserContactEntity userContact = existUserCont.get();
            LocalDateTime timeNow = LocalDateTime.now();
            UpdateUserDto updateUserDto = UpdateUserDto.builder()
                    .usrNo(id)
                    .isDlt("Y")
                    .udtNo(udtNo)
                    .udtDtm(timeNow)
                    .build();
            userMapper.updateUserContact(updateUserDto);

            UserContactHistoryEntity userContactHistory = UserContactHistoryEntity.builder()
                    .usrNo(userContact.getUsrNo())
                    .comtCd(userContact.getComtCd())
                    .comtNm(userContact.getComtNm())
                    .zipCd(userContact.getZipCd())
                    .adr(userContact.getAdr())
                    .adrDtl(userContact.getAdrDtl())
                    .isDlt(updateUserDto.getIsDlt())
                    .regNo(userContact.getRegNo())
                    .regDtm(userContact.getRegDtm())
                    .udtNo(updateUserDto.getUdtNo())
                    .udtDtm(updateUserDto.getUdtDtm())
                    .build();
            userMapper.insertUserContactHistory(userContactHistory);
            return true;
        }
        return false;
    }

    @Override
    public Boolean deleteUserContact(Long id) {
        Optional<UserContactEntity> usercont = userMapper.selectUserContactByUserNo(id);
        if (usercont.isPresent() && usercont.get().getIsDlt().equals("Y")) {
            userMapper.deleteUserContact(id);
            return true;
        }
        return false;
    }
}
```

### **Mapper**  

- UserMapper   

```
import com.education3.education3.dto.UpdateUserDto;
import com.education3.education3.entity.user.UserContactEntity;
import com.education3.education3.entity.user.UserContactHistoryEntity;
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

    //추가코드
    int insertUserContact(UserContactEntity userContactEntity);
    int insertUserContactHistory(UserContactHistoryEntity userContactHistory);
    Optional<UserContactEntity> selectUserContactByUserNo(Long usrNo);
    int updateUserContact(UpdateUserDto updateUserDto);
    int deleteUserContact(Long id);
}
```

- UserMapper.xml   

```
<mapper namespace="com.education3.education3.mapper.UserMapper">

    ...
    <!--  추가코드  -->
    <!--  사용자정보 조회  -->
    <select id="selectUserContactByUserNo">
        SELECT *
        FROM   TD_USR_CNT
        WHERE  USR_NO = #{usrNo}
    </select>

    <!--  사용자정보 생성  -->
    <insert id="insertUserContact">
        INSERT INTO TD_USR_CNT(USR_NO,  /* 유저번호 */
                               COMT_CD, /* 법정동코드 */
                               COMT_NM, /* 법정동명 */
                               ZIP_CD,  /* 우편번호 */
                               ADR,     /* 주소 */
                               ADR_DTL, /* 상세주소 */
                               IS_DLT,  /* 사용여부 */
                               REG_NO,  /* 등록자번호 */
                               REG_DTM, /* 등록날짜 */
                               UDT_NO,  /* 수정자번호 */
                               UDT_DTM) /* 수정자날짜 */
               VALUES(#{usrNo},
                      #{comtCd},
                      #{comtNm},
                      #{zipCd},
                      #{adr},
                      #{adrDtl},
                      #{isDlt},
                      #{regNo},
                      #{regDtm},
                      #{udtNo},
                      #{udtDtm})

    </insert>

    <!--  사용자정보 로그 생성  -->
    <insert id="insertUserContactHistory">
        INSERT INTO TH_USR_CNT(USR_NO,  /* 유저번호 */
                               COMT_CD, /* 법정동코드 */
                               COMT_NM, /* 법정동명 */
                               ZIP_CD,  /* 우편번호 */
                               ADR,     /* 주소 */
                               ADR_DTL, /* 상세주소 */
                               IS_DLT,  /* 사용여부 */
                               REG_NO,  /* 등록자번호 */
                               REG_DTM, /* 등록날짜 */
                               UDT_NO,  /* 수정자번호 */
                               UDT_DTM) /* 수정자날짜 */
               VALUES(#{usrNo},
                      #{comtCd},
                      #{comtNm},
                      #{zipCd},
                      #{adr},
                      #{adrDtl},
                      #{isDlt},
                      #{regNo},
                      #{regDtm},
                      #{udtNo},
                      #{udtDtm})
    </insert>

    <update id="updateUserContact">
        UPDATE TD_USR_CNT
        SET    IS_DLT  = #{isDlt}, /* 사용여부 */
               UDT_NO  = #{udtNo}, /* 수정자번호 */
               UDT_DTM = #{udtDtm} /* 수정날짜 */
        WHERE  USR_NO  = #{usrNo}  /* 유저번호 */
    </update>

    <!-- 사용자정보 삭제 -->
    <delete id="deleteUserContact">
        DELETE
        FROM  TD_USR_CNT
        WHERE USR_NO = #{usrNo}
    </delete>

</mapper>
```

### **Controller**   

조회API를 제외하고는 가입과 탈퇴 및 삭제시 회원과 함께 회원정보 service도 같이 처리되도록 코드를 수정하였다.   

- UserController       

```
import com.education3.education3.dto.SignUpUserDto;
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
    public String signUpUser(@RequestBody SignUpUserDto signUpUserDto){
        Boolean result = userService.createUser(signUpUserDto);
        if (result){
            Boolean resultCon = userService.createUserContact(signUpUserDto);
            if (resultCon) {
                return "Success sign up";
            }
            return "Not Success save your contact. Please check your info";
        }
        return "Not Success sign up. Please check your info";
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
        Boolean result = userService.updateUser(id, udtNo);
        if (result) {
            Boolean resultCont = userService.updateUserContact(id, udtNo);
            if (resultCont) {
                return "Success update user";
            }
            return "Not success update user Contact";
        }
        return "Not success update User";
    }

    @DeleteMapping("/v1/user/{id}")
    public String deleteUser(@PathVariable Long id){
        Boolean resultCont = userService.deleteUserContact(id);
        if (resultCont) {
            Boolean result = userService.deleteUser(id);
            if (result) {
                return "Success delete user";
            }
            return "Success delete user contact but Not success user";
        }
        return "Not success delete user";
    }
}
```

#### **테이블, 레코드 생성 코드**

- schema.sql   

```
--추가코드
DROP TABLE IF EXISTS td_usr_cnt CASCADE;
DROP TABLE IF EXISTS th_usr_cnt CASCADE;

--유저주소정보 테이블
CREATE TABLE td_usr_cnt (
        cnt_no bigint AUTO_INCREMENT,
        usr_no bigint,        --사용자 번호
        comt_cd varchar(255), --법정동 코드
        comt_nm varchar(255), --법정동명(시/도/군)
        zip_cd varchar(10),   --우편번호
        adr varchar(255),     --사용자 주소
        adr_dtl varchar(100), --사용자 상세주소
        is_dlt varchar(20),   --사용여부(Y,N)
        reg_no varchar(20),   --등록자 번호
        reg_dtm timestamp,    --등록날짜
        udt_no varchar(20),   --수정자 번호
        udt_dtm timestamp,    --수정날짜
        primary key (cnt_no, usr_no)
    );

CREATE TABLE th_usr_cnt (
        cnt_no bigint AUTO_INCREMENT,
        usr_no bigint,        --사용자 번호
        comt_cd varchar(255), --법정동 코드
        comt_nm varchar(255), --법정동명(시/도/군)
        zip_cd varchar(10),   --우편번호
        adr varchar(255),     --사용자 주소
        adr_dtl varchar(100), --사용자 상세주소
        is_dlt varchar(20),   --사용여부(Y,N)
        reg_no varchar(20),   --등록자 번호
        reg_dtm timestamp,    --등록날짜
        udt_no varchar(20),   --수정자 번호
        udt_dtm timestamp,    --수정날짜
        primary key (cnt_no, usr_no)
    );
```

## 실행결과

### **유저 생성**
- method: post   
- url: localhost:8085/v1/user   

**[실행 화면]**   
![api-project2-1](/assets/images/api-project2-1.png)   

**[결과]**   
![api-project2-2](/assets/images/api-project2-2.png)   

### **유저 수정**
- method: put   
- url: localhost:8085/v1/user/2   

**[실행 화면]**   
![api-project2-3](/assets/images/api-project2-3.png)   

**[결과]**   
![api-project2-4](/assets/images/api-project2-4.png)   

### **유저 삭제**
- method: delete   
- url: localhost:8085/v1/user/2   

**[실행 화면]**   
![api-project2-5](/assets/images/api-project2-5.png)   

**[결과]**   
![api-project2-6](/assets/images/api-project2-6.png)   