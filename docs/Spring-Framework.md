### Spring 基础入门

#### Spring 是什么？

- Spring 是 Java 应用最广的一个开源框架，为了解决企业应用开发的复杂性而创建的，但现在已经不止应用于企业应用

- 是一个轻量级的控制反转 `(IOC)` 和面向切面 `(AOP)` 的容器框架

- 它是当前 Java 语言开发应用程序的最重要的软件基础设施

  **Spring官网：http://spring.io/**

  

Spring 由众多设计精良模块组成，这些模块能够帮助我们快速开发高质量的程序，以下是 Spring框架的特性概述：

- **核心特性：IoC 容器（IoC Container）、面向切面编程（AOP）**、Spring 事件（Events）、资源管理（Resources）、国际化（i18n）、校验（Validation）、数据绑定（Data Binding）、类型装换（Type Conversion）、Spring 表达式（Spring Express Language）

- **数据存储：** JDBC、事务抽象（Transactions）、DAO支持（DAO Support）、O/R 映射（O/R Mapping）、XML 编列（XML Marshalling）

- **Web 技术：**  **Web Servlet 技术栈**（Spring MVC、WebSocket、SockJS）、**Web Reactive 技术栈**（Spring WebFlux、WebClient、WebSocket）

- **技术整合：** 远程调用（Remoting）、Java 消息服务（JMS）、Java 连接架构（JCA）、Java 管理扩展（JMX）、Java 邮件客户端（Email）、本地任务（Tasks）、本地调度（Scheduling）、缓存抽象（Caching）、Spring 测试 （Testing）

- **测试：**  模拟对象（Mock Objects）、TestContext 框架（TextContext Framework）、Spring  MVC 测试（Spring MVC Test）、Web 测试客户端（Web Test Client）



#### Spring Framework 有哪些核心模块?

- **spring-core：** Spring 基础API 模块，如资源管理，泛型处理 

- **spring-beans：** SpringBean 相关，如依赖查找，依赖注入 

- **spring-aop :** SpringAOP 处理，如动态代理，AOP 字节码提升 

- **spring-context :** 事件驱动、注解驱动，模块驱动等 

- **spring-expression：** Spring 表达式语言模块



#### Spring 的核心价值有哪些？

![核心价值](../assets/spring-framework/spring-core-value.png)

 

#### 什么是 IoC 容器？

- **IoC—Inversion of Control，即“控制反转”：** 不是具体的技术，而是一种设计思想。在 Java 开发中，IoC 意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。如何理解好  IoC 呢？理解好 IoC 的关键是要明确“谁控制谁，控制什么，为何是反转（有反转就应该有正转了），哪些方面反转了”，那我们来深入分析一下：
- **谁控制谁，控制什么：** 传统 Java SE 程序设计，我们直接在对象内部通过 new 进行创建对象，是程序主动去创建依赖对象；而 IoC 是有专门一个容器来创建这些对象，即由 IoC 容器来控制对象的创建；谁控制谁？当然是 IoC 容器控制了对象；控制什么？那就是主要控制了外部资源获取（不只是对象包括比如文件等）。
- **为何是反转，哪些方面反转了：** 有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。

**总结：所谓 IoC，就是由 Spring IoC 容器来负责对象的生命周期和对象之间的关系**



#### IoC 容器的职责有哪些？

- 依赖处理

  - 依赖查找 DL（Depend Lookup），主动或手动的依赖查找方式，通常需要依赖容器或标准 API 实现

  - 依赖注入 DI （Depend Injection），被动或自动依赖绑定的方式，无需依赖相关的额 API

    ![对比](../assets/spring-framework/DIvsDL.jpg)

- 生命周期管理

  - 容器
  - 托管的资源（Java Beans 或其他资源）

- 配置

  - 容器
  - 外部化配置
  - 托管的资源（Java Beans 或其他资源）



#### BeanFactory 和 ApplicationContext 谁才是 Spring IoC 容器？

- BeanFactory 和 ApplicationContext 是 Spring 的两大核心接口，而其中 ApplicationContext 是 BeanFactory 的超集。它们都可以当做 Spring 的容器，Spring 容器是生成 Bean 实例的工厂，并管理容器中的 Bean。在基于 Spring 的 Java EE 应用中，所有的组件都被当成 Bean 处理，包括数据源，Hibernate 的 SessionFactory、事务管理器等。

- 两者在功能设计上的区别

  - **BeanFactory 是 Spring 里面最低层的接口，提供了最简单的容器的功能，只提供了实例化对象和拿对象的功能；**

  - **ApplicationContext 应用上下文，继承 BeanFactory 接口，它是 Spring 的一个更高级的容器，提供了更多的有用的功能；**

    1) 面向切面 （AOP）

    2) 资源管理（Resources）

    3) 配置元信息（Configuration Metadata）

    4) 事件、响应机制（Events、ApplicationEventPublisher）

    5) 国际化（i18n）

    6) 注解（Annotations）

    7) Environment 抽象 

- 两者在装载 bean 时候的区别

  - BeanFactory 在启动的时候不会去实例化 Bean，从容器中拿 Bean 的时候才会去实例化；
  - ApplicationContext 在启动的时候就把所有的单例 Bean 全部实例化了。它还可以为 Bean 配置 lazy-init=true 来让 Bean 延迟实例化；

**注意：BeanFactory  是 IoC 的底层容器，而 FactoryBean 是创建 Bean 的一种方式，帮助实现复杂的初始化逻辑。**

#### BeanFactory  和 ApplicationContext  的示例

