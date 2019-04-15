Spirng 核心技术

## 1. IOC容器

### 1.1 IOC

IOC即控制反转，后来也被人们叫做依赖注入，其主要就是解决了依赖对象之间高度耦合的问题，使得类更加关心自己的功能而无需关心自己所需要的资源是怎么来的，从哪来的。要理解IOC的原理，需要搞清楚四件事：

1. 依赖关系是什么：依赖关系其实很简单，当类A的成员变量中有类B时，我们就说类A依赖于类B。
2. 没有IOC时类是如何获取自身所需的依赖对象的？引入IOC之后呢？ 当我们创建类A时，如果类A需要使用到类B，那么就需要创建一个类B的实例。那么我们一般都是直接new 一个类B的实例。**需要关心B何时实例化，B实例化是否需要额外的数据资源，如果需要我们还需要为类B找到其所需的资源**。整个依赖注入的过程都是IOC容器在负责完成，其中@Component相当于在告诉IOC容器，我这里有一个类B的资源。 @Autowired是告诉IOC容器，我需要一个类B的资源。然后IOC就会在自己管理的资源中，找到类B的资源给类A。
3. 什么被控制了？
   从上面的解释可以看到，什么被控制了，其实就是依赖对象（类B）的创建被控制了。依赖对象的创建要么由所需的类（类A）来控制，要么由IOC容器控制。
4. 什么被反转了？
   依赖对象（类B）创建的主动权反转了，之前是应用程序或者需要依赖对象的类（类A）自己主动创建依赖对象（类B），而引入IOC容器后，应用程序或者类A只能被动等待IOC容器为其创建所需的依赖对象。
5. IOC的好处：
   理解了上面的问题之后，我们其实能够发现其实IOC最大的好处就是降低了对象之间的耦合度，在没有IOC之前呢需要对象自己去找所需资源和依赖，但是引入IOC之后，对于资源对象或者依赖对象只需要告诉IOC容器，自己是什么样的资源。而对于需要依赖对象或者资源的对象呢，只需要告诉IOC容器自己正缺少什么样的资源，等IOC提供了这些资源，它只管用就好。而对于资源的寻找，获取注入都由IOC容器来完成。

### 1.2 Container

`org.springframework.context.ApplicationContext`接口代表Spring IoC容器，负责实例化，配置和组装bean。

ApplicationEventPublisher: 让容器拥有发布应用程序上下文事件的功能.

ResourcePatternResolver:实现了类似于资源加载器的功能，能够识别特定前缀加Ant风格的路径并加载到容器中.

LifeCycle：管理容器中bean的生命周期。

ConfigurableApplicationContext:主要新增了refresh()和close()方法，让ApplicationContext有用了启动，关系和刷新上下文的功能。

**ClassPathXmlApplicationContext与FileSystemXmlApplicationContext**:Spring提供的两个常用的ApplicationContext实现类，前者默认从类路径加载配置文件，后者从文件系统加载。 

容器通过读取配置元数据获取有关要实例化，配置和组装的对象的指令。配置元数据以XML，Java注释或Java代码表示。它允许您表达组成应用程序的对象以及这些对象之间丰富的相互依赖性。

![container-magic](.\container-magic.png)

#### 1.2.1 配置元数据

SpringIOC容器使用一种形式的配置元数据。此配置元数据表示作为应用程序开发人员，如何告诉Spring容器实例化、配置和组装应用程序中的对象。

- 基于注释的配置
- 基于Java的配置

#### 1.2.2 实例化容器

提供给`ApplicationContext`构造函数的位置路径是资源字符串，它允许容器从各种外部资源（如本地文件系统，Java等）加载配置元数据`CLASSPATH`。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

#### 1.2.3 使用容器

```java
//ClassPathXmlApplicationContext如果没有前缀默认就是classpath:
// FileSystemXmlApplicationContext如果没有前缀默认就是 file:
ApplicationContext ac =new ClassPathXmlApplicationContext("application.xml"); 

ApplicationContext ac =new FileSystemXmlApplicationContext(/User/Desktop/application.xml);

```

