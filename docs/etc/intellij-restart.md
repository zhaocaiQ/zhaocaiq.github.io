---
title: IntelliJ 소스코드 자동반영 설정
layout: default
parent: Etc
---

1. build.gradle에서 dependencies 설정   

```
/** build.gradle */

dependencies {
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
}
```

2. Settings 설정

- Compiler에서 "Build project automatically" 체크   
![intellij-restart1]("/assets/images/intellij-restart1.png")

- Advanced Settings에서 "Allow auto-make to start even if developed application is currently running" 체크   
![intellij-restart2]("/assets/images/intellij-restart2.png")

- Gradle에서 "Build and run using"과 "Run tests using"을 IntelliJ IDEA로 변경    
![intellij-restart3]("/assets/images/intellij-restart3.png")


[결과]   
콘솔에 restartedMain이라고 뜨면 설정이 완료되었다.    
![intellij-restart4]("/assets/images/intellij-restart4.png")

출처: IntelliJ 빌드/실행 플랫폼 변경 및 소스코드 자동반영 설정

[IntelliJ 빌드/실행 플랫폼 변경 및 소스코드 자동반영 설정]: https://u-it.tistory.com/entry/%EC%9D%B8%ED%85%94%EB%A6%AC%EC%A0%9C%EC%9D%B4-%EC%BB%A4%EB%AE%A4%EB%8B%88%ED%8B%B0%EB%B2%84%EC%A0%84-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0-1