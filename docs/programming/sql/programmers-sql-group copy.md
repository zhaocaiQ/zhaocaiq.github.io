---
title: 프로그래머스 코딩테스트(sql) - IS NULL
layout: default
parent: Sql
grand_parent: Programming
---


## Level1

### [경기도에 위치한 식품창고 목록 출력하기]   

```
SELECT WAREHOUSE_ID,
       WAREHOUSE_NAME,
       ADDRESS,
       IFNULL(FREEZER_YN, 'N')
FROM   FOOD_WAREHOUSE
WHERE  ADDRESS LIKE '경기도%'
ORDER  BY WAREHOUSE_ID;
```

***

### [나이 정보가 없는 회원 수 구하기]   

```
SELECT COUNT(*)
FROM   USER_INFO
WHERE  AGE IS NULL;
```

***
***

## Level2

### [NULL 처리하기]   

- 이전에 풀었던 것 중 어려운 것만 다시 풀어보려고 하는데 이 문제는 옛날에 CASE WHEN을 써서 어렵게 푼 걸 쉽게 풀 수 있을 것 같아 풀어봤다.

```
SELECT ANIMAL_TYPE,
       CASE WHEN NAME IS NULL 
            THEN "No name"
            ELSE NAME END
       SEX_UPON_INTAKE
FROM   ANIMAL_INS;
```

- 새로운 풀이   
IFNULL() 함수를 이용하면 손쉽게 해결 가능한 문제였다.

```
SELECT ANIMAL_TYPE,
       IFNULL(NAME, 'No name'),
       SEX_UPON_INTAKE
FROM   ANIMAL_INS;
```

[경기도에 위치한 식품창고 목록 출력하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131114
[나이 정보가 없는 회원 수 구하기]: https://school.programmers.co.kr/learn/courses/30/lessons/131528
[NULL 처리하기]: https://school.programmers.co.kr/learn/courses/30/lessons/59410