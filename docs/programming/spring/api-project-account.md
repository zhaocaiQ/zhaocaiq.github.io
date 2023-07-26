---
title: API프로젝트-1 계좌정보
layout: default
parent: Spring
grand_parent: Programming
---

### **목차**
  
- [구현코드](#구현코드)    
- [실행결과](#실행결과)   

***

# 테이블확장

은행공통정보 테이블과 회원계좌 테이블을 생성하여 탈퇴(수정) 및 삭제 API 생성
{: .fs-6 .fw-300 }

## 구현코드    

### **Entity**   

- BankInfoEntity   

```
import lombok.Builder;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Builder
public class BankInfoEntity {
    private Long cmmNo;            //공통번호
    private String dv1;            //대분류
    private String dv2;            //중분류
    private String dv3;            //소분류
    private String cd;             //분류코드
    private String etc;            //비고
    private String isDlt;          //사용여부
    private String regNo;          //등록자번호
    private LocalDateTime regDtm;  //등록날짜
    private String udtNo;          //수정자번호
    private LocalDateTime udtDtm;  //수정날짜
}
```

- BankInfoHistoryEntity   

```
import lombok.Builder;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Builder
public class BankInfoHistoryEntity {
    private Long cmmNo;            //공통번호
    private String dv1;            //대분류
    private String dv2;            //중분류
    private String dv3;            //소분류
    private String cd;             //분류코드
    private String etc;            //비고
    private String isDlt;          //사용여부
    private String regNo;          //등록자번호
    private LocalDateTime regDtm;  //등록날짜
    private String udtNo;          //수정자번호
    private LocalDateTime udtDtm;  //수정날짜
}
```

- UserAccountEntity   

```
import lombok.Builder;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Builder
public class UserAccountEntity {
    private Long accNo;            //계좌정보번호
    private Long usrNo;            //유저번호
    private String acctCcd;        //계좌구분코드
    private String bkNm;           //은행명
    private String acctNo;         //계좌번호
    private String isDlt;          //사용여부
    private String regNo;          //등록자번호
    private LocalDateTime regDtm;  //등록날짜
    private String udtNo;          //수정자번호
    private LocalDateTime udtDtm;  //수정날짜
}
```

- UserAccountHistoryEntity    

```
import lombok.Builder;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Builder
public class UserAccountHistoryEntity {
    private Long accNo;            //계좌정보번호
    private Long usrNo;            //유저번호
    private String acctCcd;        //계좌구분코드
    private String bkNm;           //은행명
    private String acctNo;         //계좌번호
    private String isDlt;          //사용여부
    private String regNo;          //등록자번호
    private LocalDateTime regDtm;  //등록날짜
    private String udtNo;          //수정자번호
    private LocalDateTime udtDtm;  //수정날짜
}
```

## **Dto**    

기존 UpdateUserDto를 이용하여 Account테이블도 수정하려고 했으나 Account테이블을 수정하기 위해서는 usrNo보다 accNo이 필요하여 Dto를 새로 만들었다. 

- UpdateUserAccountDto   

```
import lombok.Builder;

import java.time.LocalDateTime;

@Builder
public class UpdateUserAccountDto {
    private Long accNo;
    private String isDlt;
    private String udtNo;
    private LocalDateTime udtDtm;
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
    Boolean updateUserAccount(Long id, String udtNo);
    Boolean deleteUserAccount(Long id);
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
    public Boolean updateUserAccount(Long id, String udtNo) {
        List<UserAccountEntity> existUserAccountList = userMapper.selectUserAccountByUserNo(id);
        if (!existUserAccountList.isEmpty()) {
            LocalDateTime timeNow = LocalDateTime.now();
            for(UserAccountEntity userAccount: existUserAccountList){
                UpdateUserAccountDto updateUserAccountDto = UpdateUserAccountDto.builder()
                        .accNo(userAccount.getAccNo())
                        .isDlt("Y")
                        .udtNo(udtNo)
                        .udtDtm(timeNow)
                        .build();
                userMapper.updateUserAccount(updateUserAccountDto);

                UserAccountHistoryEntity userAccountHistoryEntity = UserAccountHistoryEntity.builder()
                        .usrNo(id)
                        .acctCcd(userAccount.getAcctCcd())
                        .bkNm(userAccount.getBkNm())
                        .acctNo(userAccount.getAcctNo())
                        .isDlt("Y")
                        .regNo(userAccount.getRegNo())
                        .regDtm(userAccount.getRegDtm())
                        .udtNo(udtNo)
                        .udtDtm(timeNow)
                        .build();
                userMapper.insertUserAccountHistory(userAccountHistoryEntity);
            }
            return true;
        }
        return false;
    }

    @Override
    public Boolean deleteUserAccount(Long id) {
        List<UserAccountEntity> userAccountList = userMapper.selectUserAccountByUserNo(id);
        if (!userAccountList.isEmpty() && userAccountList.get(0).getIsDlt().equals("Y")) {
            userMapper.deleteUserAccount(id);
            return true;
        }
        return false;
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
    int updateUserAccount(UpdateUserAccountDto updateUserAccountDto);
    int insertUserAccountHistory(UserAccountHistoryEntity userAccountHistoryEntity);
    int deleteUserAccount(Long id);
}
```

- UserMapper.xml   

```
<mapper namespace="com.education3.education3.mapper.UserMapper">

    ...
    <!--  추가코드  -->
    <!-- 사용자계좌정보 조회 -->
    <select id="selectUserAccountByUserNo">
        SELECT *
        FROM   TM_USR_ACC
        WHERE  USR_NO = #{usrNo}
    </select>

    <!-- 사용자삭제 시, 사용여부 변경("N" -> "Y") -->
    <update id="updateUserAccount">
        UPDATE TM_USR_ACC
        SET    IS_DLT  = #{isDlt}, /* 사용여부 */
               UDT_NO  = #{udtNo}, /* 수정자번호 */
               UDT_DTM = #{udtDtm} /* 수정날짜 */
        WHERE  ACC_NO  = #{accNo}  /* 유저번호 */
    </update>

    <!-- 사용자계좌정보 히스토리 생성 -->
    <insert id="insertUserAccountHistory">
        INSERT INTO TM_USR_ACC(USR_NO,  /* 유저번호 */
                               ACCT_CCD,/* 계좌분류코드 */
                               BK_NM,   /* 은행명 */
                               ACCT_NO, /* 계좌번호 */
                               IS_DLT,  /* 사용여부 */
                               REG_NO,  /* 등록자번호 */
                               REG_DTM, /* 등록날짜 */
                               UDT_NO,  /* 수정자번호 */
                               UDT_DTM) /* 수정날짜 */
               VALUES(#{usrNo},
                      #{acctCcd},
                      #{bkNm},
                      #{acctNo},
                      #{isDlt},
                      #{regNo},
                      #{regDtm},
                      #{udtNo},
                      #{udtDtm})

    </insert>

    <!-- 사용자계좌정보 삭제 -->
    <delete id="deleteUserAccount">
        DELETE
        FROM   TM_USR_ACC
        WHERE  USR_NO = #{usrNo}
    </delete>
</mapper>
```

### **Controller**   

조회API를 제외하고는 가입과 탈퇴 및 삭제시 회원계좌정보 service도 같이 처리되도록 코드를 수정함.   

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
            Boolean resultAcc = userService.updateUserAccount(id, udtNo);
            if (resultCont || resultAcc) {
                return "Success update user";
            } else if (resultCont) {
                return "Not success update user Account";
            } else {
                return "Not success update user Contact";
            }
        }
        return "Not success update User";
    }

    @DeleteMapping("/v1/user/{id}")
    public String deleteUser(@PathVariable Long id){
        Boolean resultCont = userService.deleteUserContact(id);
        Boolean resultAcct = userService.deleteUserAccount(id);
        if (resultCont || resultAcct) {
            Boolean result = userService.deleteUser(id);
            if (result) {
                return "Success delete user";
            }
            return "Success delete user contact but Not success user";
        } else if(resultCont) {
            return "Success delete user contact but Not success user account";
        } else {
            return "Not success delete user";
        }
    }
}
```

### **테이블, 레코드 생성 코드**

- schema.sql   

```
--추가코드
DROP TABLE IF EXISTS tm_usr_acc CASCADE;
DROP TABLE IF EXISTS th_usr_acc CASCADE;
DROP TABLE IF EXISTS tm_cmm_cmm CASCADE;
DROP TABLE IF EXISTS th_cmm_cmm CASCADE;

