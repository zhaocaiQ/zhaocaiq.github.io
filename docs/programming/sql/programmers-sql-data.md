---
title: 프로그래머스 코딩테스트(sql) - String, Data
layout: default
parent: Sql
grand_parent: Programming
---
## Level1

### [특정 옵션이 포함된 자동차 리스트 구하기]   

- '네비게이션' 옵션이 포함된 자동차 리스트를 출력
- 자동차 ID를 기준으로 내림차순 정렬

```
SELECT *
FROM   CAR_RENTAL_COMPANY_CAR
WHERE  OPTIONS LIKE '%네비게이션%'
ORDER  BY CAR_ID DESC;
```

***

### [자동차 대여 기록에서 장기/단기 대여 구분하기]   

- 대여 시작일이 2022년 9월에 속하는 대여 기록
- 대여 기간이 30일 이상이면 '장기 대여' 그렇지 않으면 '단기 대여' 로 표시(컬럼명: RENT_TYPE)
- 대여 기록 ID를 기준으로 내림차순 정렬    

```
SELECT HISTORY_ID,
       CAR_ID,
       DATE_FORMAT(START_DATE, '%Y-%m-%d') START_DATE,
       DATE_FORMAT(END_DATE, '%Y-%m-%d') END_DATE,
       IF(DATEDIFF(END_DATE, START_DATE)+1 >= 30, '장기 대여', '단기 대여') RENT_TYPE
FROM   CAR_RENTAL_COMPANY_RENTAL_HISTORY
WHERE  DATE_FORMAT(START_DATE, '%Y-%m') = '2022-09'
ORDER  BY HISTORY_ID DESC;
```

***
***

## Level2

### [자동차 평균 대여 기간 구하기]   

- 평균 대여 기간이 7일 이상인 자동차들의 자동차 ID   
- 평균 대여 기간(컬럼명: AVERAGE_DURATION) 리스트   
- 평균 대여 기간은 소수점 두번째 자리에서 반올림    
- 평균 대여 기간을 기준으로 내림차순 정렬   
- 평균 대여 기간이 같으면 자동차 ID를 기준으로 내림차순 정렬   

**START_DATE와 END_DATE가 동일하다면 DATEDIFF했을 때, 0일이 나오게 된다.
그러므로 "+1"을 해줘야 정확한 대여기간이 나오게 된다.   

```
SELECT CAR_ID, ROUND(AVG(DATEDIFF(END_DATE, START_DATE))+1, 1) AVERAGE_DURATION
FROM   CAR_RENTAL_COMPANY_RENTAL_HISTORY
GROUP  BY CAR_ID
HAVING AVERAGE_DURATION >= 7
ORDER  BY AVERAGE_DURATION DESC, CAR_ID DESC;
```

***

### [조건에 부합하는 중고거래 상태 조회하기]   

- 2022년 10월 5일에 등록된 중고거래 게시물   
- 게시글 ID, 작성자 ID, 게시글 제목, 가격, 거래상태 조회   
- 거래상태가 SALE 이면 판매중, RESERVED이면 예약중, DONE이면 거래완료 분류하여 출력   
- 게시글 ID를 기준으로 내림차순 정렬

```
SELECT BOARD_ID, WRITER_ID, TITLE, PRICE, CASE WHEN STATUS = 'SALE'     THEN '판매중'
                                               WHEN STATUS = 'RESERVED' THEN '예약중'
                                               WHEN STATUS = 'DONE'     THEN '거래완료' END STATUS
FROM   USED_GOODS_BOARD
WHERE  DATE_FORMAT(CREATED_DATE, '%Y-%m-%d') = '2022-10-05'
ORDER  BY BOARD_ID DESC;
```

***

### [카테고리 별 상품 개수 구하기]   

- 상품 카테고리 코드(PRODUCT_CODE 앞 2자리) 별 상품 개수를 출력
- 상품 카테고리 코드를 기준으로 오름차순 정렬

```
SELECT SUBSTR(PRODUCT_CODE, 1, 2) CATEGORY,
       COUNT(PRODUCT_ID) PRODUCTS
FROM   PRODUCT
GROUP  BY CATEGORY
ORDER  BY CATEGORY;

```