Spring容器不仅可以通过xml文件来初始化，@Configuration注解大家应该知道，也是注册Bean用的，当然了，Spring也提供了相应的AnnotationConfigApplicationContext.

```java
ApplicationContext context =new AnnotationConfigApplicationContext(Config.class);
```

Config类就是我们加了@Configuration的类了。

#### 1.2.4 WebApplicationContext

WebApplicationContext是Spring专门为Web应用准备的。它允许从相对于Web根目录的路径中装载配置文件完成初始化工作。**WebApplicationContext与ServletContext可以相互获得**.在非Web应用的环境下，Bean只有singleton和prototype两种作用域，而WebApplicationContext为Bean添加了三个新的作用域:request,session,global session。

最最核心的XmlWebApplicationContext和AnnotationConfigWebApplicationContext，大家猜都应该猜到了，一个是XML配置的，一个是@Configuration配置的。

然后说下WebApplicationContext和ServletContext是如何相互获取的，其实很简单，**在WebApplicationContext里有ServletContext成员变量，直接get就完了。在ServletContext里有一个写死的attrbute，也是直接get..而且Spring提供了一个WebApplicationContextUtils来封装了这个写死的attribute.**

```java
    //其实就是直接调用了ServletContext.getAttribute(ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
    public static WebApplicationContext getWebApplicationContext(ServletContext sc) {
        return getWebApplicationContext(sc, WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
    }
```

然后说下WebApplication的初始化，做过Web的都知道，其实要么在Web.xml里面配置一个ContextLoaderListener,要么配置一个自启动的LoaderServlet.其实比较简单了，展示下ContextLoaderListener.

```xml
  <!-- 加载spring容器 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/applicationContext-*.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
```

如果不是根据xml文件启动Spring容器而是根据@Configuration类启动，其实也很简单.

```xml
  <context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
  </context-param>
  <!-- 加载spring容器 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>com.zdy.Configuration</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
```

配置一个contextClass参数ApplicationContext改为AnnotationConfigWebApplicationContext，然后contextConfigLocation不是指定配置文件xml位置了，改为指定Configuration类的位置。

### 1.3 Bean

**JavaBeans** 是 Java 中一种特殊的类，可以将多个对象封装到一个对象（bean）中，特点是可序列化，提供无参构造器，提供 getter 方法和 setter 方法访问对象的属性。名称中的 "Bean" 是用于Java的可重用软件组件的惯用叫法

*在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。bean是一个由Spring IoC容器实例化，组装和管理的对象。*

#### 1.3.1 Bean命名

惯例是在命名bean时使用标准Java约定作为实例字段名称。也就是说，bean名称以小写字母开头，并从那里开始驼峰。这样的名字的例子包括`accountManager`， `accountService`，`userDao`，`loginController`，等等。

#### 1.3.2 实例化Bean

##### 1.3.2.1 使用构造函数实例化

1. 指定bean的class属性 
2. class需要一个默认的空构造器 

使用基于XML的配置元数据，您可以按如下方式指定bean类：

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

##### 1.3.2.2 使用静态工厂方法实例化

1. 指定class属性外 
2. 通过factory-method属性来指定创建bean实例的静态工厂方法

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

##### 1.3.2.3 使用实例工厂方法实例化

1. 定义一个工厂类 
2. 通过factory-bean属性指定工厂类，通过factory-method属性指定该工厂类的非静态工厂方法

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