--유저계좌정보 테이블
CREATE TABLE tm_usr_acc (
        acc_no bigint AUTO_INCREMENT,
        usr_no bigint,         --사용자 번호
        acct_ccd varchar(5),   --계좌구분코드
        bk_nm varchar(50),     --계좌은행명
        acct_no varchar(20),   --사용자 계좌번호
        is_dlt varchar(5),     --사용여부(Y,N)
        reg_no varchar(20),    --등록자 번호
        reg_dtm timestamp,     --등록날짜
        udt_no varchar(20),    --수정자 번호
        udt_dtm timestamp,     --수정날짜
        primary key (acc_no, usr_no)
    );

CREATE TABLE th_usr_acc (
        acc_no bigint AUTO_INCREMENT,
        usr_no bigint,         --사용자 번호
        acct_ccd varchar(5),   --계좌구분코드
        bk_nm varchar(50),     --계좌은행명
        acct_no varchar(20),   --사용자 계좌번호
        is_dlt varchar(5),     --사용여부(Y,N)
        reg_no varchar(20),    --등록자 번호
        reg_dtm timestamp,     --등록날짜
        udt_no varchar(20),    --수정자 번호
        udt_dtm timestamp,     --수정날짜
        primary key (acc_no, usr_no)
    );

--은행공통정보 테이블
CREATE TABLE tm_cmm_cmm (
        cmm_no bigint AUTO_INCREMENT,
        dv1 varchar(50),    --대분류(은행, 증권)
        dv2 varchar(50),    --중분류(1금융권, 2금융권)
        dv3 varchar(50),    --소분류(농협,우리은행 등 은행명)
        cd varchar(5),      --분류코드
        etc varchar(255),   --비고
        is_dlt varchar(5),  --사용여부(Y,N)
        reg_no varchar(20), --등록자 번호
        reg_dtm timestamp,  --등록날짜
        udt_no varchar(20), --수정자 번호
        udt_dtm timestamp,  --수정날짜
        primary key (cmm_no)
    );