- BeanFactory 示例（通过 xml 配置）

```java
public static void main(String[] args) {
    // 创建 BeanFactory 容器
    DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
    XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
    // XML 配置文件 ClassPath 路径
    String location = "classpath:/META-INF/dependency-lookup-context.xml";
    // 加载配置
    int beanDefinitionsCount = reader.loadBeanDefinitions(location);
    System.out.println("Bean 定义加载的数量：" + beanDefinitionsCount);
    // 依赖查找集合对象
    lookupCollectionByType(beanFactory);
}

private static void lookupCollectionByType(BeanFactory beanFactory) {
    if (beanFactory instanceof ListableBeanFactory) {
        ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
        Map<String, User> users = listableBeanFactory.getBeansOfType(User.class);
        System.out.println("查找到的所有的 User 集合对象：" + users);
    }
}
```

- ApplicationContext 示例（通过注解配置的方式）

```java
public static void main(String[] args) {
    // 创建 BeanFactory 容器
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    // 将当前类  作为配置类（Configuration Class）
  applicationContext.register(AnnotationApplicationContextAsIoCContainerDemo.class);
    // 启动应用上下文
    applicationContext.refresh();
    // 依赖查找集合对象
    lookupCollectionByType(applicationContext);

    // 关闭应用上下文
    applicationContext.close();

}

/**
 * 通过 Java 注解的方式，定义了一个 Bean
 */
@Bean
public User user() {
    User user = new User();
    user.setId(1L);
    user.setName("小马哥");
    return user;
}

private static void lookupCollectionByType(BeanFactory beanFactory) {
    if (beanFactory instanceof ListableBeanFactory) {
        ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
        Map<String, User> users = listableBeanFactory.getBeansOfType(User.class);
        System.out.println("查找到的所有的 User 集合对象：" + users);
    }
}
```



### Spring IoC 依赖查找

#### 依赖查找入门示例

- 根据 Bean 名称查找

  - 实时查找

   ``` Java
    private static void lookupInRealTime(BeanFactory beanFactory) {
            User user = (User) beanFactory.getBean("user");
            System.out.println("实时查找：" + user);
     }
   ```

  - 延迟查找

  ``` xml
  需要先在配置文件中定义 ObjectFactory 对象
  <bean id="objectFactory" class="org.springframework.beans.factory.config.ObjectFactoryCreatingFactoryBean">
          <property name="targetBeanName" value="user"/>
  </bean>
  ```

  ``` java
  private static void lookupInLazy(BeanFactory beanFactory) {
          ObjectFactory<User> objectFactory = (ObjectFactory<User>) beanFactory.getBean("objectFactory");
          User user = objectFactory.getObject();
          System.out.println("延迟查找：" + user);
      }
  ```

- 根据 Bean 类型查找

  - 单个 Bean 对象

  ```Java
    private static void lookupByType(BeanFactory beanFactory) {
            //如果 User 类型的对象不止一个会抛出异常
        	  User user = beanFactory.getBean(User.class);
            System.out.println("实时查找：" + user);
        }
  ```

  - 集合 Bean 对象

  ``` java
  private static void lookupCollectionByType(BeanFactory beanFactory) {
          if (beanFactory instanceof ListableBeanFactory) {
              ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
              Map<String, User> users = listableBeanFactory.getBeansOfType(User.class);
              System.out.println("查找到的所有的 User 集合对象：" + users);
          }
      }
  ```

- 根据 Java 注解查找

  - 集合 Bean 对象

  ```java
  private static void lookupByAnnotationType(BeanFactory beanFactory) {
          if (beanFactory instanceof ListableBeanFactory) {
              ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
              Map<String, User> users = (Map) listableBeanFactory.getBeansWithAnnotation(Super.class);
              System.out.println("查找标注 @Super 所有的 User 集合对象：" + users);
          }
      }
  ```



#### 单一类型依赖查找

- 根据Bean 名称查找 
  - getBean(String) 
  - Spring 2.5 覆盖默认参数：getBean(String,Object...) 
- 根据Bean 类型查找 
  - Bean 实时查找 
    - Spring 3.0 getBean(Class) 
    - Spring 4.1 覆盖默认参数：getBean(Class,Object...) 
  - Spring 5.1  Bean 延迟查找 
    - getBeanProvider(Class) 
    - getBeanProvider(ResolvableType) 
- 根据Bean 名称+ 类型查找：getBean(String,Class)



#### 集合类型依赖查找

**集合类型依赖查找接口-ListableBeanFactory** 

- 根据Bean 类型查找 
  - 获取同类型Bean 名称列表 
    - getBeanNamesForType(Class) 
    - Spring 4.2 getBeanNamesForType(ResolvableType) 
  - 获取同类型Bean 实例列表 
    - getBeansOfType(Class) 以及重载方法 
- 通过注解类型查找 
  - Spring 3.0 获取标注类型Bean 名称列表 
    - getBeanNamesForAnnotation(Class<? extends Annotation>) 
  - Spring 3.0 获取标注类型Bean 实例列表 
    - getBeansWithAnnotation(Class<? extends Annotation>) 
  - Spring 3.0 获取指定名称+ 标注类型Bean 实例 
    - findAnnotationOnBean(String,Class<? extends Annotation>)



#### 层次性依赖查找

**层次性依赖查找接口-HierarchicalBeanFactory**

