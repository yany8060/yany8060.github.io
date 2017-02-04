---
title: Spring-boot MyBatis配置 (1)
date: 2017-02-04 11:08:22
tags: [spring-boot,spring]
categories: [java]
---
> 本例使用mysql作为数据库，使用druid作为数据库连接池
> 主要有单数据源和多数据源实例
> 多数据源中又分为：1. 分包形式 2. aop形式 3. 注解形式

### 项目目录结构
![catalog.png](http://upload-images.jianshu.io/upload_images/1419542-fa4ab411fc0cd9a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/324)

[comment]: <> ( ![](/img/work/catalog.png) )

### MyBatis配置实现
> springBoot相比于原来的Spring的模式就是减少xml配置，将它们用java代码实现。

1. DataSource的bean，主要配置数据来源
2. SqlSessionFactoryBean的bean，引用 datasource，MyBatis配置，sql的xml扫描，以及各个插件的添加
3. MapperScannerConfigurer的bean的，主要设置基本扫描包，引用SqlSessionFactoryBean
4. DataSourceTransactionManager的bean，主要用设置事务

### 添加maven依赖
```
        <!-- aop -->        
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.4</version>
        </dependency>
        <!-- dataSource start -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.2.0</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.20</version>
        </dependency>
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.0.0</version>
        </dependency>
        <!-- dataSource end -->
```

### 单数据源
#### 基本配置
在application.yml中添加datasource配置：
```
spring:
  application:
    name: SpringBoot
  datasource:
    url: jdbc:mysql://localhost:3306/YanYPro?useUnicode=true&characterEncoding=UTF-8&&useSSL=false
    username: root
    password: *****
    driver-class-name: com.mysql.jdbc.Driver
    # 使用druid数据源
    type: com.alibaba.druid.pool.DruidDataSource
    # 初始化大小，最小，最大
    initialSize: 5
    minIdle: 5
    maxActive: 20
    # 配置获取连接等待超时的时间
    maxWait: 60000
    # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
    timeBetweenEvictionRunsMillis: 60000
    # 配置一个连接在池中最小生存的时间，单位是毫秒
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    # 打开PSCache，并且指定每个连接上PSCache的大小
    poolPreparedStatements: true
    maxPoolPreparedStatementPerConnectionSize: 20
    # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
    # 合并多个DruidDataSource的监控数据
    useGlobalDataSourceStat: true
```
配置mybatis-config:
以下只是实例，可自定义添加一些别的配置
```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 全局参数 -->
    <settings>
        <!-- 使全局的映射器启用或禁用缓存。 -->
        <setting name="cacheEnabled" value="true"/>
        <!-- 全局启用或禁用延迟加载。当禁用时，所有关联对象都会即时加载。 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!-- 当启用时，有延迟加载属性的对象在被调用时将会完全加载任意属性。否则，每种属性将会按需要加载。 -->
        <setting name="aggressiveLazyLoading" value="true"/>
        <!-- 是否允许单条sql 返回多个数据集  (取决于驱动的兼容性) default:true -->
        <setting name="multipleResultSetsEnabled" value="true"/>
        <!-- 是否可以使用列的别名 (取决于驱动的兼容性) default:true -->
        <setting name="useColumnLabel" value="true"/>
        <!-- 允许JDBC 生成主键。需要驱动器支持。如果设为了true，这个设置将强制使用被生成的主键，有一些驱动器不兼容不过仍然可以执行。  default:false  -->
        <setting name="useGeneratedKeys" value="true"/>
        <!-- 指定 MyBatis 如何自动映射 数据基表的列 NONE：不隐射　PARTIAL:部分  FULL:全部  -->
        <setting name="autoMappingBehavior" value="PARTIAL"/>
        <!-- 这是默认的执行类型  （SIMPLE: 简单； REUSE: 执行器可能重复使用prepared statements语句；BATCH: 执行器可以重复执行语句和批量更新）  -->
        <setting name="defaultExecutorType" value="SIMPLE"/>
        <!-- 使用驼峰命名法转换字段。 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- 设置本地缓存范围 session:就会有数据的共享  statement:语句范围 (这样就不会有数据的共享 ) defalut:session -->
        <setting name="localCacheScope" value="SESSION"/>
        <!-- 设置但JDBC类型为空时,某些驱动程序 要指定值,default:OTHER，插入空值时不需要指定类型 -->
        <setting name="jdbcTypeForNull" value="NULL"/>
    </settings>
</configuration>
```
#### 代码实现
##### SingleMyBatisConfig的类实现
继承EnvironmentAware并实现setEnvironment，为了获取默认配置文件application.yml的元素。
```java
@Configuration
public class SingleMyBatisConfig implements EnvironmentAware{
  private final static Logger logger = LoggerFactory.getLogger(SingleMyBatisConfig.class);
    private static String MYBATIS_CONFIG = "mybatis-config.xml";
    //mybatis mapper resource 路径
    private static String MAPPER_PATH = "classpath:/com/yany/mapper/single/**.xml";
    
    private RelaxedPropertyResolver propertyResolver;
    @Override
    public void setEnvironment(Environment environment) {
        this.propertyResolver = new RelaxedPropertyResolver(environment, "spring.datasource.");
    }

  .......
}
```
添加@Bean(name = "singleDataSource")，设置实现DataSource的bean
```java
    /**
     * @return
     * @Primary 优先方案，被注解的实现，优先被注入
     */
    @Primary
    @Bean(name = "singleDataSource")
    public DataSource singleDataSource() {
        logger.info("datasource url:{}", propertyResolver.getProperty("url"));

        DruidDataSource datasource = new DruidDataSource();
        datasource.setUrl(propertyResolver.getProperty("url"));
        datasource.setDriverClassName(propertyResolver.getProperty("driver-class-name"));
        datasource.setUsername(propertyResolver.getProperty("username"));
        datasource.setPassword(propertyResolver.getProperty("password"));


        datasource.setInitialSize(Integer.valueOf(propertyResolver.getProperty("initialSize")));
        datasource.setMinIdle(Integer.valueOf(propertyResolver.getProperty("minIdle")));
        datasource.setMaxWait(Long.valueOf(propertyResolver.getProperty("maxWait")));
        datasource.setMaxActive(Integer.valueOf(propertyResolver.getProperty("maxActive")));
        datasource.setTimeBetweenEvictionRunsMillis(Long.valueOf(propertyResolver.getProperty("timeBetweenEvictionRunsMillis")));
        datasource.setMinEvictableIdleTimeMillis(Long.valueOf(propertyResolver.getProperty("minEvictableIdleTimeMillis")));
        datasource.setValidationQuery(propertyResolver.getProperty("validationQuery"));
        datasource.setTestWhileIdle(Boolean.parseBoolean(propertyResolver.getProperty("testWhileIdle")));
        datasource.setTestOnBorrow(Boolean.parseBoolean(propertyResolver.getProperty("testOnBorrow")));
        datasource.setTestOnReturn(Boolean.parseBoolean(propertyResolver.getProperty("testOnReturn")));
        datasource.setPoolPreparedStatements(Boolean.parseBoolean(propertyResolver.getProperty("poolPreparedStatements")));
        datasource.setMaxPoolPreparedStatementPerConnectionSize(Integer.valueOf(propertyResolver.getProperty("maxPoolPreparedStatementPerConnectionSize")));

        try {
            datasource.setFilters(propertyResolver.getProperty("filters"));
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return datasource;

    }
```
添加@Bean(name = "singleSqlSessionFactory")，设置实现SqlSessionFactoryBean
```java
/**
     * 创建sqlSessionFactory实例
     *
     * @return
     */
    @Bean(name = "singleSqlSessionFactory")
    @Primary
    public SqlSessionFactoryBean createSqlSessionFactoryBean(@Qualifier("singleDataSource") DataSource singleDataSource) throws IOException {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        //设置mybatis configuration 扫描路径
        sqlSessionFactoryBean.setConfigLocation(new ClassPathResource(MYBATIS_CONFIG));
        sqlSessionFactoryBean.setDataSource(singleDataSource);

        PathMatchingResourcePatternResolver pathMatchingResourcePatternResolver = new PathMatchingResourcePatternResolver();
        sqlSessionFactoryBean.setMapperLocations(pathMatchingResourcePatternResolver.getResources(MAPPER_PATH));
        return sqlSessionFactoryBean;
    }
```
添加@Bean(name = "singleTransactionManager")，设置实现事务DataSourceTransactionManager
```java
    /**
     * 配置事务管理器
     */
    @Bean(name = "singleTransactionManager")
    @Primary
    public DataSourceTransactionManager transactionManager(@Qualifier("singleDataSource") DataSource singleDataSource) throws Exception {
        return new DataSourceTransactionManager(singleDataSource);
    }
```
以上SingleMyBatisConfig的配置完成，上面主要配置了数据源、SqlSessionFactoryBean、事务（DataSourceTransactionManager）。还差一个MapperScannerConfigurer的配置。

#### MapperScannerConfig
本来主要集中实现各个数据源的MapperScannerConfigurer
```java
@Configuration
public class MapperScannerConfig {

    /**
     * 单数据源配置
     *
     * @return
     */
    @Bean
    public MapperScannerConfigurer createSingleMapperScannerConfigurer() {
        System.out.println("singleDataSource");
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setBasePackage("com.yany.dao.single");
        mapperScannerConfigurer.setSqlSessionFactoryBeanName("singleSqlSessionFactory");
        return mapperScannerConfigurer;
    }
}
```

以上即为单数据源使用配置，而具体的dao层的编写以及sql的xml编写详情见：https://github.com/yany8060/SpringDemo.git
com.yany.dao.single中编写单属于的到接口
com.yany.mapper.single中编写对应的sql的xml

****
由于贴了比较多的代码，在下一篇多数据中中将直接类比这篇中的代码
详情请见：https://github.com/yany8060/SpringDemo.git
博客：http://yany8060.xyz
