```java
//匹配UserDaoImpl类中的所有方法
@Pointcut("execution(* com.zejian.dao.UserDaoImpl.*(..))")

//匹配UserDaoImpl类中的所有公共的方法
@Pointcut("execution(public * com.zejian.dao.UserDaoImpl.*(..))")

//匹配UserDaoImpl类中的所有公共方法并且返回值为int类型
@Pointcut("execution(public int com.zejian.dao.UserDaoImpl.*(..))")

//匹配UserDaoImpl类中第一个参数为int类型的所有公共的方法
@Pointcut("execution(public * com.zejian.dao.UserDaoImpl.*(int , ..))")

```

##### 其他指示符

```java
bean：Spring AOP扩展的，AspectJ没有对于指示符，用于匹配特定名称的Bean对象的执行方法；

//匹配名称中带有后缀Service的Bean。
@Pointcut("bean(*Service)")
private void myPointcut1(){}

this ：用于匹配当前AOP代理对象类型的执行方法；请注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配

//匹配了任意实现了UserDao接口的代理对象的方法进行过滤
@Pointcut("this(com.zejian.spring.springAop.dao.UserDao)")
private void myPointcut2(){}

target ：用于匹配当前目标对象类型的执行方法；

//匹配了任意实现了UserDao接口的目标对象的方法进行过滤
@Pointcut("target(com.zejian.spring.springAop.dao.UserDao)")
private void myPointcut3(){}
@within：用于匹配所以持有指定注解类型内的方法；请注意与within是有区别的， within是用于匹配指定类型内的方法执行；

//匹配使用了MarkerAnnotation注解的类(注意是类)
@Pointcut("@within(com.zejian.spring.annotation.MarkerAnnotation)")
private void myPointcut4(){}
@annotation(com.zejian.spring.MarkerMethodAnnotation) : 根据所应用的注解进行方法过滤

//匹配使用了MarkerAnnotation注解的方法(注意是方法)
@Pointcut("@annotation(com.zejian.spring.annotation.MarkerAnnotation)")
private void myPointcut5(){}

```

##### 5种通知函数

- 前置通知@Before
- 前置通知通过@Before注解进行标注，并可直接传入切点表达式的值，该通知在目标函数执行前执行，注意JoinPoint，是Spring提供的静态变量，通过joinPoint 参数，可以获取目标对象的信息,如类名称,方法参数,方法名称等，，该参数是可选的。

```java
/**

前置通知

@param joinPoint 该参数可以获取目标对象的信息,如类名称,方法参数,方法名称等
*/
@Before("execution(* com.zejian.spring.springAop.dao.UserDao.addUser(..))")
public void before(JoinPoint joinPoint){
System.out.println("我是前置通知");
}

```

- 后置通知@AfterReturning 
- 通过@AfterReturning注解进行标注，该函数在目标函数执行完成后执行，并可以获取到目标函数最终的返回值returnVal，当目标函数没有返回值时，returnVal将返回null，必须通过returning = “returnVal”注明参数的名称而且必须与通知函数的参数名称相同。请注意，在任何通知中这些参数都是可选的，需要使用时直接填写即可，不需要使用时，可以完成不用声明出来。如下

```java
/**

后置通知，不需要参数时可以不提供
*/
@AfterReturning(value="execution(* com.zejian.spring.springAop.dao.UserDao.*User(..))")
public void AfterReturning(){
 System.out.println("我是后置通知...");
}
/**

后置通知

returnVal,切点方法执行后的返回值
*/
@AfterReturning(value="execution(* com.zejian.spring.springAop.dao.UserDao.*User(..))",returning = "returnVal")
public void AfterReturning(JoinPoint joinPoint,Object returnVal){
 System.out.println("我是后置通知...returnVal+"+returnVal);
}

```



- 异常通知 @AfterThrowing
- 该通知只有在异常时才会被触发，并由throwing来声明一个接收异常信息的变量，同样异常通知也用于Joinpoint参数，需要时加上即可，如下：