- 双亲BeanFactory：getParentBeanFactory() 
- 层次性查找 
  - 根据Bean 名称查找 
    - 基于containsLocalBean 方法实现 
  - 根据Bean 类型查找实例列表 
    - 单一类型：BeanFactoryUtils#beanOfType 
    - 集合类型：BeanFactoryUtils#beansOfTypeIncludingAncestors 
  - 根据Java 注解查找名称列
    - BeanFactoryUtils#beanNamesForTypeIncludingAncestors



#### 延迟依赖查找

Bean 延迟依赖查找接口 

- org.springframework.beans.factory.ObjectFactory

- org.springframework.beans.factory.ObjectProvider 

  - Spring 5 对 Java 8 特性扩展 

    - getIfAvailable(Supplier) 

    ```java
    private static void lookupIfAvailable(AnnotationConfigApplicationContext applicationContext) {
        ObjectProvider<User> userObjectProvider = applicationContext.getBeanProvider(User.class);
        // 如果当前对象不存在，则创建一个，属于兜底的方案
        User user = userObjectProvider.getIfAvailable(User::createUser);
        System.out.println("当前 User 对象：" + user);
    }
    ```

    - ifAvailable(Consumer) 

    - Stream 扩展-stream()

    ```java
    private static void lookupByStreamOps(AnnotationConfigApplicationContext applicationContext) {
        ObjectProvider<String> objectProvider = applicationContext.getBeanProvider(String.class);
        // Stream -> Method reference
        objectProvider.stream().forEach(System.out::println);
    }
    ```

#### 安全依赖查找

依赖查找安全性对比

| 依赖查找类型 | 代表实现                           | 是否安全 |
| :----------- | ---------------------------------- | -------- |
| 单一类型查找 | BeanFactory#getBean                | ==否==   |
|              | ObjectFactory#getObject            | ==否==   |
|              | ObjectProvider#getIfAvailable      | 是       |
| 集合类型查找 | ListableBeanFactory#getBeansOfType | 是       |
|              | ObjectProvider#stream              | 是       |

注意：层次性依赖查找的安全性取决于其扩展的单一或集合类型的 BeanFactory 接口

#### 内建可查找的依赖

- AbstractApplicationContext 内建可查找的依赖

![AbstractApplicationContext内建可查找单例对象](../assets/spring-framework/AbstractApplicationContext内建可查找单例对象.png)

- 注解驱动 Spring 应用上下文内建可查找的依赖（内建 bean）（部分）

| Bean 名称                                                    | Bean 实例                                    | 使用场景                                             |
| ------------------------------------------------------------ | -------------------------------------------- | ---------------------------------------------------- |
| org.springframework.contex t.annotation.internalConfigu rationAnnotationProcessor | ConfigurationClassPostProcesso r 对象        | 处理 Spring 配置类                                   |
| org.springframework.contex t.annotation.internalAutowir edAnnotationProcessor | AutowiredAnnotationBeanPostP rocessor 对象   | 处理 @Autowired 以及 @Value 注解                     |
| org.springframework.contex t.annotation.internalCommo nAnnotationProcessor | CommonAnnotationBeanPostPr ocessor 对象      | （条件激活）处理JSR-250 注解， 如@PostConstruct 等   |
| org.springframework.contex t.event.internalEventListener Processor | EventListenerMethodProcessor 对象            | 处理标注@EventListener 的 Spring 事件监听方法        |
| org.springframework.contex t.event.internalEventListener Factory | DefaultEventListenerFactory 对 象            | @EventListener 事件监听方法适配为ApplicationListener |
| org.springframework.contex t.annotation.internalPersiste nceAnnotationProcessor | PersistenceAnnotationBeanPost Processor 对象 | （条件激活）处理JPA 注解场景                         |



#### 依赖查找中的经典异常

![依赖查找常见异常](../assets/spring-framework/依赖查找常见Exception.png)




### Spring IoC 依赖注入

- 根据 Bean 名称注入

```xml
<bean id="userRepository"
          class="org.xxx.UserRepository">
    	<!-- 单个属性配置 -->
    	<property name="car" ref="Ferrari">
        <!-- 集合属性配置 -->
        <property name="users">
            <util:list>
                <ref bean="superUser"/>
                <ref bean="user"/>
            </util:list>
        </property>
    </bean>
```

- 根据 Bean 类型注入

```xml
<bean id="userRepository"
          class="org.xxx.UserRepository" autowire="byType">
    <!-- Auto-Wiring 自动装配 -->
 </bean>
```

- 注入容器内建非 Bean 对象

```java
1. 先在实体类中定义 BeanFactory
	class UserRepository {
   		 private BeanFactory beanFactory; // 內建非 Bean 对象（依赖）
    	 //省略getter和setter方法
	}
2. 使用 autowire 自动注入这个依赖对象
 	<bean id="userRepository"
          class="org.xxx.UserRepository" autowire="byType">
     </bean>
3. 获取内建非 Bean 对象（依赖）
    // 依赖来源一：自定义 Bean
    UserRepository userRepository = ac.getBean("userRepository", 				UserRepository.class);
    System.out.println(userRepository.getUsers());
    System.out.println(userRepository.getBeanFactory());
    // 依赖来源二：依赖注入（內建依赖）
    System.out.println(userRepository.getBeanFactory());
4. 使用 applicationContext 查找会抛异常（因为不是定义的普通 bean）
    // 依赖查找（错误）
    System.out.println(ac.getBean(BeanFactory.class));
```

- 注入容器内建 Bean 对象

```java
	// 依赖来源三：容器內建 Bean
    Environment environment = ac.getBean(Environment.class);
    System.out.println("获取 Environment 类型的 Bean：" + environment);
```