***
***

## Level3

### [대여 기록이 존재하는 자동차 리스트 구하기]   

- 자동차 종류가 '세단'인 자동차   
- 10월에 대여를 시작한 기록이 있는 자동차 ID 리스트   
- 자동차 ID 리스트는 중복X   
- 자동차 ID를 기준으로 내림차순 정렬   

```
SELECT DISTINCT A.CAR_ID
FROM   CAR_RENTAL_COMPANY_CAR            A
JOIN   CAR_RENTAL_COMPANY_RENTAL_HISTORY B
ON     A.CAR_ID = B.CAR_ID
WHERE  A.CAR_TYPE = '세단'
       AND DATE_FORMAT(START_DATE, '%m') = '10'
ORDER  BY A.CAR_ID DESC;
```

***

### [조건별로 분류하여 주문상태 출력하기]   

- 5월 1일 기준으로 이전 날짜는 출고완료로 이 후 날짜는 출고 대기로 미정이면 출고미정으로 출고여부 출력
- 주문 ID, 제품 ID, 출고일자, 출고여부 조회
- 주문 ID를 기준으로 오름차순 정렬

** IF문(CASE WHEN와 비슷)
```
IF(조건, 참일 때 출력값, 거짓일 때 출력값)
```

```
SELECT ORDER_ID,
       PRODUCT_ID,
       DATE_FORMAT(OUT_DATE, '%Y-%m-%d'),
       IF(ISNULL(OUT_DATE), '출고미정', IF(OUT_DATE > '2022-05-01', '출고대기', '출고완료')) 출고여부
FROM   FOOD_ORDER
ORDER  BY ORDER_ID;
```

***

### [조회수가 가장 많은 중고거래 게시판의 첨부파일 조회하기]   

- 조회수가 가장 높은 중고거래 게시물
- 파일경로는 "/home/grep/src/ + 게시글ID + 파일ID + 파일이름 + 파일 확장자"로 구성
- 첨부파일 경로는 FILE ID를 기준으로 내림차순 정렬
- 조회수가 가장 높은 게시물은 하나만 존재

```
SELECT CONCAT('/home/grep/src/',
              B.BOARD_ID, '/',
              B.FILE_ID,
              B.FILE_NAME,
              B.FILE_EXT) FILE_PATH
FROM   USED_GOODS_BOARD                    A
JOIN   USED_GOODS_FILE                     B
ON     A.BOARD_ID = B.BOARD_ID
WHERE  A.VIEWS = (SELECT MAX(VIEWS)
                  FROM   USED_GOODS_BOARD)
ORDER  BY B.FILE_ID DESC;
```

***

### [조건에 맞는 사용자 정보 조회하기]   

- 중고 거래 게시물을 3건 이상 등록한 사용자
- 사용자 ID, 닉네임, 전체주소, 전화번호를 조회
- 전체 주소: 시 + 도로명 주소 + 상세 주소
- 전화번호의: xxx-xxxx-xxxx 형태로 출력(하이픈 문자열(-)을 삽입)
- 회원 ID를 기준으로 내림차순 정렬   


```
SELECT B.USER_ID,
       B.NICKNAME,
       CONCAT(B.CITY, ' ',
              B.STREET_ADDRESS1, ' ',
              B.STREET_ADDRESS2) 전체주소,
       CONCAT(LEFT(B.TLNO, 3), '-',
              MID(B.TLNO, 4, 4), '-',
              RIGHT(B.TLNO, 4)) 전화번호
FROM   USED_GOODS_BOARD                  A
JOIN   USED_GOODS_USER                   B
ON     A.WRITER_ID=B.USER_ID
GROUP  BY B.USER_ID
HAVING COUNT(B.USER_ID) >= 3
ORDER  BY B.USER_ID DESC;
```

***
***

## Level4

### [취소되지 않은 진료 예약 조회하기]   

