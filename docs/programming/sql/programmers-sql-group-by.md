---
title: 프로그래머스 코딩테스트(sql) - GROUP BY
layout: default
parent: Sql
grand_parent: Programming
---

## Level2

### [성분으로 구분한 아이스크림 총 주문량]   

```
SELECT B.INGREDIENT_TYPE, SUM(TOTAL_ORDER) AS TOTAL_ORDER
FROM   FIRST_HALF    A,
       ICECREAM_INFO B
WHERE  A.FLAVOR = B.FLAVOR
GROUP  BY B.INGREDIENT_TYPE;
```

***

### [자동차 종류 별 특정 옵션이 포함된 자동차 수 구하기]   

```
SELECT CAR_TYPE, COUNT(OPTIONS) AS CARS
FROM   CAR_RENTAL_COMPANY_CAR
WHERE  OPTIONS LIKE '%통풍시트%'
       OR OPTIONS LIKE '%열선시트%'
       OR OPTIONS LIKE '%가죽시트%'
GROUP  BY CAR_TYPE
ORDER  BY CAR_TYPE;
```

- '통풍시트', '열선시트', '가죽시트' 중 하나 이상의 옵션이 포함된 조건을 만족시키기 위해
WHERE LIKE를 이용하여 하나씩 조건을 써주었는데 문제야 3개여서 많이 복잡하지 않고 금방 했다고 하지만
조건이 더 많아지면 코드도 길어지고 복잡해질 것 같아서 짧게 줄일 수 있는 방법을 찾다가 [정규식]을 이용하면
LIKE보다 간단하게 결과를 도출할 수 있다.

```
SELECT CAR_TYPE, COUNT(OPTIONS) AS CARS
FROM   CAR_RENTAL_COMPANY_CAR
WHERE  REGEXP_LIKE (OPTIONS, '통풍시트|열선시트|가죽시트')
GROUP  BY CAR_TYPE
ORDER  BY CAR_TYPE;
```

***

### [진료과별 총 예약 횟수 출력하기]   

- 컬럼명의 alias가 한글이어서 뭔가 당연하게 따옴표('', "")를 사용해서 컬럼명을 써야한다고 생각해서 썼더니
아무리 봐도 틀린게 없는데 틀렸다고 나왔다.. 컬럼명은 영어든 한글이든 따옴표 없이 사용해야 한다.

```
SELECT MCDP_CD 진료과코드,
       COUNT(PT_NO) 5월예약건수
FROM   APPOINTMENT
WHERE  DATE_FORMAT(APNT_YMD, '%Y-%m') = '2022-05'
GROUP  BY MCDP_CD
ORDER  BY 5월예약건수, 진료과코드;
```

***

### [가격대 별 상품 개수 구하기]   

- 각 가격에 대한 범위를 설정하기 위해 [CASE WHEN THEN]문을 사용하였다.   
하지만 적으면서도 이렇게 일일히 하는게 비효율적으로 느껴져서 다른 방법을 찾아보았다.   

```
SELECT CASE WHEN (PRICE BETWEEN 0 AND 9000) THEN '0'
            WHEN (PRICE BETWEEN 10000 AND 19000) THEN '10000'
            WHEN (PRICE BETWEEN 10000 AND 19000) THEN '10000'
            WHEN (PRICE BETWEEN 20000 AND 29000) THEN '20000'
            WHEN (PRICE BETWEEN 30000 AND 39000) THEN '30000' 
            WHEN (PRICE BETWEEN 40000 AND 49000) THEN '40000'
            WHEN (PRICE BETWEEN 50000 AND 59000) THEN '50000' 
            WHEN (PRICE BETWEEN 60000 AND 69000) THEN '60000' 
            WHEN (PRICE BETWEEN 70000 AND 79000) THEN '70000' 
            WHEN (PRICE BETWEEN 80000 AND 89000) THEN '80000' END AS PRICE_GROUP,
       COUNT(PRODUCT_ID) AS PRODUCTS
FROM   PRODUCT
GROUP  BY PRICE_GROUP
ORDER  BY PRICE_GROUP;
```

- PRICE를 10000으로 나눈 나머지를 PRICE에서 빼서 PRICE_GROUP을 구함

```
SELECT (PRICE-PRICE%10000) AS PRICE_GROUP, COUNT(*) AS PRODUCTS
FROM PRODUCT 
GROUP BY PRICE_GROUP
ORDER BY PRICE_GROUP
```

- [TRUNCATE]로 아래 4자리를 버림(0으로 바꿈)으로써 PRICE_GROUP 구함

```
SELECT (
       CASE
       WHEN PRICE < 10000 THEN 0
       ELSE TRUNCATE(PRICE, -4)
       END
      ) AS PRICE_GROUP , COUNT(PRODUCT_ID)
FROM  PRODUCT
GROUP BY PRICE_GROUP
ORDER BY PRICE_GROUP;
```

***
***

## Level3

