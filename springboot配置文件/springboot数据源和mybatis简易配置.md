## springboot数据源和mybatis简易配置

```properties


#jdbc配置
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://192.168.31.217:3306/spring
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

#连接池的设置
#初始化时建立物理连接的个数
spring.datasource.druid.initial-size=5
#最小连接池数量
spring.datasource.druid.min-idle=5
#最大连接池数量 maxIdle已经不再使用
spring.datasource.druid.max-active=20
#获取连接时最大等待时间，单位毫秒
spring.datasource.druid.max-wait=60000
#申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
spring.datasource.druid.test-while-idle=true
#既作为检测的间隔时间又作为testWhileIdel执行的依据
spring.datasource.druid.time-between-eviction-runs-millis=60000
#销毁线程时检测当前连接的最后活动时间和当前时间差大于该值时，关闭当前连接
spring.datasource.druid.min-evictable-idle-time-millis=30000
#用来检测连接是否有效的sql 必须是一个查询语句
#mysql中为 select 'x'
#oracle中为 select 1 from dual
spring.datasource.druid.validation-query=select 'x'
#申请连接时会执行validationQuery检测连接是否有效,开启会降低性能,默认为true
spring.datasource.druid.test-on-borrow=false
#归还连接时会执行validationQuery检测连接是否有效,开启会降低性能,默认为true
spring.datasource.druid.test-on-return=false
#当数据库抛出不可恢复的异常时,抛弃该连接
#spring.datasource.druid.exception-sorter=true
#是否缓存preparedStatement,mysql5.5+建议开启
#spring.datasource.druid.pool-prepared-statements=true
#当值大于0时poolPreparedStatements会自动修改为true
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20
#配置扩展插件
spring.datasource.druid.filters=stat,wall
#通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.druid.connection-properties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
#合并多个DruidDataSource的监控数据
spring.datasource.druid.use-global-data-source-stat=true
#设置访问druid监控页的账号和密码,默认没有
#spring.datasource.druid.stat-view-servlet.login-username=admin
#spring.datasource.druid.stat-view-servlet.login-password=admin

#配置mybatis的资源映射文件
mybatis.mapper-locations=classpath:mybatis/mapping/*Mapper.xml
#设置mybatis的主配置文件的路径
mybatis.config-location=classpath:mybatis/mybatis-config.xml
#开启对mybatis的映射对象起别名
mybatis.type-aliases-package=com.oruol.datasourceconfigertest.domain
#开启mybatis的驼峰命名转换(配置了mybatis的主配置文件后，就不能在properties中配置驼峰命名转换)
#mybatis.configuration.map-underscore-to-camel-case=true
#当遇到同名的bean时，是否允许覆盖
spring.main.allow-bean-definition-overriding=true

#开启对应包的日志打印
#logging.level.com.oRuol.oRuolCache.mapper=debug

#debug=true

#spring.redis.host=192.168.31.217
```

