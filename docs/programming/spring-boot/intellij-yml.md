---
title: IntelliJ yaml파일 한글 설정
layout: default
parent: Java
grand_parent: Programming
---


## yml 파일 한글깨짐 오류 해결   

![intellij-yml2](/assets/images/intellij-yml2.png)   

yml파일에 한글을 쓰면 깨지는 오류가 있어 [해결]해보았다.


Settings → Editor → File Encodings애 들어가 아래와 같이 Encoding을 'UTF-8'로 변경 한다.   
Transparent natvive-to-ascii conversion 체크박스도 체크한다.   


Settings창을 여는 단축키: ** ctrl + alt + s **

![intellij-yml1](/assets/images/intellij-yml1.png)

[결과]   
![intellij-yml3](/assets/images/intellij-yml3.png)

[해결]: https://www.lesstif.com/java/intellij-file-console-encoding-121012310.html "인텔리제이 yaml파일 한글깨짐 현상 해결"