---
title: GSON return오류
layout: default
parent: Spring Boot
grand_parent: Error
---

{: .warning }
Could not write JSON: JsonObject


return값을 JsonObject로 하기 위해 GSON depecdencies에 추가를 해줬지만 return을 할 때, 위와 같은 오류가 발생했다.

```
//** build.gradle */
//JSON 오브젝트 사용하기 위해 GSON 불러오기
implementation 'com.google.code.gson:gson:2.9.0'
```

```
//** UserController.java */
@GetMapping("/user/info/{id}")
public JsonObject getUserInfo(@PathVariable Long id){
    JsonObject result = userService.getUserInfo(id);
    return result;
}
```

찾아보니 [GSON을 return할 때 발생하는 에러]로 application.yml 파일에 아래 코드를 추가해주면 오류 없이 json이 잘 리턴되는 것을 볼 수 있다.


```
//** application.yml */
spring:
  mvc:
    converters:
      preferred-json-mapper: gson
```

- GSON 결과

![gson-error](/assets/images/gson-error.png)

[GSON을 return할 때 발생하는 에러]: https://m.blog.naver.com/spring1a/221781696854 "GSON return에러"
