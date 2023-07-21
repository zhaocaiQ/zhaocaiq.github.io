---
title: 프로그래머스 코딩테스트(sql) - JOIN
layout: default
parent: Sql
grand_parent: Programming
---

## Level2

### [조건에 맞는 도서와 저자 리스트 출력하기]    

```
SELECT A.BOOK_ID,
       B.AUTHOR_NAME,
       DATE_FORMAT(A.PUBLISHED_DATE, '%Y-%m-%d')
FROM   BOOK                                     A
JOIN   AUTHOR                                   B
       ON A.AUTHOR_ID = B.AUTHOR_ID
WHERE  A.CATEGORY = '경제'
ORDER  BY A.PUBLISHED_DATE;
```

***

### [상품 별 오프라인 매출 구하기]    

```
SELECT A.PRODUCT_CODE, SUM(A.PRICE * B.SALES_AMOUNT) AS SALES
FROM   PRODUCT      A,
       OFFLINE_SALE B
WHERE  A.PRODUCT_ID = B.PRODUCT_ID
GROUP  BY A.PRODUCT_ID
ORDER  BY SALES DESC, A.PRODUCT_CODE;
```

***
***

## Level4

### [그룹별 조건에 맞는 식당 목록 출력하기]   

- 가장 많이 작성한 회원이 여러명이라 다 출력해야 하는 줄 알았는데 LIMIT으로 한 회원만 출력해도 되는 거였다.

```
SELECT A.MEMBER_NAME, B.REVIEW_TEXT, DATE_FORMAT(B.REVIEW_DATE, '%Y-%m-%d')
FROM   MEMBER_PROFILE  A,
       REST_REVIEW     B
WHERE  A.MEMBER_ID = B.MEMBER_ID
       AND B.MEMBER_ID = (SELECT MEMBER_ID
                          FROM   REST_REVIEW
                          GROUP  BY MEMBER_ID
                          ORDER  BY COUNT(MEMBER_ID) DESC LIMIT 1)
ORDER  BY B.REVIEW_DATE, B.REVIEW_TEXT;
```

- 여러명의 회원을 모두 출력할 수 있는 코드    
- [DENSE_RANK()]를 이용하여 행의 순위를 매겨 순위가 1인 MEMBER_ID의 정보만 출력하도록 함
     - RANK: 공동 순위가 있을 시, 공동 순위를 뛰어넘어 라벨링 함
     - DENSE_RANK: 공동 순위가 있을 시 순위를 뛰어넘지 않고 라벨링 함.

```
SELECT A.MEMBER_NAME, B.REVIEW_TEXT, 
       DATE_FORMAT(B.REVIEW_DATE, '%Y-%m-%d') AS REVIEW_DATE
FROM   MEMBER_PROFILE                                       A
JOIN   REST_REVIEW                                          B
       ON A.MEMBER_ID = B.MEMBER_ID
WHERE  A.MEMBER_ID IN 
       (SELECT MEMBER_ID
        FROM (SELECT MEMBER_ID,
                    COUNT(MEMBER_ID),
                    REVIEW_TEXT,
                    REVIEW_DATE, 
                    DENSE_RANK () OVER 
                    (ORDER BY COUNT(MEMBER_ID) DESC) AS RK
             FROM REST_REVIEW
             GROUP BY 1) C
        WHERE RK = 1)
ORDER BY B.REVIEW_DATE, B.REVIEW_TEXT
```

***

### [5월 식품들의 총매출 조회하기]   


```
SELECT A.PRODUCT_ID,
       A.PRODUCT_NAME,
       SUM((A.PRICE * B.AMOUNT)) AS TOTAL_SALES
FROM   FOOD_PRODUCT                             A
JOIN   FOOD_ORDER                               B
       ON A.PRODUCT_ID = B.PRODUCT_ID
WHERE  DATE_FORMAT(B.PRODUCE_DATE, '%Y-%m') = '2022-05'
GROUP  BY A.PRODUCT_ID
ORDER  BY TOTAL_SALES DESC, A.PRODUCT_ID;
```

***

### [주문량이 많은 아이스크림들 조회하기]   

```
SELECT A.FLAVOR
FROM   FIRST_HALF         A,
       JULY               B
WHERE  A.FLAVOR = B.FLAVOR
GROUP  BY A.FLAVOR
ORDER  BY (SUM(A.TOTAL_ORDER) + SUM(B.TOTAL_ORDER)) DESC LIMIT 3;
```