### Spring IoC 依赖来源

#### 依赖查找的来源

1. Spring BeanDefinition（自定义的 Bean）

   >  配置元数据
   >
   > <bean id="user" class = "...">
   >
   >  @Bean public User user(){}
   >
   > BeanDefinitionBuilder

2.  单例对象

   > API 配置
   >
   > 通过 SingletonBeanRegistry#registerSingleton(String beanName, Object singletonObject) 实现

3.  Spring 内建 BeanDefinition

   > Bean 实例，**在 AnnotationConfigUtils 类中可以找到这些类的 Bean 名称**
   >
   > ConfigurationClassPostProcessor 处理 Spring 配置类
   >
   > AutowiredAnnotationBeanPostProcessor 处理 @Autowired 以及 @Value 注解
   >
   > CommonAnnotationBeanPostProcesser （条件激活）处理 JSR-250 注解如@PostConstruct 等
   >
   > EventListenerMethodProcessr 处理标注 @EventListener 的 Spring 事件监听方法
   >
   > DefaultEventListenerFactory  @EventListener 事件监听方法适配为 ApplicationListener
   >
   > PersistenceAnnotationBeanPostProcessor （条件激活）处理 JPA 注解场景

4. Spring 内建单例对象

   ![AbstractApplicationContext内建可查找单例对象](../assets/spring-framework/AbstractApplicationContext内建可查找单例对象.png)

#### 依赖注入的来源

在依赖查找来源的基础上增加了 

1. ResolvableDependency （非 Spring 容器管理的对象）、

   > BeanFactory
   >
   > ResourceLoader
   >
   > ApplicationEventPublisher
   >
   > ApplicationContext
   >
   > 其中后面三个的类的实例指向同一个对象，就是 Spring 应用上下文实例
   >
   > 注册：ConfigurableListableBeanFactory#registerResolvableDependency

2. @Value 标注的外部化配置

   > @Value("${usr.name}")
   >  private String name;

#### Spring 容器管理和游离对象对比

![依赖对象](../assets/spring-framework/依赖对象.png)

#### Spring BeanDefinition 作为依赖来源

- 元数据：BeanDefinition
- 注册：BeanDefinitionRegistry#registryBeanDefinition
- 类型：延迟和非延迟
- 顺序：Bean 生命周期顺序按照注册顺序

#### 单例对象作为依赖来源

来源：外部普通 Java 对象（不一定是 POJO）

注册：SingletonBeanRegistry#registrySingleton

限制：

> 无生命周期管理
>
> 无法实现延迟初始化 Bean

#### 非 Spring 容器管理对象作为依赖来源

注册：ConfigurableListableBeanFactory#registryResolvableDependency

限制：

> 无生命周期管理
>
> 无实现延迟初始化 Bean
>
> 无法通过依赖查找

#### 外部化配置作为依赖来源

类型：非常规 Spring 对象依赖来源

限制：

> 无生命周期管理
>
> 无实现延迟初始化 Bean
>
> 无法通过依赖查找



### Spring 配置元信息
#### Spring 配置元信息

- Spring Bean 配置元信息-BeanDefinition
- Spring Bean 属性元信息-PropertyValues
- Spring 容器配置元信息
- Spring 外部化配置元信息-PropertySource
- Spring Profile 元信息-@Profile

#### Spring Bean 配置元信息



#### Spring Bean 属性元信息



#### Spring 容器配置元信息



#### 基于 XML 文件装载配置 Bean



#### 基于 Properties 文件装载配置 Bean



#### 基于 Java 注解装载配置 Bean



#### Spring Bean 配置元信息底层实现



#### 基于 XML 文件装载配置 IoC 容器



#### 基于 Java 注解装载配置 IoC 容器



#### 基于 Extensible XML authoring 扩展 Spring XML 元素



#### Extensible XML authoring 扩展原理



#### 基于 Properties 文件装载外部化配置



#### 基于 YAML 文件装载外部化配置







### Spring Bean 基础

#### 定义 Spring Bean

什么是 BeanDefinition ？

BeanDefinition 是 Spring 框架中定义 Bean 的配置元信息接口，包含：

- Bean 的类名
- Bean 行为配置元素，如作用域、自动绑定的模式，生命周期回调等
- 其他 Bean 引用，又可称作合作者或者依赖
- 配置设置，比如 Bean 属性（properties）

#### BeanDefinition 元信息

BeanDefinition 的元信息如图：

![BeanDefinition](../assets/spring-framework/BeanDefinition.png)

**BeanDefinition 的构建**

- 通过 BeanDefinitionBuilder

```java
	// 1.通过 BeanDefinitionBuilder 构建
    BeanDefinitionBuilder builder = 										BeanDefinitionBuilder.genericBeanDefinition(User.class);
	// 通过属性设置
     builder.addPropertyValue("id", 1)
        	.addPropertyValue("name", "小马哥");
    // 获取 BeanDefinition 实例
    BeanDefinition beanDefinition = builder.getBeanDefinition();
    // BeanDefinition 并非 Bean 终态，可以自定义修改
```

- 通过 AbstractBeanDefinition 以及它的派生类

```java
	// 2. 通过 AbstractBeanDefinition 以及派生类
	GenericBeanDefinition generic = new GenericBeanDefinition();
    // 设置 Bean 类型
	generic.setBeanClass(User.class);
    // 通过 MutablePropertyValues 批量操作属性
    MutablePropertyValues property = new MutablePropertyValues();
    property.add("id", 1)
            .add("name", "小马哥");
    // 通过 set MutablePropertyValues 批量操作属性
    generic.setPropertyValues(property);
```

