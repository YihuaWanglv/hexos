---
title: spring事务用法演进
date: 2016-08-18 12:13:57
tags: [spring, java, transaction, 事务, 分布式事务]
---
# Spring事务用法演进

- 内容

![](/images/spring-transaction-001.png)


- 事务用法演进

![](/images/spring-transaction-002.png)


## 编程式事务管理

### 基于底层 API 的编程式事务管理

- 配置
```
<bean id="bankService" class="footmark.spring.core.tx.programmatic.origin.BankServiceImpl">
    <property name="bankDao" ref="bankDao"/>
    <property name="txManager" ref="transactionManager"/>
    <property name="txDefinition">
        <bean class="org.springframework.transaction.support.DefaultTransactionDefinition">
            <property name="propagationBehaviorName" value="PROPAGATION_REQUIRED"/>
        </bean>
    </property>
</bean>
```

- TransactionDefinition 类型的属性，它用于定义一个事务
- PlatformTransactionManager 类型的属性，用于执行事务管理操作

- 程序代码
```
public class BankServiceImpl implements BankService {
    private BankDao bankDao;
    private TransactionDefinition txDefinition;
    private PlatformTransactionManager txManager;
    ......
    public boolean transfer(Long fromId， Long toId， double amount) {
        TransactionStatus txStatus = txManager.getTransaction(txDefinition);
        boolean result = false;
        try {
            result = bankDao.transfer(fromId， toId， amount);
            txManager.commit(txStatus);
        } catch (Exception e) {
            result = false;
            txManager.rollback(txStatus);
            System.out.println("Transfer Error!");
        }
        return result;
    }
}
```

- 事务管理的代码散落在业务逻辑代码中，破坏了原有代码的条理性，并且每一个业务方法都包含了类似的启动事务、提交/回滚事务的样板代码。

### 基于TransactionTemplate的编程式事务管理
- 配置文件
```
<bean id="bankService"
class="footmark.spring.core.tx.programmatic.template.BankServiceImpl">
    <property name="bankDao" ref="bankDao"/>
    <property name="transactionTemplate" ref="transactionTemplate"/>
</bean>
```

- 程序示例
```
public class BankServiceImpl implements BankService {
    private BankDao bankDao;
    private TransactionTemplate transactionTemplate;
    ......
    public boolean transfer(final Long fromId， final Long toId， final double amount) {
        return (Boolean) transactionTemplate.execute(new TransactionCallback(){
            public Object doInTransaction(TransactionStatus status) {
                Object result;
                try {
                    result = bankDao.transfer(fromId， toId， amount);
                } catch (Exception e) {
                    status.setRollbackOnly();
                    result = false;
                    System.out.println("Transfer Error!");
                }
                return result;
            }
        });
    }
}
```
- 在数据访问层非常常见的模板回调模式
- 以匿名内部类的方式实现 TransactionCallback 接口，并在其 doInTransaction() 方法中书写业务逻辑代码


## 声明式事务管理

- 不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明（或通过等价的基于标注的方式）
- 声明式事务管理由Spring AOP实现

### 基于TransactionInterceptor的声明式事务管理

- 示例配置文件
```
<beans...>
    ......
    <bean id="transactionInterceptor"
    class="org.springframework.transaction.interceptor.TransactionInterceptor">
        <property name="transactionManager" ref="transactionManager"/>
        <property name="transactionAttributes">
            <props>
                <prop key="transfer">PROPAGATION_REQUIRED</prop>
            </props>
        </property>
    </bean>

    <bean id="bankServiceTarget" class="footmark.spring.core.tx.declare.origin.BankServiceImpl">
        <property name="bankDao" ref="bankDao"/>
    </bean>

    <bean id="bankService"
    class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="bankServiceTarget"/>
        <property name="interceptorNames">
            <list>
                <idref bean="transactionInterceptor"/>
            </list>
        </property>
    </bean>
    ......
</beans>
```