```java
/**

抛出通知

@param e 抛出异常的信息
*/
@AfterThrowing(value="execution(* com.zejian.spring.springAop.dao.UserDao.addUser(..))",throwing = "e")
public void afterThrowable(Throwable e){
System.out.println("出现异常:msg="+e.getMessage());
}

```



- 最终通知 @After
  该通知有点类似于finally代码块，只要应用了无论什么情况下都会执行。

```java
/**

无论什么情况下都会执行的方法

joinPoint 参数
*/
@After("execution(* com.zejian.spring.springAop.dao.UserDao.*User(..))")
public void after(JoinPoint joinPoint) {
System.out.println("最终通知....");
}

```



- 环绕通知@Around 
- 环绕通知既可以在目标方法前执行也可在目标方法之后执行，更重要的是环绕通知可以控制目标方法是否指向执行，但即使如此，我们应该尽量以最简单的方式满足需求，在仅需在目标方法前执行时，应该采用前置通知而非环绕通知。案例代码如下第一个参数必须是ProceedingJoinPoint，通过该对象的proceed()方法来执行目标函数，proceed()的返回值就是环绕通知的返回值。同样的，ProceedingJoinPoint对象也是可以获取目标对象的信息,如类名称,方法参数,方法名称等等。

```java
@Around("execution(* com.zejian.spring.springAop.dao.UserDao.*User(..))")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
System.out.println("我是环绕通知前....");
//执行目标函数
Object obj= (Object) joinPoint.proceed();
System.out.println("我是环绕通知后....");
return obj;
}

```



##### 通知传递参数

在Spring AOP中，除了execution和bean指示符不能传递参数给通知方法，其他指示符都可以将匹配的方法相应参数或对象自动传递给通知方法。获取到匹配的方法参数后通过”argNames”属性指定参数名。如下，需要注意的是args(指示符)、argNames的参数名与before()方法中参数名 必须保持一致即param。

#### **XML方式**

![1555042708519](.\08519.png)

![1555042661811](.\2661811.png)

![1555042620838](.\0838.png)

![1555042414308](.\42414308.png)

- AOP的底层实际上是动态代理，动态代理分成了JDK动态代理和CGLib动态代理。如果被代理对象没有接口，那么就使用的是CGLIB代理(也可以直接配置使用CBLib代理)
- 如果是单例的话，那我们最好使用CGLib代理，因为CGLib代理对象运行速度要比JDK的代理对象要快
- AOP既然是基于动态代理的，那么它只能对方法进行拦截，它的层面上是方法级别的
- 无论经典的方式、注解方式还是XML配置方式使用Spring AOP的原理都是一样的，只不过形式变了而已。一般我们使用注解的方式使用AOP就好了。
- 注解的方式使用Spring AOP就了解几个切点表达式，几个增强/通知的注解就完事了，是不是贼简单...使用XML的方式和注解其实没有很大的区别，很快就可以上手啦。
- 引介/引入切面也算是一个比较亮的地方，可以用代理的方式为某个对象实现接口，从而能够使用借口下的方法。这种方式是非侵入式的~
- 要增强的方法还可以接收与被代理方法一样的参数、绑定被代理方法的返回值这些功能...
- ![1555042530184](.\42530184.png)



## 3. Spring事务管理

### 3.1 事务概念及其特性

**事务是逻辑上的一组操作，要么都执行，要么都不执行.**

**特性：ACID**

**原子性：** 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；

**一致性：** 执行事务前后，数据保持一致；

**隔离性：** 并发访问数据库时，一个用户的事物不被其他事物所干扰，各并发事务之间数据库是独立的；

**持久性:**  一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

### 3.2 Spring事务管理接口介绍

> **Spring事务管理接口：**

- **PlatformTransactionManager：** （平台）事务管理器
- **TransactionDefinition：** 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)
- **TransactionStatus：** 事务运行状态

**所谓事务管理，其实就是“按照给定的事务规则来执行提交或者回滚操作”**

#### 3.2.1 PlatformTransactionManager

