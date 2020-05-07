

# 一、springAop环绕通知

本实例是将aop用作日志打印上面，本次测试虽然踩了个坑，但是却也学到了很多，找出了解决办法

## 1.1@aspect标注的通知类的实现

```java
package com.oRuol.springsecuritydemo.aspectHandler;

import com.oRuol.springsecuritydemo.exception.UserNameNotExistException;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

/**
 * @author oRuol
 * @Date 2020/5/6 17:17
 */

@Component
@Aspect
public class MyAroundAspect {

    @Pointcut("execution(public * com.oRuol.springsecuritydemo.controller.TestPetController.testDog(..))")
    public void pt1(){}

    @Pointcut("execution(public * com.oRuol.springsecuritydemo.mapper.UserMapper.findUserByName(..))")
    public void pt2(){}

    @Around("pt1()")
    public Object aroundPrintLog(ProceedingJoinPoint pjp) throws Throwable {
        Object result = null;
        try {
            Object[] args = pjp.getArgs();//得到方法执行所需的参数
            for (Object arg : args) {
                System.out.println(arg + "--------------------------------");
            }

            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。前置");

            result = pjp.proceed(args);//明确调用业务层方法（切入点方法）

            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。后置");

            return result;
        }
        /**
         * 如果环绕通知用于有抛出异常且抛出的异常在@ControllerAdvice类中有@ExceptionHandler对应
         * 则，环绕通知中的异常通知(try...catch)要不不写
         * ，要不抛出一个和捕捉到的异常一样的异常(throw new UserNameNotExistException(t.getMessage());)
         * ，要不将捕捉到的异常直接抛出(throw t)
         */
        catch (Throwable t){
            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。异常");
//            System.out.println(t+"-------------------------");
            throw new UserNameNotExistException(t.getMessage());
//            return result;
//            throw t;
        }
        finally {
            System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。最终");
        }
    }
}

```

## 1.2Controller层类的实现

```java
package com.oRuol.springsecuritydemo.controller;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.oRuol.springsecuritydemo.bean.Pet;
import com.oRuol.springsecuritydemo.exception.MyValidException;
import com.oRuol.springsecuritydemo.exception.UserNameNotExistException;
import lombok.extern.slf4j.Slf4j;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;
import java.util.HashMap;
import java.util.Map;

/**
 * @author oRuol
 * @Date 2020/5/5 17:02
 */
@Slf4j
@RestController
public class TestPetController {

    private final static Logger logger = LoggerFactory.getLogger(TestPetController.class);
    private ObjectMapper objectMapper = new ObjectMapper();

    /*@PostMapping("/pet")
    public Pet testPet(@Valid Pet pet, BindingResult bindingResult) throws JsonProcessingException {
        if(bindingResult.hasErrors()){
            Map<String,String> map = new HashMap<>();
            bindingResult.getFieldErrors()
                    .forEach((FieldError fieldError)->{
                        map.put(fieldError.getField(), fieldError.getDefaultMessage());
                    });
            String message = objectMapper.writeValueAsString(map);
            throw new MyValidException(message);
        }
        return pet;
    }*/
    @GetMapping("/dog")
    public String testDog(@RequestParam("time") String name){
        logger.info("time is too long");
        throw new UserNameNotExistException(name);
    }
}

```

## 1.3异常类的实现

```java
package com.oRuol.springsecuritydemo.exception;

/**
 * @author oRuol
 * @Date 2020/5/5 16:00
 */
public class UserNameNotExistException extends RuntimeException {
    public UserNameNotExistException(String Message){
        super(Message);
    }
    public UserNameNotExistException(){

    }
}
```

## 1.4ControllerAdvice标注的类的实现

```java
package com.oRuol.springsecuritydemo.controller.exceptionHandler;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.oRuol.springsecuritydemo.bean.PetResult;
import com.oRuol.springsecuritydemo.bean.Result;
import com.oRuol.springsecuritydemo.exception.MyValidException;
import com.oRuol.springsecuritydemo.exception.UserNameNotExistException;
import lombok.extern.slf4j.Slf4j;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import java.util.HashMap;
import java.util.Map;
import java.util.logging.Logger;

/**
 * @author oRuol
 * @Date 2020/5/5 17:15
 */
@Slf4j
@ControllerAdvice
public class MyAdviceException {
//    @ExceptionHandler(MyValidException.class)
//    public Result myValidException(String message){
//        return Result.error(message);
//    }
//    private final Logger log= (Logger) LoggerFactory.getLogger(MyAdviceException.class);
    ObjectMapper objectMapper = new ObjectMapper();

    /*@ResponseBody
    @ExceptionHandler(MyValidException.class)
    public PetResult myValidException(Exception e) throws JsonProcessingException {
        Map temp = objectMapper.readValue(e.getMessage(), HashMap.class);
        return new PetResult("400","字段不符合要求",temp);
    }*/

    @ResponseBody
    @ExceptionHandler(UserNameNotExistException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public String myUserNameNotExisException(Exception e) throws JsonProcessingException {
//        return new PetResult("200","test over",message);
        System.out.println(e.getMessage()+"++++++++++++++++++++++++++++++++");
        return objectMapper.writeValueAsString(new PetResult("200","test over",e.getMessage()));
    }
}
```