- TransactionInterceptor定义相关的事务规则，有两个主要的属性：transactionManager和transactionAttributes
- transactionManager，用来指定一个事务管理器，并将具体事务相关的操作委托给它；
- transactionAttributes，Properties 类型，主要用来定义事务规则，该属性的每一个键值对中，键指定的是方法名，方法名可以使用通配符，而值就表示相应方法的所应用的事务属性。
- ProxyFactoryBean, 组装 target 和advice. 通过 ProxyFactoryBean 生成的代理类就是织入了事务管理逻辑后的目标类。

#### 事务属性取值的书写规则：
```
传播行为 [，隔离级别] [，只读属性] [，超时属性] [不影响提交的异常] [，导致回滚的异常]
```
- 传播行为是唯一必须设置的属性，其他都可以忽略，Spring为我们提供了合理的默认值
- 传播行为的取值必须以“PROPAGATION_”开头，具体包括：PROPAGATION_MANDATORY、PROPAGATION_NESTED、PROPAGATION_NEVER、PROPAGATION_NOT_SUPPORTED、PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_SUPPORTS，共七种取值。
- 隔离级别的取值必须以“ISOLATION_”开头，具体包括：ISOLATION_DEFAULT、ISOLATION_READ_COMMITTED、ISOLATION_READ_UNCOMMITTED、ISOLATION_REPEATABLE_READ、ISOLATION_SERIALIZABLE，共五种取值。
- 如果事务是只读的，那么我们可以指定只读属性，使用“readOnly”指定。否则我们不需要设置该属性。
- 超时属性的取值必须以“TIMEOUT_”开头，后面跟一个int类型的值，表示超时时间，单位是秒。
- 不影响提交的异常是指，即使事务中抛出了这些类型的异常，事务任然正常提交。必须在每一个异常的名字前面加上“+”。异常的名字可以是类名的一部分。比如“+RuntimeException”、“+tion”等等。
- 导致回滚的异常是指，当事务中抛出这些类型的异常时，事务将回滚。必须在每一个异常的名字前面加上“-”。异常的名字可以是类名的全部或者部分，比如“-RuntimeException”、“-tion”等等。
- 两个示例
```
<property name="*Service">
    PROPAGATION_REQUIRED，ISOLATION_READ_COMMITTED，TIMEOUT_20，+AbcException，+DefException，-HijException
</property>
```
针对所有方法名以 Service 结尾的方法，使用 PROPAGATION_REQUIRED 事务传播行为，事务的隔离级别是 ISOLATION_READ_COMMITTED，超时时间为20秒，当事务抛出 AbcException 或者 DefException 类型的异常，则仍然提交，当抛出 HijException 类型的异常时必须回滚事务。这里没有指定"readOnly"，表示事务不是只读的。
```
<property name="test">PROPAGATION_REQUIRED，readOnly</property>
```
针对所有方法名为 test 的方法，使用 PROPAGATION_REQUIRED 事务传播行为，并且该事务是只读的。


### 基于TransactionProxyFactoryBean的声明式事务管理

- 示例配置文件
```
<beans......>
    ......
    <bean id="bankServiceTarget"
    class="footmark.spring.core.tx.declare.classic.BankServiceImpl">
        <property name="bankDao" ref="bankDao"/>
    </bean>

    <bean id="bankService"
    class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
        <property name="target" ref="bankServiceTarget"/>
        <property name="transactionManager" ref="transactionManager"/>
        <property name="transactionAttributes">
            <props>
                <prop key="transfer">PROPAGATION_REQUIRED</prop>
            </props>
        </property>
    </bean>
    ......
</beans>
```

- 显式为每一个业务类配置一个TransactionProxyFactoryBean的做法将使得代码显得过于刻板


### 基于<tx>命名空间的声明式事务管理

- 示例配置文件
```
<beans......>
    ......

    <tx:advice id="bankAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="transfer" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:pointcut id="bankPointcut" expression="execution(* *.transfer(..))"/>
        <aop:advisor advice-ref="bankAdvice" pointcut-ref="bankPointcut"/>
    </aop:config>
    ......
</beans>
```