#### 命名 Spring Bean

- 每个 Bean 拥有一个或多个标识符，这些标识符在 Bean 所在的容器必须是唯一的（在整个应用中可以不唯一），同时，还可以为 Bean 设置别名（Alias）。

- 在基于 XML 的配置元信息中，可用 id 或者 name 属性来规定 Bean 的标识符，如果想要引入别名的话，可在 name 属性使用半角逗号或分号来间隔。

- Bean 的 id 或 name 属性并非必须制定，如果留空的话，容器会为 Bean 自动生成一个唯一的名称。
- Bean 名称生成器（BeanNameGenerator 接口）有两个实现，一个是默认通用的 DefaultBeanNameGenerator，一个是基于注解扫描的 AnnotationBeanNameGenerator

#### Spring Bean 的别名

Bean 别名配置：

```xml
<!-- 将 Spring 容器中 "user" Bean 关联/建立别名 - "xiaomage-user" -->
<alias name="user" alias="xiaomage-user" />
```

使用别名依赖查找对应的 Bean：

```java
// 通过别名 xiaomage-user 获取曾用名 user 的 bean
User user = beanFactory.getBean("user", User.class);
User xiaomageUser = beanFactory.getBean("xiaomage-user", User.class);
System.out.println(user == xiaomageUser); //true
```

#### 注册 Spring Bean

BeanDefinition 注册

- xml 配置元信息
  - <bean name =“…" /> 
- Java 注解配置元信息
  - @Bean
  - @Component
  - @Import
- Java API 配置元信息
  - 命名方式：BeanDefinitionRegistry#registerBeanDefinition(String,BeanDefinition)
  - 非命名：BeanDefinitionReaderUtils#registerWithGeneratedName(AbstractBeanDefinition,Be anDefinitionRegistry)

```java
public static void registerUserBeanDefinition(BeanDefinitionRegistry registry, String beanName) {
    BeanDefinitionBuilder beanDefinitionBuilder = genericBeanDefinition(User.class);
    beanDefinitionBuilder
            .addPropertyValue("id", 1L)
            .addPropertyValue("name", "小马哥");

    // 判断如果 beanName 参数存在时
    if (StringUtils.hasText(beanName)) {
        // 注册 BeanDefinition
        registry.registerBeanDefinition(beanName, beanDefinitionBuilder.getBeanDefinition());
    } else {
        // 非命名 Bean 注册方法
  BeanDefinitionReaderUtils.registerWithGeneratedName(beanDefinitionBuilder.getBeanDefinition(), registry);
    }
}

public static void registerUserBeanDefinition(BeanDefinitionRegistry registry) {
    registerUserBeanDefinition(registry, null);
}
```

- 配置类方式：AnnotatedBeanDefinitionReader#register(Class...)

- 外部单例对象注册

  - SingletonBeanRegistry#registerSingleton

  ```java
  // 创建一个外部 UserFactory 对象
  UserFactory userFactory = new DefaultUserFactory();
  SingletonBeanRegistry sing = appContext.getBeanFactory();
  // 注册外部单例对象
  sing.registerSingleton("userFactory", userFactory);
  // 启动 Spring 应用上下文
  appContext.refresh();
  // 通过依赖查找的方式来获取 UserFactory
  UserFactory user2 = appContext.getBean("userFactory", UserFactory.class);
  System.out.println(userFactory == user2);
  ```

#### 实例化 Spring Bean

- 通过构造器

```xml
<bean id="user-by-constructor" class="org.xxx.User">
    <constructor-arg value="20"/>
    <constructor-arg value="zhangsan"/>
</bean>
```

- 通过静态工厂方法

```xml
<!-- 静态方法实例化 Bean -->
<bean id="user-by-static-method" class="org.xxx.User"
factory-method="createUser" />
<!-- User 类中的静态方法 -->
public static User createUser() {
        User user = new User();
        user.setId(1L);
        user.setName("小马哥");
        return user;
    }
```

- 通过 Bean 工厂方法

```xml
<!-- 实例（Bean）方法实例化 Bean -->
<bean id="userFactory" class="org.xxx.DefaultUserFactory"/>
<bean id="user-by-instance-method" factory-bean="userFactory" factory-method="createUser"/>

public class DefaultUserFactory {
    @Override
    public User createUser() {
        return User.createUser();
    }
}
```

- 通过 FactoryBean 

```java
<!-- FactoryBean实例化 Bean -->
<bean id="user-by-factory-bean" class="org.xxx.UserFactoryBean" />

public class UserFactoryBean implements FactoryBean {

    @Override
    public Object getObject() throws Exception {
        return User.createUser();
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }
}
```

- 通过 ServiceLoaderFactoryBean

```java
<!-- 配置 ServiceLoaderFactoryBean  -->
<bean id="userFactoryServiceLoader"class="x.ServiceLoaderFactoryBean">
    <property name="serviceType" value="org.xx.UserFactory" />
</bean>
    
ServiceLoader<UserFactory> serviceLoader = 	beanFactory.getBean("userFactoryServiceLoader", ServiceLoader.class);

Iterator<UserFactory> iterator = serviceLoader.iterator();
while (iterator.hasNext()) {
    UserFactory userFactory = iterator.next();
    System.out.println(userFactory.createUser());
}
```

- 通过 AutowireCapableBeanFactory#createBean(Class,int,boolean)

