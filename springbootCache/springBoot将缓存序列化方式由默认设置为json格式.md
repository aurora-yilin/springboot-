# springBoot将缓存序列化方式由默认设置为json格式

## 1.1重写RedisCacheManager设置默认缓存方式：

```java
package com.oRuol.oRuolCache.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;

import java.net.UnknownHostException;

/**
 * @author oRuol
 * @Date 2020/4/13 19:05
 */
@Configuration
public class MyRedisConfig {

    /**
     * 自定义RedisTemplate序列化规则
     * @param redisConnectionFactory
     * @return
     * @throws UnknownHostException
     */
//    @Bean
//    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
//            throws UnknownHostException {
//        RedisTemplate<Object, Object> template = new RedisTemplate<>();
//        template.setConnectionFactory(redisConnectionFactory);
//        Jackson2JsonRedisSerializer<Object> objectJackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
//        template.setDefaultSerializer(objectJackson2JsonRedisSerializer);
//        return template;
//    }

    /**
     * 修改redis缓存序列化的格式为json
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory){
        //创建一个redis的json序列化对象
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        RedisSerializationContext.SerializationPair<Object> objectSerializationPair = RedisSerializationContext.SerializationPair
                .fromSerializer(genericJackson2JsonRedisSerializer);

        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeValuesWith(objectSerializationPair);

        return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();
    }
}
```

## 1.2service层使用缓存注解

```java
package com.oRuol.oRuolCache.service;

import com.oRuol.oRuolCache.bean.Employee;
import com.oRuol.oRuolCache.mapper.EmployeeMapper;
import org.springframework.cache.annotation.*;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

/**
 * @author oRuol
 * @Date 2020/4/9 15:00
 */

/**
 * @CacheConfig注解是为了抽取
 *      @Cacheable、@CachePut、@CacheEvict、@Caching
 *      这些注解当中的公共配置（例如：@CacheConfig(cacheNames = {"emp"})）
 */
@CacheConfig(cacheNames = {"emp"})
@Service("employeeService")
public class EmployeeService {

    @Resource(name = "employeeMapper")
    EmployeeMapper employeeMapper;

    /**
     * @cacheable注解的参数：
     *      cacheNames/value:指定缓存组件的名字，将方法的返回结果放在哪个缓存中，参数的类型为数组形式，
     *          可以指定多个缓存；
     *      key:指定缓存数据的时候使用的key-value键值对的key( key = "#id"把key的值设置为参数id的值)
     *      (默认是使用方法参数的值)
     *      keyGenerator: key的生成器，可以自己定义key的生成器的组件的id
     *              key/keygenerator这两个参数二选一，只可以指定一个
     *      cacheManager: 指定缓存管理器,或者是指定cacheresolver指定获取解析器
     *      condition：指定符合条件的情况写才缓存
     *      （condition="#id>0"是指只有在参数id>0的时候才缓存）
     *      unless:否定缓存，当unless指定的条件为true，方法的返回值就不会被缓存
     *          (unless = "#result == null"
     *           unless = "#a0==2":如果第一个参数的值是2，结果不缓存)
     *      sync：是否使用异步模式(异步模式的意思是查询到结果后先返回后缓存，
     *          如果不开启异步模式的话默认在方法执行完以同步的方式将方法返回的结果存入到缓存中，
     *          异步模式不支持unless参数)
     *
     * @param id
     * @return
     */
    @Cacheable(cacheNames = {"emp"}, condition = "#id>0",key = "#id")
    public Employee getEmpById(Integer id){
        System.out.println("查询" + id + "号员工");
        Employee empById = employeeMapper.getEmpById(id);
        return empById;
    }

    /**
     * @CachePut:既调用方法，又更新缓存数据；
     * 修改了数据库的某个数据，同时更新缓存
     * 运行时机：
     *      1、先调用目标方法
     *      2、将目标方法的结果缓存起来
     *
     *  切记@CachePut注解的key因该和对应的@Cacheable注解的key一样
     */
    @CachePut(cacheNames = {"emp"}, key = "#employee.id")
    public Employee updateEmp(Employee employee){
        System.out.println("updateEmp" + employee);
        Integer result = employeeMapper.updateEmp(employee);
        System.out.println("结果"+result);
        return employee;
    }

    /**
     * @CacheEvict:缓存清除
     *  key指定要清除的数据
     *  allEntries：指定是否删除缓存中的所有数据
     *  beforeInvocation：缓存的清除是否在方法执行之前默认为false代表实在方法执行之后执行
     * @param id
     * @return
     */
    @CacheEvict(cacheNames = {"emp"}, key = "#id", beforeInvocation = true)
    public void deleteEmp(Integer id){
        System.out.println("deleteEmp:" + id);
    }

    /**
     * @Caching注解的使用案例
     * @param lastName
     * @return
     */
    @Caching(
            cacheable = {
                    @Cacheable(cacheNames = {"emp"},key = "#lastName")
            },
            put = {
                    @CachePut(cacheNames = "emp",key = "#result.id"),
                    @CachePut(cacheNames = "emp",key = "#result.email")
            }
            )
    public  Employee getEmpByLastName(String lastName){
        Employee empByLastName = employeeMapper.getEmpByLastName(lastName);

        return empByLastName;
    }
}
```

## 1.3Controller层调用service层

```java
package com.oRuol.oRuolCache.controller;

import com.oRuol.oRuolCache.bean.Employee;
import com.oRuol.oRuolCache.exception.UserNotExistException;
import com.oRuol.oRuolCache.service.EmployeeService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

/**
 * @author oRuol
 * @Date 2020/4/9 15:05
 */
@RestController
public class EmployeeController {

    @Resource(name = "employeeService")
    EmployeeService employeeService;

    @GetMapping("/emp/{id}")
    public Object getEmpById(@PathVariable("id")Integer id){
        if(id<=0){
            throw new UserNotExistException("用户不存在");
        }
        Employee empById = employeeService.getEmpById(id);
        if (empById != null) {
            return empById;
        }
        else {
            throw new UserNotExistException("用户不存在");
        }
    }

    @GetMapping("/emp")
    public Employee update(Employee employee){
        Employee emp = employeeService.updateEmp(employee);

        return emp;
    }

    @GetMapping("/delemp/{id}")
    public String deleteEmp(@PathVariable("id") Integer id){
        employeeService.deleteEmp(id);
        return "success";
    }

    @GetMapping("/emp/name/{lastName}")
    public Employee getEmpByLastName(@PathVariable("lastName") String lastName){
        Employee empByLastName = employeeService.getEmpByLastName(lastName);
        System.out.println(empByLastName);
        if (empByLastName != null) {
            return empByLastName;
        }
        else{
            throw new UserNotExistException("user not !!!");
        }
    }
}
```

2.1顺便记录一下KeyGenerator的设定方式：

```java
package com.oRuol.oRuolCache.config;

import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.lang.reflect.Method;
import java.util.Arrays;


/**
 * @author oRuol
 * @Date 2020/4/9 18:32
 */
@Configuration
public class MyCacheConfig {

    /**
     * 自定义一个keygenerator
     * @return
     */
    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return new KeyGenerator(){

            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName()+"[" + Arrays.asList(params).toString() +"]";
            }
        };
    }
}
```