- 如果默认的事务属性就能满足要求，那么代码简化为:
```
<beans......>
    ......
    <tx:advice id="bankAdvice" transaction-manager="transactionManager">

    <aop:config>
        <aop:pointcut id="bankPointcut" expression="execution(**.transfer(..))"/>
        <aop:advisor advice-ref="bankAdvice" pointcut-ref="bankPointcut"/>
    </aop:config>
    ......
</beans>
```
- 使用切点表达式，就不需要针对每一个业务类创建一个代理对象.


### 基于注解@Transactional的声明式事务管理

- @Transactional 可以作用于接口、接口方法、类以及类方法上
- 当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。

- 示例配置：
```
<!-- 配置事务管理器 -->
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
        p:dataSource-ref="dataSource">
</bean>

<!-- enables scanning for @Transactional annotations -->
<tx:annotation-driven transaction-manager="txManager" />
```

- 代码示例：
```
@Transactional(propagation = Propagation.REQUIRED)
public boolean transfer(Long fromId, Long toId, double amount) {
    return bankDao.transfer(fromId, toId, amount);
}
```

- 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。
- @Transactional 注解应该只被应用到 public 方法上. 在protected、private或者默认可见性的方法上使用@Transactional 注解，将被忽略，也不会抛出任何异常。


## 事务属性

### Required（需要）
- 若事务上下文已存在，则使用，如果不存在，则为此方法开启一个新事务

### Mandatory（强制必须）
- 强制事务上下文必须存在，若不存在，则抛出TransactionRequiredException异常

### RequiresNew（需要新的）
- 总是开启新事务，前一个事务若存在则会被挂起
- 此属性在此场景是很有用：一个事务如果需要与其外围包裹事务相独立，不受其执行结果的影响，自行完成提交(比如记录日志)

### Supports（支持）
- 告知容器，对象方法并不需要一个事务上下文，但当调用到这个方法而事务上下文恰巧存在时，则该方法会使用它
- 可用于查询的方法当中，如果此查询是在一个正在进行的事务中完成的，对目标方法应用Supports属性会让容器使用当前事务上下文，参考数据库操作记录，从而将事务中作出的各种修改液包括到查询结果中

### NotSupported（不支持）
- 告知容器此方法不使用事务
- 若一个事务已存在，容器会将事务暂停直至此方法结束
- 此属性在方法逻辑中有排斥事务上下文代码存在时很有用，可以暂时屏蔽一些不需要或不能用事务的逻辑

### Never（不用）
- 告知容器不允许有事务上下文存在。若调用方法前事务存在，则抛出异常

### PROPAGATION_NESTED
- 告知spring进行事务嵌套，并采用Required属性





## 事务传播级别

- 这些事务隔离级别设置需要依赖于底层数据库。底层数据库支持，这些设置才会生效


![](/images/spring-transaction-003.png)

### TransactionReadUncommitted
- 允许事务读取其他事务在提交到数据库之前产生的未提交更改。

![](/images/spring-transaction-004.png)


### TransactionReadCommitted
- 允许多个事务访问同一份数据，但将未提交的数据对其他事务隐藏，直至数据提交

![](/images/spring-transaction-005.png)


### TransactionRepeatableRead
- 保持了事务彼此隔绝。
- 保证一旦在某一事务中服务了数据库的一个值集，在后续的每次查询操作中都读到同样的值（除非此事务拿到这些数据的读写锁，并自行更改了数据）
- 在此级别下，一个事务如果要更改数据，而这一数据被其他事务读取时，此事务需要等待占用数据事务提交的操作（或直接返回失败）

![](/images/spring-transaction-006.png)


### TransactionSerializable
- java支持的最高的事务隔离级别
- 交错发生的事务被“堆迭”起来，以致同一时间点仅仅有一个事务具备访问目标数据的权力
- 性能会受到很大影响，而数据一致性将会极大提高

![](/images/spring-transaction-007.png)




## 分布式事务

### JTA和JTS
#### jta：java transaction api
- 开发人员用于事务管理的接口
- UserTransaction接口
```
begin();
commit();
rollback();
getStatus();
```
- TransactionManager接口
```
suspend();
resume();
```

#### jts：java transaction service
- 开源或商用的实现了jta的底层事务服务

![](/images/spring-transaction-008.png)

