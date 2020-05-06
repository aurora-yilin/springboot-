# 一、@ControllerAdvice注解的使用

1.1该注解只对Controller层抛出的异常有效，相当于Controller注解针对处理异常的升级版本，该注解所捕捉的异常都是事先定义好的异常。通常都是自定义一个异常类，然后用于处理对应的异常。

```java
package com.oRuol.oRuolCache.controller.exceptionHandler;

import com.oRuol.oRuolCache.exception.UserNotExistException;
import org.springframework.expression.spel.SpelEvaluationException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

/**
 * @author oRuol
 * @Date 2020/4/9 16:28
 */
@ControllerAdvice
public class MyExceptionHandler {

    /**
    *通俗来讲，@ExceptionHandler注解主要用于处理参数中对应的注解，当Controller层
    *抛出对应的注解后会被Controller层中标有@ControllerAdvice注解的类捕捉，再由类中对应的
    *有@ExceptionHandler注解的方法所处理
    **/
    @ExceptionHandler(UserNotExistException.class)
    @ResponseBody
    public Map<String,Object> handlerUserNotExistException(Exception e){
        Map<String,Object> map = new HashMap<>();
        map.put("code", "user.notexist");
        map.put("message", e.getMessage());

        return map;
    }

    @ExceptionHandler(SpelEvaluationException.class)
    @ResponseBody
    public Map<String,Object> handlerSpelEvaluationException(Exception e){
        Map<String,Object> map = new HashMap<>();
        map.put("code", "user.notexist");
        map.put("message", e.getMessage());

        return map;
    }
}
```

