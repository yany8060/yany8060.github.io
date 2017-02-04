---
title: Spring-boot-MyBatis配置-2
date: 2017-02-04 16:35:04
tags: [spring-boot,spring]
categories: [java]
---
### 多数据源配置
1. 分包: 不同数据源的在不同的目录下;事务的回滚需要创建根据数据源创建
2. 注解
3. AOP: aop注解切面需要在Service层进行数据源切换;事务可以将多个数据源放在一个事务中;


### 分包形式
> 不同的数据源的sql操作分布在不同的路径下

两个数据源的basepackage分别为：
* com.yany.dao.multi.ads
* com.yany.dao.multi.rds

在创建MapperScannerConfigurer时，对应不同的数据源扫描不同的basepackage路径
```java
mapperScannerConfigurer.setBasePackage("xxxx");
```

对应的xml，即MAPPER_PATH
* classpath:/com/yany/mapper/multi/ads/**.xml
* classpath:/com/yany/mapper/multi/rds/**.xml

在创建SqlSessionFactoryBean时，MAPPER_PATH对应分别对应于ads和rds的sql路径
```java
 sessionFactory.setMapperLocations(pathMatchingResourcePatternResolver.getResources(MAPPER_PATH));
```
在使用时，不同数据源的操作在分别在不同的路径创建即可。

### 注解形式
#### 准备好两个注解类，分别对应于两个数据源：
```java
public @interface RdsRepository {
}
public @interface AdsRepository {
}
```
同分包类似分别为Rds和Ads两个数据源创建两个SqlSessionFactoryBean和DataSourceTransactionManager，略微不同的是SqlSessionFactoryBean的setMapperLocations是__允许相同路径__。
#### 在创建两个MapperScannerConfigurer
```java
    /**
     * 以注解的方式 进行多数据源配置
     *
     * @return
     */
    @Bean
    public MapperScannerConfigurer createAnnotatationAdsMapperScannerConfigurer() {
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setBasePackage("com.yany.dao.multi.annotation");
        mapperScannerConfigurer.setSqlSessionFactoryBeanName("annotationAdsSqlSessionFactory");
        mapperScannerConfigurer.setAnnotationClass(AdsRepository.class);
        return mapperScannerConfigurer;
    }

    /**
     * 以注解的方式 进行多数据源配置
     *
     * @return
     */
    @Bean
    public MapperScannerConfigurer createAnnotatationRdsMapperScannerConfigurer() {
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setBasePackage("com.yany.dao.multi.annotation");
        mapperScannerConfigurer.setSqlSessionFactoryBeanName("annotationRdsSqlSessionFactory");
        mapperScannerConfigurer.setAnnotationClass(RdsRepository.class);
        return mapperScannerConfigurer;
    }
```
上述代码和以前的主要的区别在：setAnnotationClass设置对应不同的注解类

#### 使用时，在不同数据源的dao接口上添加对应的注解

```java
@RdsRepository
public interface AnnotationRdsDao {
    int selectCount();
}
@AdsRepository
public interface AnnotationAdsDao {
    int selectCount();
}
```
而sql对应的xml不变，对应好namespace即可

### AOP形式
#### 创建一个动态数据源
创建数据源类型的枚举类
```java
public enum DatabaseType {
    Ads, Rds
}
```

创建一个线程安全的DatabaseType容器

```java
public class DatabaseContextHolder {
    private final static ThreadLocal<DatabaseType> contextHolder = new ThreadLocal<>();

    public static DatabaseType getDatabaseType() {
        return contextHolder.get();
    }

    public static void setDatabaseType(DatabaseType type) {
        contextHolder.set(type);
    }

}
```
ThreadLocal类为每一个线程都维护了自己独有的变量拷贝，每个线程都拥有了自己独立的一个变量，避免并发问题。

创建动态数据源DynamicDataSource继承AbstractRoutingDataSource
```java
public class DynamicDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return DatabaseContextHolder.getDatabaseType();
    }
}
```

#### 创建AOP对应的MyBatis配置
  创建动态数据源的bean
```java
    @Bean
    public DynamicDataSource setDataSource() {
        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put(DatabaseType.Ads, adsDataSource);
        targetDataSources.put(DatabaseType.Rds, rdsDataSource);

        DynamicDataSource dataSource = new DynamicDataSource();
        dataSource.setTargetDataSources(targetDataSources);
        dataSource.setDefaultTargetDataSource(rdsDataSource);// 默认的datasource设置为rdsDataSource
        return dataSource;
    }
```
创建SqlSessionFactoryBean和DataSourceTransactionManager以及这个和以前类似不再赘述，具体看github上代码

#### 创建切边
扫描对应的Service层，在执行具体的服务代码前，根据调用的Service类进行数据源的切换。
```java
@Aspect
@Component
public class DataSourceAspect {
    /**
     * 使用空方法定义切点表达式
     */
    @Pointcut("execution(* com.yany.service.**.*(..))")
    public void declareJointPointExpression() {
    }

    @Before("declareJointPointExpression()")
    public void setDataSourceKey(JoinPoint point) {
        if (point.getTarget() instanceof IAdsAopService ||
                point.getTarget() instanceof AdsAopServiceImpl) {
            //根据连接点所属的类实例，动态切换数据源
            System.out.println("IAdsAopService Aspect");
            DatabaseContextHolder.setDatabaseType(DatabaseType.Ads);
        } else {//连接点所属的类实例是（当然，这一步也可以不写，因为defaultTargertDataSource就是该类所用的rdsDataSource）
            System.out.println("IRdsAopService Aspect");
            DatabaseContextHolder.setDatabaseType(DatabaseType.Rds);
        }

    }
}
```
上述切换规则比较简单，具体可根据业务情况，包目录结构，或者是类名规则等进行解析切换。

具体代码将github：https://github.com/yany8060/SpringDemo.git
博客：http://yany8060.xyz