```

### 1.4 依赖

#### 1.4.1 依赖注入（控制反转）

Spring提供了好几种的方式来给属性赋值

- 通过构造函数
- **通过set方法给属性注入值**
- p名称空间
- 自动装配(了解)
- **注解**

### 1.5 Bean Scopes

| Scope       | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| singleton   | （默认）将单个bean定义范围限定为每个Spring IoC容器的单个对象实例。 |
| prototype   | 将单个bean定义范围限定为任意数量的对象实例。                 |
| requst      | 将单个bean定义范围限定为单个HTTP请求的生命周期。也就是说，每个HTTP请求都有自己的bean实例，它是在单个bean定义的后面创建的。仅在具有Web感知功能的Spring环境中有效`ApplicationContext`。 |
| session     | 将单个bean定义范围限定为HTTP的生命周期`Session`。仅在具有Web感知功能的Spring环境中有效`ApplicationContext`。 |
| application | 将单个bean定义范围限定为a的生命周期`ServletContext`。仅在具有Web感知功能的Spring环境中有效`ApplicationContext`。 |
| websocket   | 将单个bean定义范围限定为a的生命周期`WebSocket`。仅在具有Web感知功能的Spring环境中有效`ApplicationContext`。 |

#### 1.5.1 singleton

当您定义bean定义并将其作为单一作用域时，Spring IoC容器只创建该bean定义定义的对象的一个实例。此单个实例存储在此类单例bean的缓存中，并且该命名Bean的所有后续请求和引用都将返回缓存对象。

#### 1.5.2 The Prototype Scope

如果作用域设置为 prototype，那么每次特定的 bean 发出请求时 Spring IoC 容器就创建对象的新的 Bean 实例。一般说来，满状态的 bean 使用 prototype 作用域和没有状态的 bean 使用 singleton 作用域。

#### 1.5.3  Singleton Beans with Prototype-bean Dependencies

当您使用具有原型bean依赖关系的单一范围bean时，请注意， 在实例化时会解析依赖关系。因此，如果您将原型范围的bean依赖注入到单例范围的bean中，则将实例化新的原型bean，然后将依赖注入到单例bean中。原型实例是唯一提供给单例范围bean的唯一实例。
但是，假设您希望单例范围的bean在运行时重复获取原型范围的bean的新实例。你不能依赖注入一个原型范围的bean到你的单例bean中，因为这个注入只发生 一次，当Spring容器实例化单例bean并解析和注入它的依赖关系时。如果您不止一次在运行时需要一个原型bean的新实例，请参阅方法注入。

#### 1.5.4 请求，会话，应用程序和WebSocket范围

- request，session，application，和websocket范围域仅在你如果使用**基于web的Spring容器**ApplicationContext实现（如 XmlWebApplicationContext）。
- 如果你将这些范围用于常规的Spring IoC容器（非基于web的容器）（比如ClassPathXmlApplicationContext），那么IllegalStateException会抛出一个抱怨未知bean范围的问题。 
  具体表现在以下五个方面：

1.初始网页配置
为了支持Bean的范围界定在request，session，application，和 websocket（即具有web作用域bean），在你定义你的Bean之前需要做少量的初始配置。（这些初始设置对于标准的范围域是不需要的例如：singleton和prototype）。

2.Request scope（请求范围）
考虑以下用于bean定义的XML配置：

<bean id="loginAction" class="com.foo.LoginAction" scope="request"/>
Spring容器LoginAction通过loginAction为每个HTTP请求使用bean定义来创建一个新的bean 实例。也就是说，这个 loginAction bean的作用域是HTTP请求级别。您可以根据需要更改创建的实例的内部状态，因为从同一个loginActionbean定义创建的其他实例将不会看到这些状态变化; 他们对个人的要求很特别。当请求完成处理时，作用于该请求的bean将被丢弃。

使用注释驱动组件或Java Config时，@RequestScope可以使用注释将组件分配给request作用域。

```java
@RequestScope
@Component
public class LoginAction {
// ...
}

