# 一、rabbitMQ注解监听和AmqpAdmin的使用

## 1.1开启rabbitMQ的注解扫描

```java
package com.oRuol.amqp;

import org.springframework.amqp.rabbit.annotation.EnableRabbit;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * rabbitmq自动配置
 * 1、RabbitAutoConfiguration自动配置类
 * 2、自动配置的连接工厂CachingConnectionFactory
 * 3、RabbitProperties 其中封装了RabbitMQ的配置
 * 4、RabbitTemplate 给RabbitMQ发送和接收消息的
 * 5、AmqpAdmin RabbitMQ的系统管理组件
 */
@EnableRabbit   //开启rabbit注解
@SpringBootApplication
public class SpringbootAmqpApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootAmqpApplication.class, args);
    }

}
```

## 1.2编写rabbitMQ的队列消息监听类

```java
package com.oRuol.amqp.rabbit;

import com.oRuol.amqp.bean.Author;
import com.oRuol.amqp.bean.Book;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * @author oRuol
 * @Date 2020/5/9 15:14
 */
@Component
public class Listener {

    /**
     * 监听队列名称为atguigu.news中的消息
     * @param book 将队列中的消息取出并自动转换为Book类型
     *             (前提是消息队列中取出的数据可以转换为Book类型,
     *             若队列中存储的信息和方法的参数类型不相同的话，
     *             获取到的数据的各项参数都为null,
     *             并且队列中的数据也会消失)
     */
    @RabbitListener(queues = {"atguigu.news"})
    public void listenerQueue(Book book){
        System.out.println(book);
    }
}
```

## 1.3AmqpAdmin的使用

```java
package com.oRuol.amqp;

import com.oRuol.amqp.bean.Author;
import com.oRuol.amqp.bean.Book;
import org.junit.jupiter.api.Test;
import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.AutoConfigureOrder;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

@SpringBootTest
class SpringbootAmqpApplicationTests {

    @Autowired
    RabbitTemplate rabbitTemplate;

    /**
     * 发送数据到rabbitmq对应的队列中的方法
     */
    @Test
    void contextLoads() {
//        message需要自己构造一个，定义消息体内容和消息头
//        rabbitTemplate.send(exchange,routingKey, Message);

//        object默认当成消息体，只需要传入要发送的对象，自动序列化发送给rabbitmq;
//        rabbitTemplate.convertAndSend(exchage,routeKey,Object);


//        Map<String,Object> map = new HashMap<>();
//        map.put("msg","这是第一个消息");
//        map.put("date", Arrays.asList("helloworld",123,true));
////        map对象被默认序列化以后发送出去
//        rabbitTemplate.convertAndSend("exchange.direct","atguigu.news",map);

        //测试direct类型的exchange 自定义类存入消息队列，自定义类需要实现序列化接口，还得有无参构造方法
        Book book = new Book("曹雪芹", "红楼", "100");
        Author author = new Author("wunai","世界是多么的美好","2");
        rabbitTemplate.convertAndSend("exchange.direct","atguigu.news",author);

        //测试fanout类型的exchange
//        Book book = new Book("曹雪芹", "红楼", "100");
//        rabbitTemplate.convertAndSend("exchange.fanout","",book);

        //测试topic类型的exchange
//        Book book = new Book("曹雪芹", "红楼", "100");
//        rabbitTemplate.convertAndSend("exchange.topic","atguigu.time",book);

//        Book book = new Book("曹雪芹", "红楼", "100");
//        rabbitTemplate.convertAndSend("amqpAdmin.exchange","amqpadmin",book);
    }

    /**
     * 从rabbitmq中接收数据的方法
     */
    @Test
    public void receive(){
        Object o = rabbitTemplate.receiveAndConvert("atguigu.news");
        System.out.println(o.getClass());
        System.out.println(o);
    }


    /**
     *测试amqpAdmin的方法
     *
     */
    @Autowired
    AmqpAdmin amqpAdmin;

    @Test
    public void testAmqpAdminForExchange(){
        amqpAdmin.declareExchange(new DirectExchange("amqpAdmin.exchange"));
        System.out.println("exchange创建完成");
    }

    @Test
    public void testAmqpAdminForQueue(){
        amqpAdmin.declareQueue(new Queue("amqpAdmin.Queue"));
        System.out.println("queue创建完成");
    }

    @Test
    public void testAmqpAdminForBinding(){
        amqpAdmin.declareBinding(new Binding("amqpAdmin.Queue", Binding.DestinationType.QUEUE,"amqpAdmin.exchange","amqpadmin", null));
    }

    @Test
    public void testAmqpAdminDeleteForExchange(){
        amqpAdmin.deleteExchange("amqpAdmin.exchange");
    }

    @Test
    public void testAmqpAdminDeleteForQueue(){
        amqpAdmin.deleteQueue("amqpAdmin.Queue");
    }

}
```

## 1.4改变rabbitMQ   MessageConverter为json格式

```java
package com.oRuol.amqp.config;

import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author oRuol
 * @Date 2020/5/7 18:23
 */
@Configuration
public class MyAMQPConfig {

    @Autowired
    org.springframework.amqp.rabbit.connection.ConnectionFactory connectionFactory;

    /**
     * 设置rabbitMq的messageConverter的方式一、
     *      可以通过创建一个RabbitTemplate对象，通过RabbitTemplate对象的setMessageConverter()方法
     *      来设置messageConverter
     * @return
     */
//    @Bean
//    public RabbitTemplate messageConverter(){
//        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
//        rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());
//        return rabbitTemplate;
//    }

    /**
     * 设置rabbitMq的messageConverter的方式二、
     *      直接创建一个MessageConverter Bean对象
     * @return
     */
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

## 1.5测试用到的类

```java
package com.oRuol.amqp.bean;

import java.io.Serializable;

/**
 * @author oRuol
 * @Date 2020/5/8 17:23
 */
public class Book implements Serializable {

    private String author;
    private String bookName;
    private String price;

    public Book() {
    }

    public Book(String author, String bookName, String price) {
        this.author = author;
        this.bookName = bookName;
        this.price = price;
    }

    @Override
    public String toString() {
        return "Book{" +
                "author='" + author + '\'' +
                ", bookName='" + bookName + '\'' +
                ", price='" + price + '\'' +
                '}';
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }

    public String getPrice() {
        return price;
    }

    public void setPrice(String price) {
        this.price = price;
    }
}
```