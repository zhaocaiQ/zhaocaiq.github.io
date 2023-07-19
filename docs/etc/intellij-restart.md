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