**Spring并不直接管理事务，而是提供了多种事务管理器** ，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。  Spring事务管理器的接口是： **org.springframework.transaction.PlatformTransactionManager** ，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。

```java
Public interface PlatformTransactionManager()...{  
    // Return a currently active transaction or create a new one, according to the specified propagation behavior（根据指定的传播行为，返回当前活动的事务或创建一个新事务。）
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
    // Commit the given transaction, with regard to its status（使用事务目前的状态提交事务）
    Void commit(TransactionStatus status) throws TransactionException;  
    // Perform a rollback of the given transaction（对执行的事务进行回滚）
    Void rollback(TransactionStatus status) throws TransactionException;  
    } 


```

![1555061201683](.\83.png)

```xml
<!-- 事务管理器 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<!-- 数据源 -->
		<property name="dataSource" ref="dataSource" />
	</bean>



```

#### 3.2.2 TransactionDefinition

事务管理器接口 **PlatformTransactionManager** 通过 **getTransaction(TransactionDefinition definition)** 方法来得到一个事务，这个方法里面的参数是 **TransactionDefinition类** ，这个类就定义了一些基本的事务属性。

![1555061392720](.\2720.png)

**TransactionDefinition接口中的方法如下：**

TransactionDefinition接口中定义了5个方法以及一些表示事务属性的常量比如隔离级别、传播行为等等的常量。

我下面只是列出了TransactionDefinition接口中的方法而没有给出接口中定义的常量，该接口中的常量信息会在后面依次介绍到。

```java
public interface TransactionDefinition {
    // 返回事务的传播行为
    int getPropagationBehavior(); 
    // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getIsolationLevel(); 
    //返回事务的名字
    String getName()；
    // 返回事务必须在多少秒内完成
    int getTimeout();  
    // 返回是否优化为只读事务。
    boolean isReadOnly();
} 

```

##### 3.2.2.1事务隔离级别

事务隔离级别（定义了一个事务可能受其他并发事务影响的程度）

> **并发事务带来的问题**

在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务（多个用户对统一数据进行操作）。并发虽然是必须的，但可能会导致一下的问题。

- **脏读（Dirty read）:** 当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。

- **丢失修改（Lost to modify）:** 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。

  例如：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=A-1，事务2也修改A=A-1，最终结果A=19，事务1的修改被丢失。

- **不可重复读（Unrepeatableread）:** 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。

- **幻读（Phantom read）:** 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

**不可重复度和幻读区别：**

不可重复读的重点是修改，幻读的重点在于新增或者删除。

> **隔离级别**

TransactionDefinition 接口中定义了五个表示隔离级别的常量：

- **TransactionDefinition.ISOLATION_DEFAULT:**				使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:**         最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:**              允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 	   对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **TransactionDefinition.ISOLATION_SERIALIZABLE:** 	             最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

##### 3.2.2.2事务传播行为（为了解决业务层方法之间互相调用的事务问题）：

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：

**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

这里需要指出的是，前面的六种事务传播行为是 Spring 从 EJB 中引入的，他们共享相同的概念。而 **PROPAGATION_NESTED** 是 Spring 所特有的。以 PROPAGATION_NESTED 启动的事务内嵌于外部事务中（如果存在外部事务的话），此时，内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交。如果熟悉 JDBC 中的保存点（SavePoint）的概念，那嵌套事务就很容易理解了，其实嵌套的子事务就是保存点的一个应用，一个事务中可以包括多个保存点，每一个嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。

##### 3.2.2.3 事务超时属性(一个事务允许执行的最长时间)

所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

##### 3.2.2.4 事务只读属性（对事物资源是否执行只读操作）

事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。所谓事务性资源就是指那些被事务管理的资源，比如数据源、 JMS 资源，以及自定义的事务性资源等等。如果确定只对事务性资源进行只读操作，那么我们可以将事务标志为只读的，以提高事务处理的性能。在 TransactionDefinition 中以 boolean 类型来表示该事务是否只读。

##### 3.2.2.5 回滚规则（定义事务回滚规则）

这些规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的）。 但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

#### 3.2.3 TransactionStatus

