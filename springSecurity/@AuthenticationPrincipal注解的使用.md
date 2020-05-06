# @AuthenticationPrincipal注解的使用

@AuthenticationPrincipal注解是用于在用户通过SpringSecurity认证成功后用来获取用户数据才使用的注解，该注解可用于标注在Controller层方法的参数前，如果用户认证过，则对该方法发送请求后springSecurity会将认证过的用户信息存入到该注解标注的参数中，以便后来使用。

```java
package com.oRuol.springsecuritydemo.controller;

import com.oRuol.springsecuritydemo.bean.Result;
import com.oRuol.springsecuritydemo.bean.User;
import com.oRuol.springsecuritydemo.returnJson.Render;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author oRuol
 * @Date 2020/5/4 16:09
 */
@RestController
public class UserController {

    @GetMapping("/")
    public String hello(){
        return "hello";
    }
    @GetMapping("/page1")
    public String welPage1(){
        return "welcome page1";
    }

    @GetMapping("/page2")
    public String welPage2(){
        return "welcome page2";
    }

    @GetMapping("/page3")
    public String welPage3(){
        return "welcome page3";
    }

    @GetMapping("/test")
    public Object test(@AuthenticationPrincipal User user){
        if(user == null){
            return Result.error("未认证");
        }
        else {
            return user;
        }
    }
}
```