```

3.session scope（会话范围）
考虑用以下的XML配置来定义一个会话范围：

<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
Spring容器UserPreferences通过在userPreferences单个HTTP的生命周期中使用bean定义来创建bean 的新实例Session。换句话说，这个userPreferencesbean的有效范围在HTTP Session级别。和request-scopedbean一样，您可以根据需要更改创建的实例的内部状态，因为知道其他Session使用同一个userPreferencesbean定义创建的实例的HTTP 实例也不会看到这些状态变化，因为它们是特定的到一个单独的HTTP Session。当HTTP Session最终被丢弃时，作用于该特定HTTP的bean Session也被丢弃。

使用注释驱动组件或Java Config时，@SessionScope可以使用注释将组件分配给session作用域。

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}

```

4.Application scope（应用范围）
考虑以下用以下的XML文件去给Bean定义一个应用程序的范围。

<bean id="appPreferences" class="com.foo.AppPreferences" scope="application"/>
Spring容器使用在整个Web应用程序只定义一次的appPreferences bean，创建一个新的appPreferences实例。 
也就是说，这个 appPreferencesbean的作用域是ServletContext作为一个常规ServletContext属性存储的 。这有点类似于Spring单例bean，但在两个重要方面有所不同：它是单例ServletContext，而不是每个Spring’ApplicationContext’（在任何给定的Web应用程序中可能有几个），它实际上是公开的，因此是可见的作为ServletContext属性。

使用注释驱动组件或Java Config时，@ApplicationScope 可以使用注释将组件分配给application作用域。

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}

```

5.作为依赖关系的作用域bean
Spring IoC容器不仅管理对象（bean）的实例化，还管理协作者（或依赖项）的连接。如果您想要将HTTP请求范围的bean注入（例如）一个更长寿命范围的另一个bean中，您可以选择注入一个AOP代理来代替范围bean。也就是说，您需要注入一个代理对象，该对象公开与作用域对象相同的公共接口，但也可以从相关作用域（例如HTTP请求）中检索真实目标对象，并将方法调用委托给实际对象。

#### 1.5.5 自定义范围

### 1.6  Bean本质

#### 1.6.1 Bean的生命周期

理解 Spring bean 的生命周期很容易。当一个 bean 被实例化时，它可能需要执行一些初始化使它转换成可用状态。同样，当 bean 不再需要，并且从容器中移除时，可能需要做一些清除工作。

尽管还有一些在 Bean 实例化和销毁之间发生的活动，但是本章将只讨论两个重要的生命周期回调方法，它们在 bean 的初始化和销毁的时候是必需的。

为了定义安装和拆卸一个 bean，我们只要声明带有 **init-method** 和/或 **destroy-method** 参数的 。init-method 属性指定一个方法，在实例化 bean 时，立即调用该方法。同样，destroy-method 指定一个方法，只有从容器中移除 bean 之后，才能调用该方法。

**初始化回调**

*org.springframework.beans.factory.InitializingBean* 接口指定一个单一的方法：

```java
void afterPropertiesSet() throws Exception;

```

因此，你可以简单地实现上述接口和初始化工作可以在 afterPropertiesSet() 方法中执行，如下所示：

```java
public class ExampleBean implements InitializingBean {
   public void afterPropertiesSet() {
      // do some initialization work
   }
}

```

在基于 XML 的配置元数据的情况下，你可以使用 **init-method** 属性来指定带有 void 无参数方法的名称。例如：

```xml
<bean id="exampleBean" 
         class="examples.ExampleBean" init-method="init"/>

```

下面是类的定义：

```java
public class ExampleBean {
   public void init() {
      // do some initialization work
   }
}

```

**销毁回调**

*org.springframework.beans.factory.DisposableBean* 接口指定一个单一的方法：

```java
void destroy() throws Exception;

```

因此，你可以简单地实现上述接口并且结束工作可以在 destroy() 方法中执行，如下所示：

```java
public class ExampleBean implements DisposableBean {
   public void destroy() {
      // do some destruction work
   }
}

```

在基于 XML 的配置元数据的情况下，你可以使用 **destroy-method** 属性来指定带有 void 无参数方法的名称。例如：

```xml
<bean id="exampleBean"
         class="examples.ExampleBean" destroy-method="destroy"/>