- 2022년 4월 13일 취소되지 않은 흉부외과(CS) 진료 예약 내역을 조회
- 진료예약번호, 환자이름, 환자번호, 진료과코드, 의사이름, 진료예약일시 조회
- 진료예약일시를 기준으로 오름차순 정렬

```
SELECT A.APNT_NO,
       C.PT_NAME,
       C.PT_NO,
       A.MCDP_CD,
       B.DR_NAME,
       A.APNT_YMD
FROM   APPOINTMENT        A
JOIN   DOCTOR             B
ON     A.MDDR_ID = B.DR_ID
JOIN   PATIENT            C
ON     A.PT_NO = C.PT_NO
WHERE  DATE_FORMAT(A.APNT_YMD, '%Y-%m-%d') = '2022-04-13'
       AND A.APNT_CNCL_YN = "N"
       AND A.MCDP_CD = 'CS'
ORDER  BY A.APNT_YMD;
```

***

### [자동차 대여 기록 별 대여 금액 구하기]   

- 자동차 종류가 '트럭'인 자동차
- 대여 기록 별로 대여 금액(컬럼명: FEE)을 구하여 대여 기록 ID와 대여 금액 리스트를 출력
- 대여 금액을 기준으로 내림차순 정렬
- 대여 금액이 같은 경우 대여 기록 ID를 기준으로 내림차순 정렬

** CAR_RENTAL_COMPANY_RENTAL_HISTORY테이블에서 대여 기간별 DURATION_TYPE 컬럼을 생성하여 CAR_RENTAL_COMPANY_DISCOUNT_PLAN테이블과 JOIN 시킴.   
** LEFT JOIN으로 B테이블에 있는 레코드에 겹치는 부분만 JOIN 시킴.
![join-img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99219C345BE91A7E32)    


```
SELECT B.HISTORY_ID, IF(ISNULL(C.DISCOUNT_RATE),
                        B.DURATION*A.DAILY_FEE,
                        ROUND(A.DAILY_FEE*((100-C.DISCOUNT_RATE)/100))*B.DURATION) FEE
FROM   CAR_RENTAL_COMPANY_CAR                                                           A,
       (SELECT CAR_ID,
               HISTORY_ID,
               DATEDIFF(END_DATE, START_DATE) + 1 as DURATION,
               CASE
                    WHEN DATEDIFF(END_DATE, START_DATE) + 1 >= 90 THEN "90일 이상"
                    WHEN DATEDIFF(END_DATE, START_DATE) + 1 >= 30 THEN "30일 이상"
                    WHEN DATEDIFF(END_DATE, START_DATE) + 1 >= 7 THEN "7일 이상"
                END DURATION_TYPE
        FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY)                                        B
LEFT   JOIN   (SELECT DURATION_TYPE, DISCOUNT_RATE
               FROM   CAR_RENTAL_COMPANY_DISCOUNT_PLAN
               WHERE  CAR_TYPE = '트럭')                                                C
ON     B.DURATION_TYPE = C.DURATION_TYPE
WHERE  A.CAR_ID=B.CAR_ID
       AND A.CAR_TYPE = '트럭'
ORDER  BY FEE DESC, B.HISTORY_ID DESC;
```

[특정 옵션이 포함된 자동차 리스트 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/157343
[자동차 대여 기록에서 장기/단기 대여 구분하기]: https://school.programmers.co.kr/learn/courses/30/lessons/151138

[자동차 평균 대여 기간 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/157342
[조건에 부합하는 중고거래 상태 조회하기]: https://school.programmers.co.kr/learn/courses/30/lessons/164672
[카테고리 별 상품 개수 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131529

[대여 기록이 존재하는 자동차 리스트 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/157341
[조건별로 분류하여 주문상태 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131113
[조회수가 가장 많은 중고거래 게시판의 첨부파일 조회하기]: https://school.programmers.co.kr/learn/courses/30/lessons/164671
[조건에 맞는 사용자 정보 조회하기]: https://school.programmers.co.kr/learn/courses/30/lessons/164670

[취소되지 않은 진료 예약 조회하기]: https://school.programmers.co.kr/learn/courses/30/lessons/132204
[자동차 대여 기록 별 대여 금액 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/151141