### [즐겨찾기가 가장 많은 식당 정보 출력하기]   

```
SELECT FOOD_TYPE, REST_ID, REST_NAME, FAVORITES
FROM   REST_INFO
WHERE  (FOOD_TYPE, FAVORITES)
       IN (SELECT FOOD_TYPE, MAX(FAVORITES) AS FAVORITES
           FROM   REST_INFO
           GROUP  BY FOOD_TYPE)
ORDER  BY FOOD_TYPE DESC;
```

- 다른 답을 보면 GROUP BY를 한번 더하던데 나와 [같은 의문]을 가진 사람이 있었다.   
답변을 보면 즐겨찾기 수가 같을 경우 같은 음식타입의 결과가 두개 출력되기 때문에 하나로 출력되도록 해주기 위해 하는 듯 싶다.   

***

### [조건에 맞는 사용자와 총 거래금액 조회하기]   

```
SELECT A.WRITER_ID,
       B.NICKNAME,
       SUM(A.PRICE) AS TOTAL_PRICE
FROM   USED_GOODS_BOARD            A
JOIN   USED_GOODS_USER             B
       ON A.WRITER_ID = B.USER_ID
WHERE  A.STATUS = 'DONE'
GROUP  BY A.WRITER_ID
HAVING TOTAL_PRICE >= 700000
ORDER  BY TOTAL_PRICE;
```  

***

### [카테고리 별 도서 판매량 집계하기]   

```
SELECT A.CATEGORY,
       SUM(B.SALES) TOTAL_SALES
FROM   BOOK                    A,
       BOOK_SALES              B
WHERE  A.BOOK_ID = B.BOOK_ID
       AND DATE_FORMAT(B.SALES_DATE, '%Y-%m') = '2022-01'
GROUP  BY A.CATEGORY
ORDER  BY A.CATEGORY;
```  

***

### **[자동차 대여 기록에서 대여중 / 대여 가능 여부 구분하기]   

- 서브쿼리로 START_DATE와 END_DATE 사이에 '2022-10-16'가 해당되는 CAR_ID를 반환하여
IF문으로 CAR_ID가 있다면 "대여중" 없다면 "대여 가능"으로 표시함

```
SELECT CAR_ID, IF (CAR_ID IN (
    SELECT CAR_ID
    FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY
    WHERE '2022-10-16' BETWEEN START_DATE AND END_DATE), '대여중', '대여 가능')AS AVAILABILITY
FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY
GROUP BY CAR_ID
ORDER BY CAR_ID DESC
```  

***

### [대여 횟수가 많은 자동차들의 월별 대여 횟수 구하기]   

- 먼저 대여 시작일을 기준으로 2022년 8월부터 2022년 10월까지 총 대여 횟수가 5회 이상인 자동차들의 CAR_ID를 추출   
- WHERE IN로 해당 CAR_ID의 정보만 추출 후 "해당 기간" 동안의 월별 자동차 ID 별 총 대여 횟수(컬럼명: RECORDS) 출력    
- 특정 월의 총 대여 횟수가 0인 경우에는 결과에서 제외
- 월을 기준으로 오름차순 정렬하고, 월이 같다면 자동차 ID를 기준으로 내림차순 정렬

```
SELECT MONTH(START_DATE) AS MONTH, CAR_ID, COUNT(CAR_ID) AS RECORDS
FROM   CAR_RENTAL_COMPANY_RENTAL_HISTORY
WHERE  CAR_ID IN (SELECT CAR_ID
                  FROM   CAR_RENTAL_COMPANY_RENTAL_HISTORY
                  WHERE  START_DATE BETWEEN '2022-08-01' AND '2022-10-31'
                  GROUP  BY CAR_ID
                  HAVING COUNT(CAR_ID) >= 5)
       AND START_DATE BETWEEN '2022-08-01' AND '2022-10-31'
GROUP  BY MONTH(START_DATE), CAR_ID
HAVING RECORDS > 0
ORDER  BY MONTH(START_DATE), CAR_ID DESC;
```  

***
***

## Level4

### [저자 별 카테고리 별 매출액 집계하기]   

- 테이블 ３개를 조인하여 결과를 추출헤야 하는 문제
  한번에 3개를 조인 후 총판매액을 구하려고 하니 좀 복잡한 듯 싶어 
  먼저 BOOK과 BOOK_SALES 테이블을 조인하여 각 레코드별 총판매액을 구한 후 
  AUTHOR 테이블을 추가로 조인하여 저자별, 카테고리별 총판매액을 더하여 결과를 도출하였다.


