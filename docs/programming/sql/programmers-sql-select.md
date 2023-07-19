---
title: 프로그래머스 코딩테스트(sql) - SELECT
layout: default
parent: Sql
grand_parent: Programming
---


## Level1

### [강원도에 위치한 생산공장 목록 출력하기]   

```
SELECT FACTORY_ID, FACTORY_NAME, ADDRESS
FROM   FOOD_FACTORY
WHERE  ADDRESS LIKE '강원도%'
ORDER  BY FACTORY_ID
```

***

### [흉부외과 또는 일반외과 의사 목록 출력하기]   

```
SELECT DR_NAME, DR_ID, MCDP_CD, DATE_FORMAT(HIRE_YMD, '%Y-%m-%d')
FROM   DOCTOR
WHERE  MCDP_CD = 'CS' OR MCDP_CD = 'GS'
ORDER  BY HIRE_YMD DESC, DR_NAME
```

***

    
### [조건에 맞는 도서 리스트 출력하기]   

```
SELECT BOOK_ID, DATE_FORMAT(PUBLISHED_DATE,'%Y-%m-%d')
FROM   BOOK
WHERE  DATE_FORMAT(PUBLISHED_DATE, '%Y') = '2021'
       AND CATEGORY = '인문'
ORDER  BY PUBLISHED_DATE;
```

***


### [과일로 만든 아이스크림 고르기]   

```
SELECT FH.FLAVOR FROM FIRST_HALF FH
JOIN   ICECREAM_INFO II 
       ON FH.FLAVOR = II.FLAVOR 
WHERE  FH.TOTAL_ORDER >= 3000 
       AND II.INGREDIENT_TYPE = 'fruit_based';
```

***

### [12세 이하인 여자 환자 목록 출력하기]   

```
SELECT PT_NAME, PT_NO, GEND_CD, AGE, IFNULL(TLNO, 'NONE')
FROM   PATIENT
WHERE  AGE <= 12
       AND GEND_CD = 'W'
ORDER  BY AGE DESC, PT_NAME;
```

***

### [조건에 부합하는 중고거래 댓글 조회하기]   

```
SELECT UGB.TITLE, UGB.BOARD_ID, UGR.REPLY_ID, UGR.WRITER_ID, UGR.CONTENTS, DATE_FORMAT(UGR.CREATED_DATE, '%Y-%m-%d')
FROM   USED_GOODS_BOARD UGB, USED_GOODS_REPLY UGR
WHERE  UGB.BOARD_ID = UGR.BOARD_ID
       AND DATE_FORMAT(UGB.CREATED_DATE, '%Y-%m') = '2022-10'
ORDER  BY DATE_FORMAT(UGR.CREATED_DATE, '%Y-%m-%d'), UGB.TITLE;
```

***

### [평균 일일 대여 요금 구하기]   

```
SELECT ROUND(AVG(DAILY_FEE), 0) AS AVERAGE_FEE
FROM   CAR_RENTAL_COMPANY_CAR
WHERE  CAR_TYPE = 'SUV'
GROUP  BY CAR_TYPE
```

***

### [인기있는 아이스크림]   

```
SELECT FLAVOR
FROM   FIRST_HALF
ORDER  BY TOTAL_ORDER DESC, SHIPMENT_ID; 
```

***

### [조건에 맞는 회원수 구하기]   

```
SELECT COUNT(*)
FROM   USER_INFO
WHERE  DATE_FORMAT(JOINED, '%Y') = '2021'
       AND AGE BETWEEN 20 AND 29;
```

- 20세 이상 29세 이하 범위의 조건에 맞는 회원을 추출해야 하므로 BETWEEN을 이용함

```
컬럼명 BETWEEN 조건1 AND 조건2
```

***
***

## Level2

### [3월에 태어난 여성 회원 목록 출력하기]   


```
SELECT MEMBER_ID, MEMBER_NAME, GENDER, DATE_FORMAT(DATE_OF_BIRTH, "%Y-%m-%d")
FROM   MEMBER_PROFILE 
WHERE  DATE_FORMAT(DATE_OF_BIRTH, "%m") LIKE "03"
       AND GENDER = "W"
       AND NOT TLNO IS NULL
ORDER  BY MEMBER_ID
```
   
- NULL인 경우 출력에서 제외하는 방법     

```
NOT 컬럼명 IS NULL
```

***