```java
// 通过 ApplicationContext 获取 AutowireCapableBeanFactory
AutowireCapableBeanFactory beanFactory = 									applicationContext.getAutowireCapableBeanFactory();

// 创建 UserFactory 对象，通过 AutowireCapableBeanFactory
UserFactory factory = beanFactory.createBean(DefaultUserFactory.class);
System.out.println(factory.createUser());
```

- 通过 BeanDefinitionRegistry#registerBeanDefinition(String,BeanDefinition)

#### 初始化 Spring Bean

- @PostConstruct 标注方法

```java
// 1. 基于 @PostConstruct 注解
@PostConstruct
public void init() {
    System.out.println("@PostConstruct : UserFactory 初始化中...");
}
```

- 实现 InitializingBean 接口的 afterPropertiesSet() 方法

```java
public class DefaultUserFactory implements UserFactory, InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean#afterPropertiesSet() : UserFactory 初始化中...");
    }
}
```

- 自定义初始化方法
  - Java 注解：@Bean(initMethod=”init”) 
  - XML 配置：<bean init-method=”init” ... />
  - Java API：AbstractBeanDefinition#setInitMethodName(String)

```java
@Bean(initMethod = "initUserFactory")
public UserFactory userFactory() {
    return new DefaultUserFactory();
}
public void initUserFactory() {
    System.out.println("自定义初始化方法 initUserFactory() : UserFactory 初始化中...");
}
```

 **注意：初始化方法执行顺序：@PostConstruct 》 afterPropertiesSet() 》自定义初始化方法**



#### 延迟初始化 Spring Bean

- XML 配置：<bean lazy-init=”true” ... />

- Java 注解：@Lazy

```java
@Lazy
public UserFactory userFactory() {
    return new DefaultUserFactory();
}
```

总结：延迟初始化是在容器需要这个 Bean 的时候才去初始化

#### 销毁 Spring Bean

- @PreDestroy 标注方法

```java
@PreDestroy
public void preDestroy() {
    System.out.println("@PreDestroy : UserFactory 销毁中...");
}
```

- 实现 DisposableBean 接口的 destroy() 方法

```java
public class DefaultUserFactory implements UserFactory, DisposableBean {
    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean#destroy() : UserFactory 销毁中...");
    }
}
```

- 自定义销毁方法
  - XML 配置：<bean destroy=”destroy” ... /> 
  - Java 注解：@Bean(destroy=”destroy”) 
  - Java API：AbstractBeanDefinition#setDestroyMethodName(String)

```java
@Bean(destroyMethod = "doDestroy")
public UserFactory userFactory() {
    return new DefaultUserFactory();
}
public void doDestroy() {
    System.out.println("自定义销毁方法 doDestroy() : UserFactory 销毁中...");
}
```

**注意：销毁方法执行顺序：@PreDestroy 》 destroy() 》自定义销毁方法**

#### 垃圾回收 Spring Bean

Bean 垃圾回收

1. 关闭 Spring 容器（应用上下文）
2. 执行 GC
3. Spring Bean 覆盖的 finalize() 方法被回调（可能）



### Spring Bean 作用域

#### Spring Bean 有哪些作用域

![spring-bean作用域](../assets/spring-framework/spring-bean作用域.png)

#### singleton bean 作用域

- 对于 Singleton Bean，Spring IoC容器中只会存在一个共享的 bean 实例。
- Singleton Bean 无论依赖查找还是依赖注入，均为同一个对象。
- Spring 容器管理着 Singleton Bean 的完整生命周期。

#### prototype bean 作用域

- Prototype Bean 无论依赖查找还是依赖注入，均为新生成的对象。
- 如果依赖注入集合类型的对象，Singleton Bean 和 Prototype Bean 均会存在一个， Prototype Bean 有别于其他地方的依赖注入的 Prototype Bean。
- Spring 容器没有办法管理 Prototype Bean 的完整生命周期，也没有办法记录实例的存在。销毁回调方法将不会执行，可以利用 BeanPostProcessor 进行清扫工作。
- Prototype 是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取 bean 的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象。
- **有状态的 bean 应该使用 prototype 作用域，无状态的 bean 则应该使用 singleton 作用域。**

#### request bean 作用域

- Request 作用域针对的是每次的 Http 请求，Spring容器会根据相关的Bean的定义来创建一个全新的 Bean 实例。而且该 Bean 只在当前 request 内是有效的。

配置：

- XML  <bean class="..."scope  = "request"/>
- Java 注解 -@RequestScope 或 @Scope(WebApplicationContext.SCOPE_REQUEST)

实现：

- API - RequestScope

#### session bean 作用域

- 在每个 Session 的生命周期内，Spring 容器会根据 id 为 userPreferences 的 bean 定义创建一个 UserPreferences bean 的新实例。 也就是说，userPreferences bean 的作用域限于Session 范围。和请求作用域 request-scoped bean 类似， 因为每个会话域 session-scoped bean 的范围限于特定的 HTTP Session 内部，所以一个 Session 内的 userPreferences bean也是可以被随意修改， 而不会影响到其他 Session 中的 userPreferences bean。当一个 HTTP Session 最终用完被 JVM 回收时，相关的会话域 session-scoped bean 也被一起回收。

配置：

- XML  <bean class="..."scope  = "session"/>
- Java 注解 -@SessionScope 或 @Scope(WebApplicationContext.SCOPE_SESSION)

实现：

- API - SessionScope

#### application bean 作用域

配置：