```

下面是类的定义：

```java
public class ExampleBean {
   public void destroy() {
      // do some destruction work
   }
}

```

如果你在非 web 应用程序环境中使用 Spring 的 IoC 容器；例如在丰富的客户端桌面环境中；那么在 JVM 中你要注册关闭 hook。这样做可以确保正常关闭，为了让所有的资源都被释放，可以在单个 beans 上调用 destroy 方法。

建议你不要使用 InitializingBean 或者 DisposableBean 的回调方法，因为 XML 配置在命名方法上提供了极大的灵活性。

**默认的初始化和销毁方法**

如果你有太多具有相同名称的初始化或者销毁方法的 Bean，那么你不需要在每一个 bean 上声明**初始化方法**和**销毁方法**。框架使用 元素中的 **default-init-method** 和 **default-destroy-method** 属性提供了灵活地配置这种情况，如下所示：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
    default-init-method="init" 
    default-destroy-method="destroy">

   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

</beans>

```

## 2.AOP

![A0FD228E-87B6-43A1-BA4A-96F1DCC1C533](.\A0FD228E-87B6-43A1-BA4A-96F1DCC1C533.png)

### 2.1 概念

面向切面编程，指扩展功能不修改源代码，将功能代码从业务逻辑代码中分离出来。

主要功能：日志记录，性能统计，安全控制，事务处理，异常处理等等。
主要意图：将日志记录，性能统计，安全控制，事务处理，异常处理等代码从业务逻辑代码中划分出来，通过对这些行为的分离，我们希望可以将它们独立到非指导业务逻辑的方法中，进而改变这些行为的时候不影响业务逻辑的代码。

### 2.2 特点

采用横向抽取机制，取代了传统纵向继承体系重复性代码。

### 2.3 底层实现

AOP底层使用动态代理实现。包括两种方式：

- 使用JDK动态代理实现。
- 使用cglib来实现

JDK动态代理实现：
      只能对实现了接口的类生成代理，而不是针对类，该目标类型实现的接口都将被代理。原理是通过在运行期间创建一个接口的实现类来完成对目标对象的代理。步骤如下：

1. 定义一个实现接口InvocationHandler的类
2. 通过构造函数，注入被代理类
3. 实现invoke（ Object proxy, Method method, Object[] args）方法
4. 在主函数中获得被代理类的类加载器
5. 使用Proxy.newProxyInstance( )产生一个代理对象
6. 通过代理对象调用各种方法

cglib动态代理实现：
       针对类实现代理，对是否实现接口无要求。原理是对指定的类生成一个子类，覆盖其中的方法，因为是继承，所以被代理的类或方法最好不要声明为final类型。

1. 定义一个实现了MethodInterceptor接口的类
2. 实现其intercept（）方法，在其中调用proxy.invokeSuper( )

代理方式的选择：
如果目标对象实现了接口，默认情况下回采用JDK的动态代理实现AOP，也可以强制使用cglib实现AOP
如果目标对象没有实现接口，必须采用cglib库，Spring会自动在JDK动态代理和cglib之间转换

静态代理和动态代理的区别：
静态代理：自己编写创建代理类，然后再进行编译，在程序运行前，代理类的.class文件就已经存在了。
动态代理：在实现阶段不用关心代理谁，而在运行阶段（通过反射机制）才指定代理哪一个对象。

### 2.4 AOP 原理

Spring AOP使用纯Java实现，它不需要专门的编译过程，也不需要特殊的类装载器，它在**运行期通过代理方式向目标类织入增强代码**。在Spring中可以无缝地将Spring AOP、IoC和AspectJ整合在一起。

### 2.5 AOP操作术语