### [재구매가 일어난 상품과 회원 리스트 구하기]   


```
SELECT USER_ID, PRODUCT_ID
FROM   ONLINE_SALE 
GROUP  BY USER_ID, PRODUCT_ID
HAVING COUNT(PRODUCT_ID) > 1
ORDER  BY USER_ID, PRODUCT_ID DESC;
``` 

- GROUP BY로 USER_ID와 PRODUCT_ID가 동일한 컬럼을 묶은 후
  HAVING으로 그룹화 한 결과에서 원하는 조건을 적용시키면 된다.

- [WHERE과 HAVING의 차이점]   
WHERE: 해당 컬럼에서 조건을 만족하는 데이터 필터링   
HAVING: GROUP BY로 집계한 결과에서 조건에 만족하는 데이터 필터링

***
***


## Level4   

### [서울에 위치한 식당 목록 출력하기]   

```
SELECT RI.REST_ID, RI.REST_NAME, RI.FOOD_TYPE, RI.FAVORITES, RI.ADDRESS, RR.SCORE
FROM   REST_INFO RI
JOIN   (SELECT REST_ID, ROUND(AVG(REVIEW_SCORE), 2) AS SCORE 
        FROM   REST_REVIEW 
        GROUP BY REST_ID) RR
ON     RI.REST_ID = RR.REST_ID
WHERE  RI.ADDRESS LIKE '서울%'
ORDER  BY RR.SCORE DESC, RI.FAVORITES DESC;
```

- 리뷰 평균 점수는 식당별로 해야 하므로 GROUP BY를 이용하여 식당별로 묶어주어야 한다  
  또한 JOIN을 할 때 SELECT문으로 원하는 컬럼만 추출하여 이용할 수 있다.     

```
SELECT REST_ID, ROUND(AVG(REVIEW_SCORE), 2) AS SCORE 
FROM   REST_REVIEW 
GROUP  BY REST_ID
```

***

### [오프라인/온라인 판매 데이터 통합하기]   

```
SELECT DATE_FORMAT(SALES_DATE, '%Y-%m-%d') AS SALES_DATE, PRODUCT_ID, USER_ID, SALES_AMOUNT
FROM   ONLINE_SALE
WHERE  DATE_FORMAT(SALES_DATE, '%Y-%m') = '2022-03'
UNION  ALL
SELECT DATE_FORMAT(SALES_DATE, '%Y-%m-%d') AS SALES_DATE, PRODUCT_ID, NULL AS USER_ID, SALES_AMOUNT
FROM   OFFLINE_SALE
WHERE  DATE_FORMAT(SALES_DATE, '%Y-%m') = '2022-03'
ORDER  BY SALES_DATE, PRODUCT_ID, USER_ID;
```

- [UNION]으로 두 개의 SELECT문을 합쳐서 결과를 반환해야 한다   
UNION: 중복되는 행이 있을 경우 중복을 제거하고 하나만 더함   
UNION ALL: 중복과 상관없이 모두 다 더함

- ORDER BY는 UNION ALL한 결과에 대해 정렬해주므로 한번만 입력하면 됨



[강원도에 위치한 생산공장 목록 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131112
[흉부외과 또는 일반외과 의사 목록 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/132203
[조건에 맞는 도서 리스트 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/144853
[과일로 만든 아이스크림 고르기]: https://school.programmers.co.kr/learn/courses/30/lessons/133025
[12세 이하인 여자 환자 목록 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/132201
[조건에 부합하는 중고거래 댓글 조회하기]: https://school.programmers.co.kr/learn/courses/30/lessons/164673
[평균 일일 대여 요금 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/151136
[인기있는 아이스크림]: https://school.programmers.co.kr/learn/courses/30/lessons/133024
[조건에 맞는 회원수 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131535

[3월에 태어난 여성 회원 목록 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131120
[재구매가 일어난 상품과 회원 리스트 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131536
[WHERE과 HAVING의 차이점]: https://kkw-da.tistory.com/29
[UNION]: https://jhnyang.tistory.com/entry/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-SQL-UNION-ALL-%EC%BF%BC%EB%A6%AC-%EA%B2%B0%EA%B3%BC-%EB%8D%94%ED%95%98%EA%B8%B0-select-%EB%8D%94%ED%95%98%EA%B8%B0

[서울에 위치한 식당 목록 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131118
[오프라인/온라인 판매 데이터 통합하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131537