#### 要进行事务管理，我们需要两个东西：事务管理器和资源管理器
- 资源管理器（Resource Manager），对于数据库，就是数据源
- 事务管理器（Transaction Manager）
*控制JTA事务，管理事务生命周期，并协调资源*
*在JTA中，事务管理器抽象为TransactionManager，并通过底层事务服务（JTS）实现*
*负责控制和管理实际资源（数据库或JMS队列）*

### 分布式事务/XA事务
- 分布式事务和单机事务的区别就是，单机事务是事务管理器管理一个资源，而分布式事务则是事务管理器管理多个数据资源.

#### XA环境
在同一个处理单元中，需要协调多个数据资源完成逻辑，并保证ACID准备，则需要XA事务保证

#### XA接口
![](/images/spring-transaction-009.png)

- XA支持两阶段提交协议

### 实例：
- 使用spring boot + jta + atomikos实现分布式事务管理的代码例子
- 资金在库（atomikos_one），红包在库（atomikos_two）. 资金账号1转10元到资金账号2；红包账号2转10元到红包账号1

#### 表数据
- capital_account

![](/images/spring-transaction-010.png)

- red_packet_account

![](/images/spring-transaction-011.png)

#### 程序代码

- 配置2个数据源
```
order.datasource.url=jdbc:mysql://localhost:3306/atomikos_two
order.datasource.username=root
order.datasource.password=root

customer.datasource.url=jdbc:mysql://localhost:3306/atomikos_one
customer.datasource.username=root
customer.datasource.password=root
```

atomikos的maven依赖
```
    <dependency>
        <groupId>com.atomikos</groupId>
        <artifactId>transactions</artifactId>
        <version>3.9.3</version>
    </dependency>
    <dependency>
        <groupId>com.atomikos</groupId>
        <artifactId>transactions-jta</artifactId>
        <version>3.9.3</version>
    </dependency>
    <dependency>
        <groupId>com.atomikos</groupId>
        <artifactId>transactions-hibernate3</artifactId>
        <version>3.9.3</version>
        <exclusions>
            <exclusion>
                <artifactId>hibernate</artifactId>
                <groupId>org.hibernate</groupId>
            </exclusion>
        </exclusions>
    </dependency>
```

**AtomikosJtaPlatform这个类指明了，Atomikos需要我们提供UserTransaction和TransactionManager的实现**
```
public class AtomikosJtaPlatform extends AbstractJtaPlatform {
    private static final long serialVersionUID = 1L;
    static TransactionManager transactionManager;
    static UserTransaction transaction;
    @Override
    protected TransactionManager locateTransactionManager() {
        return transactionManager;
    }
    @Override
    protected UserTransaction locateUserTransaction() {
        return transaction;
    }
}
```

- transactionManager等配置
```
@Configuration
@ComponentScan
@EnableTransactionManagement
public class MainConfig {
    @Bean
    public PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
    @Bean
    public JpaVendorAdapter jpaVendorAdapter() {
        HibernateJpaVendorAdapter hibernateJpaVendorAdapter = new HibernateJpaVendorAdapter();
        hibernateJpaVendorAdapter.setShowSql(true);
        hibernateJpaVendorAdapter.setGenerateDdl(true);
        hibernateJpaVendorAdapter.setDatabase(Database.MYSQL);
        return hibernateJpaVendorAdapter;
    }
    @Bean(name = "userTransaction")
    public UserTransaction userTransaction() throws Throwable {
        UserTransactionImp userTransactionImp = new UserTransactionImp();
        userTransactionImp.setTransactionTimeout(10000);
        return userTransactionImp;
    }
    @Bean(name = "atomikosTransactionManager", initMethod = "init"
    , destroyMethod = "close")
    public TransactionManager atomikosTransactionManager() throws Throwable {
        UserTransactionManager userTransactionManager = new UserTransactionManager();
        userTransactionManager.setForceShutdown(false);
        AtomikosJtaPlatform.transactionManager = userTransactionManager;
        return userTransactionManager;
    }
    @Bean(name = "transactionManager")
    @DependsOn({ "userTransaction", "atomikosTransactionManager" })
    public PlatformTransactionManager transactionManager() throws Throwable {
        UserTransaction userTransaction = userTransaction();
        AtomikosJtaPlatform.transaction = userTransaction;
        TransactionManager atomikosTransactionManager = atomikosTransactionManager();
        return new JtaTransactionManager(userTransaction, atomikosTransactionManager);
    }
}
```