CREATE TABLE th_cmm_cmm (
        cmm_no bigint AUTO_INCREMENT,
        dv1 varchar(50),    --대분류(은행, 증권)
        dv2 varchar(50),    --중분류(1금융권, 2금융권)
        dv3 varchar(50),    --소분류(농협,우리은행 등 은행명)
        cd varchar(5),      --분류코드
        etc varchar(255),   --비고
        is_dlt varchar(5),  --사용여부(Y,N)
        reg_no varchar(20), --등록자 번호
        reg_dtm timestamp,  --등록날짜
        udt_no varchar(20), --수정자 번호
        udt_dtm timestamp,  --수정날짜
        primary key (cmm_no)
    );
```

- data.sql
```
--추가코드
--은행정보
INSERT INTO TM_CMM_CMM(DV1,
                       DV2,
                       DV3,
                       CD,
                       ETC,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('은행',
              '1금융권',
              '농협은행',
              '001',
              '',
              'N',
              '000',
              '2023-07-07 17:35:02.462186',
              '000',
              '2023-07-07 17:35:02.462186');

INSERT INTO TM_CMM_CMM(DV1,
                       DV2,
                       DV3,
                       CD,
                       ETC,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('은행',
              '1금융권',
              '우리은행',
              '002',
              '',
              'N',
              '000',
              '2023-07-07 17:35:02.462186',
              '000',
              '2023-07-07 17:35:02.462186');

INSERT INTO TM_CMM_CMM(DV1,
                       DV2,
                       DV3,
                       CD,
                       ETC,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('은행',
              '2금융권',
              '새마을금고',
              '003',
              '',
              'N',
              '000',
              '2023-07-07 17:35:02.462186',
              '000',
              '2023-07-07 17:35:02.462186');

INSERT INTO TH_CMM_CMM(DV1,
                       DV2,
                       DV3,
                       CD,
                       ETC,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('은행',
              '1금융권',
              '농협은행',
              '001',
              '',
              'N',
              '000',
              '2023-07-07 17:35:02.462186',
              '000',
              '2023-07-07 17:35:02.462186');

INSERT INTO TH_CMM_CMM(DV1,
                       DV2,
                       DV3,
                       CD,
                       ETC,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('은행',
              '1금융권',
              '우리은행',
              '002',
              '',
              'N',
              '000',
              '2023-07-07 17:35:02.462186',
              '000',
              '2023-07-07 17:35:02.462186');

INSERT INTO TH_CMM_CMM(DV1,
                       DV2,
                       DV3,
                       CD,
                       ETC,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('은행',
              '2금융권',
              '새마을금고',
              '003',
              '',
              'N',
              '000',
              '2023-07-07 17:35:02.462186',
              '000',
              '2023-07-07 17:35:02.462186');


--유저계좌정보
INSERT INTO TM_USR_ACC(USR_NO,
                       ACCT_CCD,
                       BK_NM,
                       ACCT_NO,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('2',
              '003',
              '새마을금고',
              '251235795',
              'N',
              '001',
              '2023-07-07 17:35:02.462186',
              '001',
              '2023-07-07 17:35:02.462186');

INSERT INTO TH_USR_ACC(USR_NO,
                       ACCT_CCD,
                       BK_NM,
                       ACCT_NO,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('2',
              '003',
              '새마을금고',
              '251235795',
              'N',
              '001',
              '2023-07-07 17:35:02.462186',
              '001',
              '2023-07-07 17:35:02.462186');

INSERT INTO TM_USR_ACC(USR_NO,
                       ACCT_CCD,
                       BK_NM,
                       ACCT_NO,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('1',
              '001',
              '농협은행',
              '1258638792',
              'N',
              '001',
              '2023-07-07 17:35:02.462186',
              '001',
              '2023-07-07 17:35:02.462186');

INSERT INTO TM_USR_ACC(USR_NO,
                       ACCT_CCD,
                       BK_NM,
                       ACCT_NO,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('1',
              '002',
              '우리은행',
              '1258638792',
              'N',
              '001',
              '2023-07-07 17:35:02.462186',
              '001',
              '2023-07-07 17:35:02.462186');

INSERT INTO TH_USR_ACC(USR_NO,
                       ACCT_CCD,
                       BK_NM,
                       ACCT_NO,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('1',
              '001',
              '농협은행',
              '1258638792',
              'N',
              '001',
              '2023-07-07 17:35:02.462186',
              '001',
              '2023-07-07 17:35:02.462186');

INSERT INTO TM_USR_ACC(USR_NO,
                       ACCT_CCD,
                       BK_NM,
                       ACCT_NO,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('1',
              '001',
              '농협은행',
              '12598735324',
              'N',
              '001',
              '2023-07-07 17:35:02.462186',
              '001',
              '2023-07-07 17:35:02.462186');

INSERT INTO TH_USR_ACC(USR_NO,
                       ACCT_CCD,
                       BK_NM,
                       ACCT_NO,
                       IS_DLT,
                       REG_NO,
                       REG_DTM,
                       UDT_NO,
                       UDT_DTM)
       VALUES('1',
              '001',
              '농협은행',
              '12598735324',
              'N',
              '001',
              '2023-07-07 17:35:02.462186',
              '001',
              '2023-07-07 17:35:02.462186')
```

## 실행결과

### **유저 수정**
- method: put   
- url: localhost:8085/v1/user/2   

**[실행 화면]**   
![api-project3-1](/assets/images/api-project3-1.png)   

**[결과]**   
![api-project3-2](/assets/images/api-project3-2.png)   

### **유저 삭제**
- method: delete   
- url: localhost:8085/v1/user/2   

**[실행 화면]**   
![api-project3-3](/assets/images/api-project3-3.png)   

**[결과]**   
![api-project3-4](/assets/images/api-project3-4.png)   