TransactionStatus接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息.

PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。

**TransactionStatus接口接口内容如下：**

```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事物
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
} 

```

### 3.3 事务管理 

#### 3.3.1 编程式事务管理

基于底层 API 的编程式事务管理

根据PlatformTransactionManager、TransactionDefinition 和 TransactionStatus 三个核心接口，我们完全可以通过编程的方式来进行事务管理。

- ##### 基于底层 API 的事务管理示例代码

```java
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

- ##### 基于底层API的事务管理示例配置文件

```xml
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

如上所示，我们在类中增加了两个属性：一个是 TransactionDefinition 类型的属性，它用于定义一个事务；另一个是 PlatformTransactionManager 类型的属性，用于执行事务管理操作。

如果方法需要实施事务管理，我们首先需要在方法开始执行前启动一个事务，调用PlatformTransactionManager.getTransaction(...) 方法便可启动一个事务。创建并启动了事务之后，便可以开始编写业务逻辑代码，然后在适当的地方执行事务的提交或者回滚。

- ##### 基于 TransactionTemplate 的编程式事务管理

```java
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

```xml
<bean id="bankService"
class="footmark.spring.core.tx.programmatic.template.BankServiceImpl">
<property name="bankDao" ref="bankDao"/>
<property name="transactionTemplate" ref="transactionTemplate"/>
</bean>

```

TransactionTemplate 的 execute() 方法有一个 TransactionCallback 类型的参数，该接口中定义了一个 doInTransaction() 方法，通常我们以匿名内部类的方式实现 TransactionCallback 接口，并在其 doInTransaction() 方法中书写业务逻辑代码。这里可以使用默认的事务提交和回滚规则，这样在业务代码中就不需要显式调用任何事务管理的 API。doInTransaction() 方法有一个TransactionStatus 类型的参数，我们可以在方法的任何位置调用该参数的 setRollbackOnly() 方法将事务标识为回滚的，以执行事务回滚。

根据默认规则，如果在执行回调方法的过程中抛出了未检查异常，或者显式调用了TransacationStatus.setRollbackOnly() 方法，则回滚事务；如果事务执行完成或者抛出了 checked 类型的异常，则提交事务。

TransactionCallback 接口有一个子接口 TransactionCallbackWithoutResult，该接口中定义了一个 doInTransactionWithoutResult() 方法，TransactionCallbackWithoutResult 接口主要用于事务过程中不需要返回值的情况。当然，对于不需要返回值的情况，我们仍然可以使用 TransactionCallback 接口，并在方法中返回任意值即可。

#### 3.3.2 声明式事务管理

##### 3.3.2.1Spring 的声明式事务管理概述

Spring 的声明式事务管理在底层是建立在 AOP 的基础之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明（或通过等价的基于标注的方式），便可以将事务规则应用到业务逻辑中。因为事务管理本身就是一个典型的横切逻辑，正是 AOP 的用武之地。Spring 开发团队也意识到了这一点，为声明式事务提供了简单而强大的支持。

声明式事务管理曾经是 EJB 引以为傲的一个亮点，如今 Spring 让 POJO 在事务管理方面也拥有了和 EJB 一样的待遇，让开发人员在 EJB 容器之外也用上了强大的声明式事务管理功能，这主要得益于 Spring 依赖注入容器和 Spring AOP 的支持。依赖注入容器为声明式事务管理提供了基础设施，使得 Bean 对于 Spring 框架而言是可管理的；而 Spring AOP 则是声明式事务管理的直接实现者，这一点通过清单8可以看出来。

通常情况下，笔者强烈建议在开发中使用声明式事务，不仅因为其简单，更主要是因为这样使得纯业务代码不被污染，极大方便后期的代码维护。

和编程式事务相比，声明式事务唯一不足地方是，后者的最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。但是即便有这样的需求，也存在很多变通的方法，比如，可以将需要进行事务管理的代码块独立为方法等等。

下面就来看看 Spring 为我们提供的声明式事务管理功能。

##### 3.3.2.2 基于 TransactionInter... 的声明式事务管理

最初，Spring 提供了 TransactionInterceptor 类来实施声明式事务管理功能。先看清单8的配置文件：

- ##### 基于 TransactionInterceptor 的事务管理示例配置文件

```xml
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
<bean id="bankServiceTarget"
class="footmark.spring.core.tx.declare.origin.BankServiceImpl">
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