数据库1的数据源配置CustomerConfig
```
@Configuration
@DependsOn("transactionManager")
@EnableJpaRepositories(basePackages = "com.iyihua.sample.repository.customer"
, entityManagerFactoryRef = "customerEntityManager", transactionManagerRef = "transactionManager")
@EnableConfigurationProperties(CustomerDatasourceProperties.class)
public class CustomerConfig {
    @Autowired
    private JpaVendorAdapter jpaVendorAdapter;
    @Autowired
    private CustomerDatasourceProperties customerDatasourceProperties;

    @Primary
    @Bean(name = "customerDataSource", initMethod = "init", destroyMethod = "close")
    public DataSource customerDataSource() {
        MysqlXADataSource mysqlXaDataSource = new MysqlXADataSource();
        mysqlXaDataSource.setUrl(customerDatasourceProperties.getUrl());
        mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
        mysqlXaDataSource.setPassword(customerDatasourceProperties.getPassword());
        mysqlXaDataSource.setUser(customerDatasourceProperties.getUsername());
        mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
        AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
        xaDataSource.setXaDataSource(mysqlXaDataSource);
        xaDataSource.setUniqueResourceName("xads1");
        return xaDataSource;
    }
    @Primary
    @Bean(name = "customerEntityManager")
    @DependsOn("transactionManager")
    public LocalContainerEntityManagerFactoryBean customerEntityManager() throws Throwable {
        HashMap<String, Object> properties = new HashMap<String, Object>();
        properties.put("hibernate.transaction.jta.platform", AtomikosJtaPlatform.class.getName());
        properties.put("javax.persistence.transactionType", "JTA");
        LocalContainerEntityManagerFactoryBean entityManager = new LocalContainerEntityManagerFactoryBean();
        entityManager.setJtaDataSource(customerDataSource());
        entityManager.setJpaVendorAdapter(jpaVendorAdapter);
        entityManager.setPackagesToScan("com.iyihua.sample.domain.customer");
        entityManager.setPersistenceUnitName("customerPersistenceUnit");
        entityManager.setJpaPropertyMap(properties);
        return entityManager;
    }
}
```

数据库2的数据源配置OrderConfig
```
@Configuration
@DependsOn("transactionManager")
@EnableJpaRepositories(basePackages = "com.iyihua.sample.repository.order"
, entityManagerFactoryRef = "orderEntityManager", transactionManagerRef = "transactionManager")
@EnableConfigurationProperties(OrderDatasourceProperties.class)
public class OrderConfig {
    

}
```

- 业务方法:资金在库（atomikos_one），红包在库（atomikos_two）资金账号1转10元到资金账号2；红包账号2转10元到红包账号1**
```
@Service
public class StoreServiceImpl implements StoreService {
@Transactional()
    public void transfer() {
        CapitalAccount ca1 = capitalAccountRepository.findOne(1l);
        CapitalAccount ca2 = capitalAccountRepository.findOne(2l);
        RedPacketAccount rp1 = redPacketAccountRepository.findOne(1l);
        RedPacketAccount rp2 = redPacketAccountRepository.findOne(2l);
        BigDecimal capital = BigDecimal.TEN;
        BigDecimal red = BigDecimal.TEN;
        ca1.transferFrom(capital);
        ca2.transferTo(capital);
        capitalAccountRepository.save(ca1);
        capitalAccountRepository.save(ca2);
        rp2.transferFrom(red);
        rp1.transferTo(red);
        redPacketAccountRepository.save(rp1);
        redPacketAccountRepository.save(rp2);
    }
}
```



### get and run demo
- git clone https://github.com/YihuaWanglv/spring-boot-jta-atomikos-sample.git
- import db script in folder "docs"
- import project into ide and run App.java or build project and run the jar
- visit utl:http://localhost:8082/save to see saveTest