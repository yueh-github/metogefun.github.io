---
title: 基于Spring aop技术来实现java内部的读写分离
layout: post
author: 岳浩
catalog: true
data:2018-01-09 13:00:00
tags: 
 - java mysql spring 
---


我们在基于java的项目里面，如果有场景需要做读写分离的话，其实上可以通过两种方式来实现

1：基于第三方进行来实现，比如Amoeba，mysql-proxy等，优点是不需要程序做任何改动，缺点是中间件做了中转代理，性能有所下降，增加运维的成本，支持的库有限
原理基本上是根据sql的前缀来做区分，比如insert,update 走主库，select 走从库


2：基于spring aop来实现读写分离，优点是速度快，所有的数据库都试用，缺点是需要修改程序，写代码要严格按照配置来写方法名，原理很简单
aop拦截service层的访问，获取到service调用的方法名，根据方法名设置一个ThreadLocal线程安全的key，根据这个key(master,slave)来切换获取数据源


第一步：定义动态数据源，简单的来解释就是determineCurrentLookupKey会根据key来返回你需要的数据源，afterPropertiesSet的作用是初始化读库，如果有多个读库

我们要自己来实现负载均衡算法，不能所有请求都落到一个从库，要负载均衡的落到每个读库，这样才起到多从的效果。
``` java
/**
 * 定义动态数据源，实现通过集成Spring提供的AbstractRoutingDataSource，只需要实现determineCurrentLookupKey方法即可
 * <p>
 * 由于DynamicDataSource是单例的，线程不安全的，所以采用ThreadLocal保证线程安全，由DynamicDataSourceHolder完成。
 *
 * @author yuehao
 */
@Slf4j
public class DynamicDataSource extends AbstractRoutingDataSource {

    private Integer slaveCount;

    // 轮询计数,初始为-1,AtomicInteger是线程安全的
    private AtomicInteger counter = new AtomicInteger(-1);

    // 记录读库的key
    private List<Object> slaveDataSources = new ArrayList<>(0);

    @Override
    protected Object determineCurrentLookupKey() {
        System.out.println("determineCurrentLookupKey 切换数据库 key：" + DynamicDataSourceHolder.getDataSourceKey());
        // 使用DynamicDataSourceHolder保证线程安全，并且得到当前线程中的数据源key
        if (DynamicDataSourceHolder.getDataSourceKey() == null) {
            return null;
        }
        if (DynamicDataSourceHolder.getDataSourceKey().equals("master")) {
            Object key = DynamicDataSourceHolder.getDataSourceKey();
            log.info("当前DataSource的key为: " + key);
            return key;
        }
        Object key = getSlaveKey();
        log.info("当前DataSource的key为: " + key);
        return key;
    }

    @SuppressWarnings("unchecked")
    @Override
    public void afterPropertiesSet() {
        super.afterPropertiesSet();

        // 由于父类的resolvedDataSources属性是私有的子类获取不到，需要使用反射获取
        Field field = ReflectionUtils.findField(AbstractRoutingDataSource.class, "resolvedDataSources");
        field.setAccessible(true); // 设置可访问

        try {
            Map<Object, DataSource> resolvedDataSources = (Map<Object, DataSource>) field.get(this);
            // 读库的数据量等于数据源总数减去写库的数量
            this.slaveCount = resolvedDataSources.size() - 1;
            for (Map.Entry<Object, DataSource> entry : resolvedDataSources.entrySet()) {
                if (entry.getKey().equals("master")) {
                    continue;
                }
                slaveDataSources.add(entry.getKey());
            }
        } catch (Exception e) {
            log.error("afterPropertiesSet error! ", e);
        }
    }

    /**
     * 轮询算法实现
     *
     * @return
     */
    public Object getSlaveKey() {
        // 得到的下标为：0、1、2、3……
        Integer index = counter.incrementAndGet() % slaveCount;
        if (counter.get() > 9999) { // 以免超出Integer范围
            counter.set(-1); // 还原
        }
        return slaveDataSources.get(index);
    }
}
```