***

### [특정 기간동안 대여 가능한 자동차들의 대여비용 구하기]   

- 자동차 종류가 '세단' 또는 'SUV' 인 자동차
- 2022년 11월 1일부터 2022년 11월 30일까지 대여 가능
- 대여 금액이 50만원 이상 200만원 미만인 자동차
     - 30일 이상 대여시, 할인율 적용 필요
- 자동차 ID, 자동차 종류, 대여 금액(컬럼명: FEE) 리스트를 출력
- 정렬
     - 대여 금액을 기준으로 내림차순 정렬
     - 대여 금액이 같은 경우 자동차 종류를 기준으로 오름차순 정렬
     - 자동차 종류까지 같은 경우 자동차 ID를 기준으로 내림차순 정렬

```
SELECT A.CAR_ID, 
       A.CAR_TYPE,
       ROUND((A.DAILY_FEE * (100 - B.DISCOUNT_RATE)/100)*30) AS FEE
FROM   CAR_RENTAL_COMPANY_CAR A
JOIN   CAR_RENTAL_COMPANY_DISCOUNT_PLAN B 
ON     A.CAR_TYPE = B.CAR_TYPE
WHERE  A.CAR_TYPE REGEXP '세단|SUV' AND
       B.DURATION_TYPE = '30일 이상' AND
       A.CAR_ID NOT IN (SELECT CAR_ID
                        FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY
                        WHERE END_DATE >= '2022-11-01' AND START_DATE <= '2022-11-30')
HAVING FEE >= 500000 AND FEE < 2000000
ORDER  BY FEE DESC, 
          A.CAR_TYPE, 
          A.CAR_ID DESC;
```

***
***

## Level5

### [상품을 구매한 회원 비율 구하기]   

- 다른 조건을 만족하는 코드는 쉽게 작성했는데 21년에 가입한 전체 회원 수를 JOIN하는냐에 많은 시간을 소비했다.   
결국 혼자서는 해결하지 못해 답변을 찾아봤는데.. 그냥 서브쿼리로 SELECT를 하면 됐던거였다.   
sql문제를 풀면서 서브쿼리나 UNION 등 필요한 데이터를 추가로 접목시키는 것이 많이 부족한 부분인 것 같다.   
계속하면서 이런 부분에 익숙해지도록 연습을 더 많이 해야할 것 같다.   

- 2021년에 가입한 전체 회원들 중 상품을 구매한 회원수
- 상품을 구매한 회원의 비율(2021년에 가입한 회원 중 상품을 구매한 회원수 / 2021년에 가입한 전체 회원 수)
- 년, 월 별로 출력
- 상품을 구매한 회원의 비율은 소수점 두번째자리에서 반올림
- 년을 기준으로 오름차순 정렬해주시고 년이 같다면 월을 기준으로 오름차순 정렬

```
SELECT YEAR(B.SALES_DATE) YEARS,
       MONTH(B.SALES_DATE) MONTHS,
       COUNT(DISTINCT A.USER_ID) PUCHASED_USERS,
       ROUND(COUNT(DISTINCT A.USER_ID) / (SELECT COUNT(*)
                                          FROM   USER_INFO
                                          WHERE  YEAR(JOINED) = '2021'), 1) PUCHASED_RATIO
FROM   USER_INFO               A
JOIN   ONLINE_SALE             B
ON     A.USER_ID = B.USER_ID
WHERE  YEAR(A.JOINED) = '2021'
GROUP  BY YEARS, MONTHS;
```

[조건에 맞는 도서와 저자 리스트 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/144854
[상품 별 오프라인 매출 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131533

[그룹별 조건에 맞는 식당 목록 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131124
[DENSE_RANK()]: https://velog.io/@ljs7463/MySQL-%ED%96%89%EC%9D%98-%EC%88%9C%EC%9C%84%EB%A5%BC-%EB%A7%8C%EB%93%9C%EB%8A%94-%ED%95%A8%EC%88%98-RANK-DENSERANK-ROWNUMBER
[5월 식품들의 총매출 조회하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131117
[주문량이 많은 아이스크림들 조회하기]: https://school.programmers.co.kr/learn/courses/30/lessons/133027
[특정 기간동안 대여 가능한 자동차들의 대여비용 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/157339

[상품을 구매한 회원 비율 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131534