---
layout: default
title: LocalDateTime compareTo
parent: Programming
nav_order: 1
---


## LocalDateTime.compareTo()

compareTo() 메서드는 각 객체를 서로 비굑하여 숫자로 결과값을 반환해줌

```
LocalDateTime plusOneYear = LocalDateTime.now().plusYears(1).compareTo(LocalDateTime.now());
LocalDateTime timeNow = LocalDateTime.now().compareTo(LocalDateTime.now());
LocalDateTime minusOneYear = LocalDateTime.now()..minusYears(1).compareTo(LocalDateTime.now());
```

현재날짜 timeNow를 기준으로 plusOneYear, minusOneYear을 각각 비교해보면, 1, -1로 결과값이 나옴
만약, 1년이 아니라 각 2년씩 더하고 뺐다면 결과값은 2, -2로 나오므로 현재보다 이후 날짜인 경우 양수, 현재보다 이전 날짜인 경우 음수 그리고 동일한 날짜인 경우 0을 반환함

```
System.out.println(timeNow.compareTo(plusOneYear));
```

[실행결과]   


```
1
```

```
System.out.println(timeNow.compareTo(minusOneYear));
```

[실행결과]   


```
-1
```

```
System.out.println(timeNow.compareTo(timeNow));
```

[실행결과]   


```
0
```