首先，我们配置了一个 TransactionInterceptor 来定义相关的事务规则，他有两个主要的属性：一个是 transactionManager，用来指定一个事务管理器，并将具体事务相关的操作委托给它；另一个是 Properties 类型的 transactionAttributes 属性，它主要用来定义事务规则，该属性的每一个键值对中，键指定的是方法名，方法名可以使用通配符，而值就表示相应方法的所应用的事务属性。

指定事务属性的取值有较复杂的规则，这在 Spring 中算得上是一件让人头疼的事。具体的书写规则如下：

```
`传播行为 [，隔离级别] [，只读属性] [，超时属性] [不影响提交的异常] [，导致回滚的异常]`

```

- 传播行为是唯一必须设置的属性，其他都可以忽略，Spring为我们提供了合理的默认值。
- 传播行为的取值必须以“PROPAGATION_”开头，具体包括：PROPAGATION_MANDATORY、PROPAGATION_NESTED、PROPAGATION_NEVER、PROPAGATION_NOT_SUPPORTED、PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_SUPPORTS，共七种取值。
- 隔离级别的取值必须以“ISOLATION_”开头，具体包括：ISOLATION_DEFAULT、ISOLATION_READ_COMMITTED、ISOLATION_READ_UNCOMMITTED、ISOLATION_REPEATABLE_READ、ISOLATION_SERIALIZABLE，共五种取值。
- 如果事务是只读的，那么我们可以指定只读属性，使用“readOnly”指定。否则我们不需要设置该属性。
- 超时属性的取值必须以“TIMEOUT_”开头，后面跟一个int类型的值，表示超时时间，单位是秒。
- 不影响提交的异常是指，即使事务中抛出了这些类型的异常，事务任然正常提交。必须在每一个异常的名字前面加上“+”。异常的名字可以是类名的一部分。比如“+RuntimeException”、“+tion”等等。
- 导致回滚的异常是指，当事务中抛出这些类型的异常时，事务将回滚。必须在每一个异常的名字前面加上“-”。异常的名字可以是类名的全部或者部分，比如“-RuntimeException”、“-tion”等等。

以下是两个示例：

```xml
`<``property` `name``=``"*Service"``>``PROPAGATION_REQUIRED，ISOLATION_READ_COMMITTED，TIMEOUT_20，``+AbcException，+DefException，-HijException``</``property``>`

```

以上表达式表示，针对所有方法名以 Service 结尾的方法，使用 PROPAGATION_REQUIRED 事务传播行为，事务的隔离级别是 ISOLATION_READ_COMMITTED，超时时间为20秒，当事务抛出 AbcException 或者 DefException 类型的异常，则仍然提交，当抛出 HijException 类型的异常时必须回滚事务。这里没有指定"readOnly"，表示事务不是只读的。

```xml
`<``property` `name``=``"test"``>PROPAGATION_REQUIRED，readOnly</``property``>`

```

以上表达式表示，针对所有方法名为 test 的方法，使用 PROPAGATION_REQUIRED 事务传播行为，并且该事务是只读的。除此之外，其他的属性均使用默认值。比如，隔离级别和超时时间使用底层事务性资源的默认值，并且当发生未检查异常，则回滚事务，发生已检查异常则仍提交事务。

配置好了 TransactionInterceptor，我们还需要配置一个 ProxyFactoryBean 来组装 target 和advice。这也是典型的 Spring AOP 的做法。通过 ProxyFactoryBean 生成的代理类就是织入了事务管理逻辑后的目标类。至此，声明式事务管理就算是实现了。我们没有对业务代码进行任何操作，所有设置均在配置文件中完成，这就是声明式事务的最大优点。