```
SELECT D.AUTHOR_ID,
       D.AUTHOR_NAME,
       C.CATEGORY,
       SUM(C.TOTAL_SALES) AS TOTAL_SALES
FROM   (SELECT A.AUTHOR_ID,
               A.CATEGORY,
               (A.PRICE * B.SALES) AS TOTAL_SALES
        FROM   BOOK                                            A
        JOIN   BOOK_SALES                                      B
               ON A.BOOK_ID = B.BOOK_ID
        WHERE  DATE_FORMAT(B.SALES_DATE, '%Y-%m') = '2022-01') C
JOIN   AUTHOR                                                  D
       ON C.AUTHOR_ID = D.AUTHOR_ID
GROUP  BY C.AUTHOR_ID, C.CATEGORY
ORDER  BY C.AUTHOR_ID, C.CATEGORY DESC;
```

- 위의 코드를 짜고 보니 그냥 3개 테이블을 한번에 묶는게 더 깔끔할 듯 싶어 해봤다.
   이게 더 보기 쉬운 것 같기도 하다.

```
SELECT A.AUTHOR_ID,
       C.AUTHOR_NAME,
       A.CATEGORY,
       SUM((A.PRICE * B.SALES)) AS TOTAL_SALES
FROM   BOOK                                            A
JOIN   BOOK_SALES                                      B
       ON A.BOOK_ID = B.BOOK_ID
JOIN   AUTHOR                                          C
       ON A.AUTHOR_ID = C.AUTHOR_ID
WHERE  DATE_FORMAT(B.SALES_DATE, '%Y-%m') = '2022-01'
GROUP  BY A.AUTHOR_ID, A.CATEGORY
ORDER  BY A.AUTHOR_ID, A.CATEGORY DESC;
```

***

### [식품분류별 가장 비싼 식품의 정보 조회하기]   

- WHERE IN으로 식품분류별로 가격이 제일 비싼 식품 추출,   
정규식사용하여 원하는 식품분류의 경우만 추출   

```
SELECT CATEGORY, PRICE AS MAX_PRICE, PRODUCT_NAME
FROM   FOOD_PRODUCT
WHERE  (CATEGORY, PRICE) IN (SELECT CATEGORY,
                                    MAX(PRICE) AS PRICE
                             FROM   FOOD_PRODUCT
                             GROUP  BY CATEGORY)
       AND CATEGORY REGEXP '과자|국|김치|식용유'
ORDER  BY MAX_PRICE DESC;
```

***

### [년, 월, 성별 별 상품 구매 회원 수 구하기]   

- 구매한 회원수를 집계하는 것으로 동일한 회원을 중복 카운트하는 경우를 방지하기 위해 DISTINCT를 이용함

```
SELECT YEAR(B.SALES_DATE) YEAR,
       MONTH(B.SALES_DATE) MONTH,
       A.GENDER,
       COUNT(DISTINCT B.USER_ID) AS USERS
FROM   USER_INFO                          A,
       ONLINE_SALE                        B
WHERE  A.USER_ID = B.USER_ID
       AND NOT A.GENDER IS NULL
GROUP  BY YEAR, MONTH, A.GENDER
ORDER  BY YEAR, MONTH, A.GENDER;
```

[성분으로 구분한 아이스크림 총 주문량]: https://school.programmers.co.kr/learn/courses/30/lessons/133026
[자동차 종류 별 특정 옵션이 포함된 자동차 수 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/151137
[정규식]: https://jhnyang.tistory.com/entry/%EC%98%A4%EB%9D%BC%ED%81%B4-ORACLE-REGEXPLIKE-%EC%82%AC%EC%9A%A9%EB%B2%95-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-LIKE%EC%99%80-IN%EC%9D%84-%ED%95%A9%ED%95%9C-%EA%B2%83%EA%B3%BC-%EA%B0%99%EC%9D%80-%ED%95%A8%EC%88%98-%EC%97%AC%EB%9F%AC-%ED%8C%A8%ED%84%B4%EC%97%90-%EC%86%8D%ED%95%98%EB%8A%94-%EC%A1%B0%EA%B1%B4-%EC%B6%94%EC%B6%9C%ED%95%98%EA%B8%B0-%EB%8B%A4
[진료과별 총 예약 횟수 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/132202
[가격대 별 상품 개수 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131530
[CASE WHEN THEN]: https://info-lab.tistory.com/305
[TRUNCATE]: https://devjhs.tistory.com/87

[즐겨찾기가 가장 많은 식당 정보 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131123
[같은 의문]: https://school.programmers.co.kr/questions/51688
[조건에 맞는 사용자와 총 거래금액 조회하기]: https://school.programmers.co.kr/learn/courses/30/lessons/164668
[카테고리 별 도서 판매량 집계하기]: https://school.programmers.co.kr/learn/courses/30/lessons/144855
[자동차 대여 기록에서 대여중 / 대여 가능 여부 구분하기]: https://school.programmers.co.kr/learn/courses/30/lessons/157340
[대여 횟수가 많은 자동차들의 월별 대여 횟수 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/151139

[저자 별 카테고리 별 매출액 집계하기]: https://school.programmers.co.kr/learn/courses/30/lessons/144856
[식품분류별 가장 비싼 식품의 정보 조회하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131116
[년, 월, 성별 별 상품 구매 회원 수 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131532