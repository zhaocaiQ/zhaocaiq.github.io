---
title: github add error
layout: default
parent: Github
grand_parent: Error
nav_order: 1
---


{: .warning }
warning: in the working copy of '--.md', LF will be replaced by CRLF the next time Git touches it



github에 add를 할 때 위와 같은 warning이 뜨면서 업로드될 파일들이 add되지 않는다.  
[CRLF 개행 문자]의 차이로 인해 발생하는 문제로 아래와 같이 설정 후에 다시 add를 하면 오류없이 잘 되는 것을 볼 수 있다.   

```
git config core.autocrlf true
```



혹은  

```
git config --global core.autocrlf true
```

[CRLF 개행 문자]: https://www.lesstif.com/gitbook/git-crlf-20776404.html "CRLF 개행 문자"