# springSecurity基本功能的实现

## 1.1WebSecurityConfigurerAdapter类的实现：

```java
package com.oRuol.springsecuritydemo.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.configurers.ExceptionHandlingConfigurer;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.crypto.password.StandardPasswordEncoder;
import org.springframework.security.crypto.scrypt.SCryptPasswordEncoder;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.security.web.authentication.logout.LogoutSuccessHandler;

/**
 * @author oRuol
 * @Date 2020/5/4 14:20
 */
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private Logger log= LoggerFactory.getLogger(SecurityConfig.class);
    private UserDetailsService userDetailsService;

    @Autowired
    //认证失败处理器实现
    private AuthenticationFailureHandler authenticationFailureHandler;

    @Autowired
    //AccessDeineHandler 用来解决认证过的用户访问无权限资源时的异常
    private AccessDeniedHandler accessDeniedHandler;

    @Autowired
    //认证成功的处理器的实现
    private AuthenticationSuccessHandler authenticationSuccessHandler;

    @Autowired
    //AuthenticationEntryPoint 用来解决匿名用户访问无权限资源时的异常
    private AuthenticationEntryPoint authenticationEntryPoint;

    @Autowired
    //登出成功的处理器的实现
    private LogoutSuccessHandler logoutSuccessHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .exceptionHandling()
                    .accessDeniedHandler(accessDeniedHandler)
                    .authenticationEntryPoint(authenticationEntryPoint)
                .and().authorizeRequests()
                    .antMatchers("/page1").hasAuthority("page1")
                    .antMatchers("/page2").hasAuthority("page2")
                    .antMatchers("/page3").hasAuthority("page3")
                    .antMatchers("/**","/").permitAll()
                .and().formLogin()
                    .loginProcessingUrl("/userlogin")//设置监听登录请求的路径
                    .usernameParameter("userName")//设置用户名字段
                    .passwordParameter("passwd")//设置密码字段
                    .failureHandler(authenticationFailureHandler)
                    .successHandler(authenticationSuccessHandler)
                .and().logout()
                    .logoutUrl("/logout")//设置监听登出请求的路径
                    .logoutSuccessHandler(logoutSuccessHandler)
                    .invalidateHttpSession(true)//invalidate-session 默认为true,用户在退出后Http session失效
                    .deleteCookies("JSESSIONID")
                .and().csrf().disable();//关闭csrf防护
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        super.configure(auth);
        //设置userDetailService和passwordEncoder
        auth.userDetailsService(userDetailsService).passwordEncoder(encoder());
    }

    /**
     * 创建一个passwordEncoder Bean
     * @return
     */
    @Bean public PasswordEncoder encoder(){
        return new BCryptPasswordEncoder();
    }
}
```

## 1.2LogoutSuccessHandler登出成功处理器的实现：

```java
package com.oRuol.springsecuritydemo.component;

import com.oRuol.springsecuritydemo.returnJson.Render;
import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.logout.LogoutSuccessHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author oRuol
 * @Date 2020/5/5 10:47
 */
@Component
public class MyLogoutSuccessHandler implements LogoutSuccessHandler {
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        Render.respJson("登出成功",response);
    }
}
```

## 1.3AuthenticationSuccessHandler认证成功处理器的实现

```java
package com.oRuol.springsecuritydemo.component;

import com.oRuol.springsecuritydemo.returnJson.Render;
import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author oRuol
 * @Date 2020/5/5 10:38
 */
@Component
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        Render.respJson("认证成功", response);
    }
}
```

## 1.4AuthenticationFailureHandler认证失败处理器的实现：

```java
package com.oRuol.springsecuritydemo.component;

import com.oRuol.springsecuritydemo.bean.Result;
import com.oRuol.springsecuritydemo.returnJson.Render;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author oRuol
 * @Date 2020/5/4 18:02
 */
@Component
public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        Render.respJson("认证失败", response);
    }
}
```

## 1.5AccessDeineHandler 用来解决认证过的用户访问无权限资源时的异常:

```java
package com.oRuol.springsecuritydemo.component;

import com.oRuol.springsecuritydemo.returnJson.Render;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author oRuol
 * @Date 2020/5/4 17:59
 */
@Component
public class MyAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        Render.respJson(accessDeniedException.getMessage(),response);
    }
}
```

## 1.6AuthenticationEntryPoint 用来解决匿名用户访问无权限资源时的异常

```java
package com.oRuol.springsecuritydemo.component;

import com.oRuol.springsecuritydemo.returnJson.Render;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author oRuol
 * @Date 2020/5/5 10:44
 */
@Component
public class MyAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        Render.respJson(authException.getMessage(), response);
    }
}
```

## 1.7Render类的实现

```java
package com.oRuol.springsecuritydemo.returnJson;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.oRuol.springsecuritydemo.bean.Result;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

/**
 * @author oRuol
 * @Date 2020/5/4 17:55
 */
//@Component
public class Render {

//    @Autowired
    private static ObjectMapper objectMapper=new ObjectMapper();

    public static void respJson(String msg, HttpServletResponse httpServletResponse) {
        httpServletResponse.setContentType("application/json");
        httpServletResponse.setCharacterEncoding("utf-8");
        PrintWriter writer = null;
        try {
            writer = httpServletResponse.getWriter();
            writer.write(objectMapper.writeValueAsString(Result.error(msg)));
            writer.flush();
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            writer.close();
        }
    }
}
```

## 1.8Result类的实现：

```java
package com.oRuol.springsecuritydemo.bean;

import org.springframework.boot.jackson.JsonObjectDeserializer;

/**
 * @author oRuol
 * @Date 2020/5/4 17:46
 */
public class Result<T> {
    private Integer code;
    private String msg;
    private T data=null;

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public static Result success(String msg){
        Result result=new Result();
        result.code=0;
        result.msg=msg;
        return result;
    }

//    public static Result success(JsonObject data){
//        Result result=new Result();
//        result.code=0;
//        result.msg="success";
//        result.data=data;
//        return result;
//    }

    public static Result error(String msg){
        Result result=new Result();
        result.code=-1;
        result.msg=msg;
        return result;
    }
}
```