##### 3.3.2.3 基于 TransactionProxy... 的声明式事务管理

前面的声明式事务虽然好，但是却存在一个非常恼人的问题：配置文件太多。我们必须针对每一个目标对象配置一个 ProxyFactoryBean；另外，虽然可以通过父子 Bean 的方式来复用 TransactionInterceptor 的配置，但是实际的复用几率也不高；这样，加上目标对象本身，每一个业务类可能需要对应三个 <bean/> 配置，随着业务类的增多，配置文件将会变得越来越庞大，管理配置文件又成了问题。

为了缓解这个问题，Spring 为我们提供了 TransactionProxyFactoryBean，用于将TransactionInterceptor 和 ProxyFactoryBean 的配置合二为一。如清单9所示：

- ##### 基于 TransactionProxyFactoryBean 的事务管理示例配置文件

```xml
`<``beans......``>``......``<``bean` `id``=``"bankServiceTarget"``class``=``"footmark.spring.core.tx.declare.classic.BankServiceImpl"``>``<``property` `name``=``"bankDao"` `ref``=``"bankDao"``/>``</``bean``>``<``bean` `id``=``"bankService"``class``=``"org.springframework.transaction.interceptor.TransactionProxyFactoryBean"``>``<``property` `name``=``"target"` `ref``=``"bankServiceTarget"``/>``<``property` `name``=``"transactionManager"` `ref``=``"transactionManager"``/>``<``property` `name``=``"transactionAttributes"``>``<``props``>``<``prop` `key``=``"transfer"``>PROPAGATION_REQUIRED</``prop``>``</``props``>``</``property``>``</``bean``>``......``</``beans``>`

```

如此一来，配置文件与先前相比简化了很多。我们把这种配置方式称为 Spring 经典的声明式事务管理。相信在早期使用 Spring 的开发人员对这种配置声明式事务的方式一定非常熟悉。

但是，显式为每一个业务类配置一个 TransactionProxyFactoryBean 的做法将使得代码显得过于刻板，为此我们可以使用自动创建代理的方式来将其简化，使用自动创建代理是纯 AOP 知识，请读者参考相关文档，不在此赘述。

##### 3.3.2.4 基于 <tx> 命名空间的声明式事务管理

前面两种声明式事务配置方式奠定了 Spring 声明式事务管理的基石。在此基础上，Spring 2.x 引入了 <tx> 命名空间，结合使用 <aop> 命名空间，带给开发人员配置声明式事务的全新体验，配置变得更加简单和灵活。另外，得益于 <aop> 命名空间的切点表达式支持，声明式事务也变得更加强大。

如清单10所示：

- ##### 基于 <tx> 的事务管理示例配置文件

```xml
`<``beans......``>``......``<``bean` `id``=``"bankService"``class``=``"footmark.spring.core.tx.declare.namespace.BankServiceImpl"``>``<``property` `name``=``"bankDao"` `ref``=``"bankDao"``/>``</``bean``>``<``tx:advice` `id``=``"bankAdvice"` `transaction-manager``=``"transactionManager"``>``<``tx:attributes``>``<``tx:method` `name``=``"transfer"` `propagation``=``"REQUIRED"``/>``</``tx:attributes``>``</``tx:advice``>` `<``aop:config``>``<``aop:pointcut` `id``=``"bankPointcut"` `expression``=``"execution(* *.transfer(..))"``/>``<``aop:advisor` `advice-ref``=``"bankAdvice"` `pointcut-ref``=``"bankPointcut"``/>``</``aop:config``>``......``</``beans``>`

```

如果默认的事务属性就能满足要求，那么代码简化为如清单 11 所示：

- ##### 简化后的基于 <tx> 的事务管理示例配置文件