- XML  <bean class="..."scope  = "application "/>
- Java 注解 -@ApplicationScope 或 @Scope(WebApplicationContext.SCOPE_APPLICATION)

实现：

- API - SessionScope

#### 自定义 bean 作用域

- 实现 Scope 

  - org.springframework.beans.factory.config.Scope

- 注册 Scope

  - API - ConfigurableBeanFactory#registerScope
  - 配置

  ```xml
  <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer"> <property name="scopes"> 
     <map> <entry key="..."> </entry> </map>
  </property>
  </bean>
  ```


### Spring Bean 生命周期

#### Spring Bean 元信息配置阶段

BeanDefinition 配置

- 面向资源
  - XML 配置
  - Properties 资源配置
- 面向注解
- 面向 API

#### Spring Bean 元信息解析阶段

- 面向资源 BeanDefinition 解析
  - BeanDefinitionReader
  - XML 解析器 - BeanDefinitionParser
- 面向注解 BeanDefinition 解析
  - AnnotatedBeanDefinitionReader （没有继承 BeanDefinitionReader）

#### Spring Bean 注册阶段

- BeanDefinitionRegistry 接口的 registerBeanDefinition(beanName, beanDefinition) 方法

  它的最重要的实现类是 DefaultListableBeanFactory，AnnotationConfigApplicationContext 最终也是引用它来实现。

#### Spring BeanDefinition 合并阶段

在 Spring 中每一个 Bean 基本上都会对应有一个 BeanDefinition，对于它们的实例化和初始化的过程 BeanDefinition 也起着关键性作用，BeanDefinition 存在父 bd（BeanDefinition），子 bd 可以继承它的父 bd 的属性，所以我们在实例化和初始化 Bean 的时候必须考虑父 bd。

在 Bean 的初始化流程中很多地方都出现了与合并 bd 相关的操作，特别是合并 bd 更是多次调用，为什么要这么做呢？

- 很多地方我们需要拿到完整的 bd，不能只是一个子 bd，只要会有缺失，很多判断会有问题，所以每次拿到 bd 都需要合并 bd。
- 因为我们可以在很多地方将 bd 中的属性改变，所以我们不能只是在一个地方去合并 bd，所以在很多地方需要合并 bd。比如我们在 BeanDefinitionRegistryPostProcessor 或者 BeanFactoryPostProcessor 可以很简单的拿到 bd 甚至注册 bd，所以我们可通过这种方式更改 bd 中的属性或者添加父 bd，这个时候之前合并的 bd 就不准确了，所以需要多次合并 bd。

**合并 BeanDefinition 的方法定义在 ConfigurableBeanFactory 接口中，AbstractBeanFactory 抽象类实现了这个接口的 getMergedBeanDefinition() 方法。**

合并流程：

1. 找到父 bd，如果没有找到就直接将自身当作父 bd，找到的话就看看父 bd 有没有父 bd，并且通过 getMergedBeanDefinition() 方法得到父 bd 合并之后的 bd。
2. 合并 bd，通过父 bd 创建出一个 RootBeanDefinition 对象，然后子 bd 覆盖父 bd 创建出来的对象（覆盖属性之类的）。
3. 将合并后的 bd 放入一个 mergedBeanDefinitions 集合中，最后将合并后的 bd 返回。
4. Spring 框架同时提供了一个机会给框架其他部分，或者开发人员用于在 bean 创建过程中，MergedBeanDefinition (mbd) 生成之后，bean 属性填充之前，对该 bean 和该MergedBeanDefinition 做一次回调，相应的回调接口是MergedBeanDefinitionPostProcessor。

#### Spring Bean Class 加载阶段

- ClassLoader 类加载
- Java Security 安全控制
- ConfigurableBeanFactory 临时 ClassLoader

#### Spring Bean 实例化前阶段

**InstantiationAwareBeanPostProcessor** 继承自 **BeanPostProcessor**，是 Spring 非常重要的拓展接口，代表着 bean 的一段生命周期：实例化（**Instantiation**）。 接口方法说明：

1. **postProcessBeforeInstantiation** 调用时机为 bean 实例化（**Instantiation**）之前如果返回了bean 实例, 则会替代原来正常通过 target bean 生成的 bean 的流程。此时 bean 的执行流程将会缩短, 只会执行 BeanPostProcessor#postProcessAfterInitialization 接口完成初始化。
2. **postProcessAfterInstantiation** 调用时机为 bean 实例化（**Instantiation**）之后和任何初始化（**Initialization**）之前。
3. **postProcessProperties** 调用时机为 postProcessAfterInstantiation 执行之后并返回 true, 返回的 PropertyValues 将作用于给定 bean 属性赋值。
4. **postProcessPropertyValues** 已经被标注@Deprecated，后续将会被 postProcessProperties 取代。

**InstantiationAwareBeanPostProcessor** 与 **BeanPostProcessor** 对比：

1. BeanPostProcessor 执行时机为 bean 初始化（**Initialization**）阶段，日常可以拓展该接口对 bean 初始化进行定制化处理。
2. InstantiationAwareBeanPostProcessor 执行时机 bean 实例化（**Instantiation**）阶段，典型用于替换 bean 默认创建方式，例如 aop 通过拓展接口生成代理对应，主要用于基础框架层面。如果日常业务中需要拓展该，Spring 推荐使用缺省适配器类InstantiationAwareBeanPostProcessorAdapter。
3. **所有 bean 创建都会进行回调。**

**InstantiationAwareBeanPostProcessor 的触发点**

1. 从 AbstractAutowireCapableBeanFactory#**createBean** 开始