Joinpoint(连接点): 类里面可以被增强的方法，这些方法称为连接点
Pointcut(切入点):所谓切入点是指我们要对哪些Joinpoint进行拦截的定义
Advice(通知/增强):所谓通知是指拦截到Joinpoint之后所要做的事情就是通知.通知分为前置通知,后置通知,异常通知,最终通知,环绕通知(切面要完成的功能)
Aspect(切面): 是切入点和通知（引介）的结合
Introduction(引介):引介是一种特殊的通知在不修改类代码的前提下, Introduction可以在运行期为类动态地添加一些方法或Field.
Target(目标对象):代理的目标对象(要增强的类)
Weaving(织入):是把增强应用到目标的过程，把advice 应用到 target的过程
Proxy（代理）:一个类被AOP织入增强后，就产生一个结果代理类   
其实我们只需要这么记忆即可：

切入点：在类里边可以有很多方法被增强，比如实际操作中，只是增强了个别方法，则定义实际被增强的某个方法为切入点。
通知/增强：增强的逻辑，称为增强，比如扩展日志功能，这个日志功能称为增强。包括：
前置通知：在方法之前执行
后置通知：在方法之后执行
异常通知：方法出现异常执行
最终通知：在后置之后执行
环绕通知：在方法之前和之后执行
切面：把增强应用到具体方法上面的过程称为切面。

### 2.6 Spring对AOP的支持

- 基于代理的经典SpringAOP

  - 需要实现接口，手动创建代理

- 纯POJO切面

  - 使用XML配置，aop命名空间

- ```
  @AspectJ注解驱动的切面
  
  ```

  - 使用注解的方式，这是最简洁和最方便的！

#### 基于注解的Spring AOP开发

```java
@After(value="execution(* com.zejian.spring.springAop.dao.UserDao.addUser(..))")
  public void after(){
      System.out.println("最终通知....");
  }

```

```java
/**

使用Pointcut定义切点
*/
@Pointcut("execution(* com.zejian.spring.springAop.dao.UserDao.addUser(..))")
private void myPointcut(){}

/**

应用切入点函数
*/
@After(value="myPointcut()")
public void afterDemo(){
System.out.println("最终通知....");
}

```

#### 切入点指示符

##### 通配符

在定义匹配表达式时，通配符几乎随处可见，如*、.. 、+ ，它们的含义如下：

- ：匹配方法定义中的任意数量的参数，此外还匹配类定义中的任意数量包

//任意返回值，任意名称，任意参数的公共方法
execution(public * *(..))
//匹配com.zejian.dao包及其子包中所有类中的所有方法
within(com.zejian.dao..*)

- ：匹配给定类的任意子类

//匹配实现了DaoUser接口的所有子类的方法
within(com.zejian.dao.DaoUser+)

- ：匹配任意数量的字符

//匹配com.zejian.service包及其子包中所有类的所有方法
within(com.zejian.service..*)
//匹配以set开头，参数为int类型，任意返回值的方法
execution(* set*(int))

##### 类型签名表达式

为了方便类型（如接口、类名、包名）过滤方法，Spring AOP 提供了within关键字。其语法格式如下：

within(<type name>)

```java
//匹配com.zejian.dao包及其子包中所有类中的所有方法
@Pointcut("within(com.zejian.dao..*)")

//匹配UserDaoImpl类中所有方法
@Pointcut("within(com.zejian.dao.UserDaoImpl)")

//匹配UserDaoImpl类及其子类中所有方法
@Pointcut("within(com.zejian.dao.UserDaoImpl+)")

//匹配所有实现UserDao接口的类的所有方法
@Pointcut("within(com.zejian.dao.UserDao+)")


```

##### 方法签名表达式

如果想根据方法签名进行过滤，关键字execution可以帮到我们，语法表达式如下

scope ：方法作用域，如public,private,protect
returnt-type：方法返回值类型
fully-qualified-class-name：方法所在类的完全限定名称
parameters 方法参数
execution(<scope> <return-type> <fully-qualified-class-name>.*(parameters))

对于给定的作用域、返回值类型、完全限定类名以及参数匹配的方法将会应用切点函数指定的通知，这里给出模型案例：