第二步：使用ThreadLocal技术来记录当前线程中的数据源的key,保证请求的key是线程安全的
``` java
/**
 * 使用ThreadLocal技术来记录当前线程中的数据源的key
 * Created by yuehao on 2018/1/10.
 */
public class DynamicDataSourceHolder {

    //写库对应的数据源key
    private static final String MASTER = "master";

    //读库对应的数据源key
    private static final String SLAVE = "slave";

    //使用ThreadLocal记录当前线程的数据源key
    private static final ThreadLocal<String> holder = new ThreadLocal<>();

    /**
     * 设置数据源key
     *
     * @param key
     */
    public static void putDataSourceKey(String key) {
        holder.set(key);
    }

    /**
     * 获取数据源key
     *
     * @return
     */
    public static String getDataSourceKey() {
        return holder.get();
    }

    /**
     * 标记写库
     */
    public static void markMaster() {
        putDataSourceKey(MASTER);
    }

    /**
     * 标记读库
     */
    public static void markSlave() {
        putDataSourceKey(SLAVE);
    }

    /**
     * 是否是master
     *
     * @return
     */
    public static boolean isMaster() {
        return holder.get().equals(MASTER) ? true : false;
    }
}
```


第三步：AOP切面，通过该Service的方法名判断是应该走读库还是写库
``` java
/**
 * 定义数据源的AOP切面，通过该Service的方法名判断是应该走读库还是写库
 * Created by yuehao on 2018/1/10.
 */
public class DataSourceAspect {

    /**
     * 在进入Service方法之前执行
     *
     * @param point 切面对象
     */
    public void before(JoinPoint point) {
        // 获取到当前执行的方法名
        String methodName = point.getSignature().getName();
        if (isSlave(methodName)) {
            // 标记为读库
            DynamicDataSourceHolder.markSlave();
        } else {
            // 标记为写库
            DynamicDataSourceHolder.markMaster();
        }
    }

    /**
     * 判断是否为读库
     *
     * @param methodName
     * @return
     */
    private Boolean isSlave(String methodName) {
        // 方法名以query、find、get，select，search开头的方法名走从库
        return StringUtils.startsWithAny(methodName, "query", "find", "get","select","search");
    }
```