2. 跟进 AbstractAutowireCapableBeanFactory#**resolveBeforeInstantiation** 会触发 **postProcessorsBeforeInstantiation** 执行，返回 bean **非 null** 则直接执行**beanPostProcessorsAfterInitialization** 进行初始化后置处理，返回的 bean 直接返回容器，生命周期缩短。
3. 跟进 AbstractAutowireCapableBeanFactory#**doCreateBean#populateBean** 先会触发 **postProcessAfterInstantiation** 执行，紧接着**执行 IoC 依赖注入**，然后优先触发 **postProcessProperties** 执行，如没覆盖则触发 postProcessPropertyValues。



- Bean 实例化前阶段
  - InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation

#### Spring Bean 实例化阶段

- 实例化方式
  - 传统实例化方式 
    - 实例化策略-InstantiationStrategy
  - 构造器依赖注入

#### Spring Bean 实例化后阶段

- Bean 属性赋值（Populate）判断

  - InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation

    这个方法的默认返回值为 true，如果返回 false，将防止对此 bean 实例调用任何后续 InstantiationAwareBeanPostProcessor 操作。

#### Spring Bean 属性赋值前阶段

- Bean 属性值元信息
  - PropertyValues
- Bean 属性赋值前回调
  - Spring 1.2 -5.0：InstantiationAwareBeanPostProcessor#postProcessPropertyValues
  - Spring 5.1：InstantiationAwareBeanPostProcessor#**postProcessProperties**

#### Spring Bean Aware 接口回调阶段

**Aware** 是已感知的，意识到的意思。所以这些接口应该是能感知到所有 **Aware** 前面的含义的。它的目的是为了让 bean 获取 Spring 容器的服务。

- Spring Aware 接口（以下也是回调执行顺序）
  - BeanFactory 生命周期
    - BeanNameAware 
    - BeanClassLoaderAware 
    - BeanFactoryAware 
  - ApplicationContext 生命周期
    - EnvironmentAware 
    - EmbeddedValueResolverAware 
    - ResourceLoaderAware 
    - ApplicationEventPublisherAware 
    - MessageSourceAware 
    - ApplicationContextAware

#### Spring Bean 初始化前阶段

- 已完成
  - Bean 实例化
  - Bean 属性赋值
  - Bean Aware 接口回调
- 方法回调
  - BeanPostProcessor#postProcessBeforeInitialization

#### Spring Bean 初始化阶段

- Bean 初始化（Initialization）
  - @PostConstruct 标注方法（依赖于注解驱动，在 BeanFactory 中可以使用 CommonAnnotationBeanPostProcessor 触发）
  - 实现 InitializingBean 接口的 afterPropertiesSet() 方法
  - 自定义初始化方法

#### Spring Bean 初始化后阶段

- 方法回调
  - BeanPostProcessor#postProcessAfterInitialization

#### Spring Bean 初始化完成阶段

- 方法回调
  - Spring 4.1 +：SmartInitializingSingleton#afterSingletonsInstantiated

#### Spring Bean 销毁前阶段

- 方法回调
  - DestructionAwareBeanPostProcessor#postProcessBeforeDestruction

#### Spring Bean 销毁阶段

- Bean 销毁（Destroy）
  - @PreDestroy 标注方法
  - 实现 DisposableBean 接口的 destroy() 方法
  - 自定义销毁方法

#### Spring Bean 垃圾收集	

- Bean 垃圾回收（GC）
  - 关闭 Spring 容器（应用上下文）
  - 执行 GC 
  - Spring Bean 覆盖的 finalize() 方法被回调

#### BeanPostProcessor 的使用场景有哪些？

BeanPostProcessor 提供 Spring Bean 初始化前和初始化后的生命周期回调，分别对应postProcessBeforeInitialization 以及 postProcessAfterInitialization 方法，允许对关心的 Bean 进行扩展 ，甚至是替换。其中，ApplicationContext 相关的 Aware 回调也是基于 BeanPostProcessor 实现，即 ApplicationContextAwareProcessor。

#### BeanFactoryPostProcessor 与 BeanPostProcessor 的区别

BeanFactoryPostProcessor 是 Spring BeanFactory（实际为 ConfigurableListableBeanFactory）的后置处理器，用于扩展 BeanFactory，或通过 BeanFactory 进行依赖查找和依赖注入。 BeanFactoryPostProcessor 必须有 Spring ApplicationContext 执行，BeanFactory 无法与其直接交互。 而 BeanPostProcessor 则直接与 BeanFactory 关联，属于 N 对 1 的关系。

#### BeanFactory 是怎样处理 Bean 生命周期？

BeanFactory 的默认实现为 DefaultListableBeanFactory，其中 Bean 生命周期与方法映射如下： 

- BeanDefinition 注册阶段-registerBeanDefinition 
- BeanDefinition 合并阶段-getMergedBeanDefinition 
- Bean 实例化前阶段-resolveBeforeInstantiation 
- Bean 实例化阶段-createBeanInstance 
- Bean 实例化后阶段-populateBean 
- Bean 属性赋值前阶段-populateBean 
- Bean 属性赋值阶段-populateBean 
- Bean Aware 接口回调阶段-initializeBean 
- Bean 初始化前阶段-initializeBean 
- Bean 初始化阶段-initializeBean 
- Bean 初始化后阶段-initializeBean 
- Bean 初始化完成阶段-preInstantiateSingletons 
- Bean 销毁前阶段-destroyBean 
- Bean 销毁阶段-destroyBean