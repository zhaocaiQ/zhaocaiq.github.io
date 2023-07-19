---
title: 프로그래머스 코딩테스트(sql) - SUM, MAX, MIN
layout: default
parent: Sql
grand_parent: Programming
---


## Level1

### [가장 비싼 상품 구하기]   

```
SELECT MAX(PRICE) AS MAX_PRICE
FROM   PRODUCT;
```

***
***

## Level2

### [가격이 제일 비싼 식품의 정보 출력하기]   

```
SELECT PRODUCT_ID,
       PRODUCT_NAME,
       PRODUCT_CD,
       CATEGORY,
       PRICE
FROM   FOOD_PRODUCT
WHERE  PRICE = (SELECT MAX(PRICE) FROM FOOD_PRODUCT);
```

{: .warning }
Invalid use of group function MAX

그룹함수가 허용되지 않는 위치에서 그룹 함수를 사용하여 발생하는 에러    
** 그룹함수: COUNT, MAX, MIN, SUM ... **   

[오류]    
```
SELECT PRODUCT_ID,
       PRODUCT_NAME,
       PRODUCT_CD,
       CATEGORY,
       PRICE
FROM   FOOD_PRODUCT
WHERE  PRICE = MAX(PRICE);
```

[해결]   

SELECT문을 이용하여 해당 컬럼값을 불러와서 이용

```
SELECT PRODUCT_ID,
       PRODUCT_NAME,
       PRODUCT_CD,
       CATEGORY,
       PRICE
FROM   FOOD_PRODUCT
WHERE  PRICE = (SELECT MAX(PRICE) FROM FOOD_PRODUCT);
```

[가장 비싼 상품 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131697
[가격이 제일 비싼 식품의 정보 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131115