```xml
`<``beans......``>``......``<``bean` `id``=``"bankService"``class``=``"footmark.spring.core.tx.declare.namespace.BankServiceImpl"``>``<``property` `name``=``"bankDao"` `ref``=``"bankDao"``/>``</``bean``>``<``tx:advice` `id``=``"bankAdvice"` `transaction-manager``=``"transactionManager"``>``<``aop:config``>``<``aop:pointcut` `id``=``"bankPointcut"` `expression``=``"execution(**.transfer(..))"``/>``<``aop:advisor` `advice-ref``=``"bankAdvice"` `pointcut-ref``=``"bankPointcut"``/>``</``aop:config``>``......``</``beans``>`

```

由于使用了切点表达式，我们就不需要针对每一个业务类创建一个代理对象了。另外，如果配置的事务管理器 Bean 的名字取值为“transactionManager”，则我们可以省略 <tx:advice> 的 transaction-manager 属性，因为该属性的默认值即为“transactionManager”。

##### 3.3.2.5 基于 @Transactional 的声明式事务管理

除了基于命名空间的事务配置方式，Spring 2.x 还引入了基于 Annotation 的方式，具体主要涉及@Transactional 标注。@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如清单12所示：

- ##### 基于 @Transactional 的事务管理示例配置文件

```java
`@Transactional(propagation = Propagation.REQUIRED)``public boolean transfer(Long fromId， Long toId， double amount) {``return bankDao.transfer(fromId， toId， amount);``}`

```

Spring 使用 BeanPostProcessor 来处理 Bean 中的标注，因此我们需要在配置文件中作如下声明来激活该后处理 Bean，如清单13所示：

- ##### 启用后处理Bean的配置

```xml
`<``tx:annotation-driven` `transaction-manager``=``"transactionManager"``/>`

```

与前面相似，transaction-manager 属性的默认值是 transactionManager，如果事务管理器 Bean 的名字即为该值，则可以省略该属性。

虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 小组建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

基于 <tx> 命名空间和基于 @Transactional 的事务声明方式各有优缺点。基于 <tx> 的方式，其优点是与切点表达式结合，功能强大。利用切点表达式，一个配置可以匹配多个方法，而基于 @Transactional 的方式必须在每一个需要使用事务的方法或者类上用 @Transactional 标注，尽管可能大多数事务的规则是一致的，但是对 @Transactional 而言，也无法重用，必须逐个指定。另一方面，基于 @Transactional 的方式使用起来非常简单明了，没有学习成本。开发人员可以根据需要，任选其中一种使用，甚至也可以根据需要混合使用这两种方式。

如果不是对遗留代码进行维护，则不建议再使用基于 TransactionInterceptor 以及基于TransactionProxyFactoryBean 的声明式事务管理方式，但是，学习这两种方式非常有利于对底层实现的理解。

虽然上面共列举了四种声明式事务管理方式，但是这样的划分只是为了便于理解，其实后台的实现方式是一样的，只是用户使用的方式不同而已。

### 3.4 结束语

本教程的知识点大致总结如下：

- 基于 TransactionDefinition、PlatformTransactionManager、TransactionStatus 编程式事务管理是 Spring 提供的最原始的方式，通常我们不会这么写，但是了解这种方式对理解 Spring 事务管理的本质有很大作用。
- 基于 TransactionTemplate 的编程式事务管理是对上一种方式的封装，使得编码更简单、清晰。
- 基于 TransactionInterceptor 的声明式事务是 Spring 声明式事务的基础，通常也不建议使用这种方式，但是与前面一样，了解这种方式对理解 Spring 声明式事务有很大作用。
- 基于 TransactionProxyFactoryBean 的声明式事务是上中方式的改进版本，简化的配置文件的书写，这是 Spring 早期推荐的声明式事务管理方式，但是在 Spring 2.0 中已经不推荐了。
- 基于 <tx> 和 <aop> 命名空间的声明式事务管理是目前推荐的方式，其最大特点是与 Spring AOP 结合紧密，可以充分利用切点表达式的强大支持，使得管理事务更加灵活。
- 基于 @Transactional 的方式将声明式事务管理简化到了极致。开发人员只需在配置文件中加上一行启用相关后处理 Bean 的配置，然后在需要实施事务管理的方法或者类上使用 @Transactional 指定事务规则即可实现事务管理，而且功能也不必其他方式逊色。