第四步：配置spring文件,自己读一下，注释很清楚
``` spel
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.1.xsd

        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.1.xsd"
       default-autowire="no">


    <!--主库-->
    <bean name="masterDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init"
          destroy-method="close">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="driverClassName" value="${jdbc.driverClassName}"/>

        <property name="initialSize" value="5"/>
        <property name="maxActive" value="20"/>
        <property name="maxIdle" value="20"/>
        <property name="minIdle" value="0"/>
        <property name="maxWait" value="60000"/>

        <property name="validationQuery" value="SELECT 1 FROM DUAL"/>
        <property name="testOnBorrow" value="false"/>
        <property name="testOnReturn" value="false"/>
        <property name="testWhileIdle" value="true"/>
        <property name="timeBetweenEvictionRunsMillis" value="60000"/>
        <property name="minEvictableIdleTimeMillis" value="25200000"/>
        <property name="removeAbandoned" value="true"/>
        <property name="removeAbandonedTimeout" value="1800"/>
        <property name="logAbandoned" value="true"/>
        <property name="filters" value="mergeStat"/>
    </bean>

    <!--从库01-->
    <bean name="slave01DataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init"
          destroy-method="close">
        <property name="url" value="${jdbc.slave01.url}"/>
        <property name="username" value="${jdbc.slave01.username}"/>
        <property name="password" value="${jdbc.slave01.password}"/>
        <property name="driverClassName" value="${jdbc.slave01.driverClassName}"/>

        <property name="initialSize" value="5"/>
        <property name="maxActive" value="20"/>
        <property name="maxIdle" value="20"/>
        <property name="minIdle" value="0"/>
        <property name="maxWait" value="60000"/>

        <property name="validationQuery" value="SELECT 1 FROM DUAL"/>
        <property name="testOnBorrow" value="false"/>
        <property name="testOnReturn" value="false"/>
        <property name="testWhileIdle" value="true"/>
        <property name="timeBetweenEvictionRunsMillis" value="60000"/>
        <property name="minEvictableIdleTimeMillis" value="25200000"/>
        <property name="removeAbandoned" value="true"/>
        <property name="removeAbandonedTimeout" value="1800"/>
        <property name="logAbandoned" value="true"/>
        <property name="filters" value="mergeStat"/>
    </bean>


    <!-- 定义数据源，使用自己实现的数据源 -->
    <bean id="dataSource" class="com.saf.panpan.splitting.DynamicDataSource">
        <!-- 设置多个数据源 -->
        <property name="targetDataSources">
            <map key-type="java.lang.String">
                <!-- 这个key需要和程序中的key一致 -->
                <entry key="master" value-ref="masterDataSource"/>
                <entry key="slave" value-ref="slave01DataSource"/>
            </map>
        </property>
        <!-- 设置默认的数据源，这里默认走写库 -->
        <property name="defaultTargetDataSource" ref="masterDataSource"/>
    </bean>

    <!--<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">-->
    <!--<property name="dataSource" ref="masterDataSource"/>-->
    <!--</bean>-->

    <!-- 定义事务管理器 -->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!-- dao接口映射成spring bean-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.saf.panpan.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>


    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:/mappers/base/mybatis_base.xml"/>
        <property name="mapperLocations" value="classpath:/mappers/*.xml"/>
    </bean>



    <!-- 开启AOP监听,只对当前配置文件有效 -->
    <aop:aspectj-autoproxy expose-proxy="true"/>
    <!-- 定义AOP切面处理器 -->
    <bean class="com.saf.panpan.splitting.DataSourceAspect" id="dataSourceAspect" />
    <aop:config>
        <!-- 定义切面，所有的service的所有方法 -->
        <aop:pointcut id="txPointcut" expression="execution(* *..service..*.*(..))" />
        <!-- 应用事务策略到Service切面 -->
        <aop:advisor advice-ref="transactionAdvice" pointcut-ref="txPointcut"/>

        <!-- 将切面应用到自定义的切面处理器上，-9999保证该切面优先级最高执行 -->
        <aop:aspect ref="dataSourceAspect" order="-9999">
            <aop:before method="before" pointcut-ref="txPointcut" />
        </aop:aspect>
    </aop:config>


    <!--<aop:config>-->
        <!--<aop:pointcut expression="execution(* *..service..*.*(..))"-->
                      <!--id="transactionPointCut"/>-->
        <!--<aop:advisor advice-ref="transactionAdvice" pointcut-ref="transactionPointCut"/>-->
    <!--</aop:config>-->

    <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
        <tx:attributes>

            <!-- 查询操作不走事务-->
            <tx:method name="query*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>


            <!-- 增删改事务-->
            <tx:method name="do*" propagation="REQUIRED" isolation="READ_COMMITTED"
                       rollback-for="java.lang.Throwable"/>
            <tx:method name="update*" propagation="REQUIRED" isolation="READ_COMMITTED"
                       rollback-for="java.lang.Throwable"/>
            <tx:method name="save*" propagation="REQUIRED" isolation="READ_COMMITTED"
                       rollback-for="java.lang.Throwable"/>
            <tx:method name="insert*" propagation="REQUIRED" isolation="READ_COMMITTED"
                       rollback-for="java.lang.Throwable"/>
            <tx:method name="cancel*" propagation="REQUIRED" isolation="READ_COMMITTED"
                       rollback-for="java.lang.Throwable"/>
            <tx:method name="complete*" propagation="REQUIRED" isolation="READ_COMMITTED"
                       rollback-for="java.lang.Throwable"/>

            <!-- 其他走默认策略-->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>

    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate" scope="prototype">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>

    <!-- JDBCTemplate配置 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```

测试一下吧，需要注意aop实际上是通过方法名来判断是走读还是走写，所以开发的时候，需要事务保证，需要走写库的，一定要注意方法名称的定义,这是约定


关注公众号获取更多干货文章
<img src="/img/weixin/qrcode_jishujiagou.jpg" width="100" height="100" alt="AltText" />

