入口注解

@SpringBootConfiguration

下属有3个注解

@SpringBootConfiguration，@EnableAutoConfiguration，@ComponentScan

***\*@SpringBootConfiguration\****

发现其实就是一个@Configuration注解，作用就是将配置了这个注解的类加载到ioc容器。主要是为了替换掉xml配置文件。

***\*@EnableAutoConfiguration\****

加上此注解就能开始自动装配，Spring会试图在你的classpath下找到所有配置的Bean然后进行装配。当然装配Bean时，会根据若干个(Conditional)定制规则来进行初始化。我们看一下它的源码。

AutoConfigurationImportSelector，实现了DeferredImportSelector接口，这个接口继承了ImportSelector。ImportSelector接口主要是为了导入@Configuration的配置项，而DeferredImportSelector是延期导入，当所有的@Configuration都处理过后才会执行。

AutoConfigurationImportSelector的selectImport，先判断是否自动装配

getCandidateConfigurations

这个方法表明了是从

META-INF/spring.factories

中查找自动装配的类

***\*@ComponentScan\****

这个注解是大家接触得最多的了，相当于xml配置文件中的<context:component-scan>。它的主要作用就是扫描指定路径下的标识了需要装配的类，自动装配到spring的Ioc容器中。

标识需要装配的类的形式主要是：@Component、@Repository、@Service、@Controller这类的注解标识的类。ComponentScan默认会扫描当前package下的的所有加了相关注解标识的类到IoC容器中











https://www.cnblogs.com/theRhyme/p/11057233.html#_label8

# [SPRINGBOOT启动流程及其原理](https://www.cnblogs.com/theRhyme/p/11057233.html)





- [SpringBoot启动原理精简版](https://www.cnblogs.com/theRhyme/p/11057233.html#_label0)

- [Spring Boot、Spring MVC 和 Spring 有什么区别？](https://www.cnblogs.com/theRhyme/p/11057233.html#_label1)

- [一 springboot启动原理及相关流程概览](https://www.cnblogs.com/theRhyme/p/11057233.html#_label2)

- [二 springboot的启动类入口](https://www.cnblogs.com/theRhyme/p/11057233.html#_label3)

- [三 单单是SpringBootApplication接口用到了这些注解](https://www.cnblogs.com/theRhyme/p/11057233.html#_label4)

- - [　　1）@Configuration注解](https://www.cnblogs.com/theRhyme/p/11057233.html#_lab2_4_0)

  - [　　2） @ComponentScan注解](https://www.cnblogs.com/theRhyme/p/11057233.html#_lab2_4_1)

  - [3) @EnableAutoConfiguration](https://www.cnblogs.com/theRhyme/p/11057233.html#_lab2_4_2)

  - - [AutoConfigurationPackage注解：](https://www.cnblogs.com/theRhyme/p/11057233.html#_label3_4_2_0)
    - [Import(AutoConfigurationImportSelector.class)注解](https://www.cnblogs.com/theRhyme/p/11057233.html#_label3_4_2_1)

  - [SpringFactoriesLoader详解](https://www.cnblogs.com/theRhyme/p/11057233.html#_lab2_4_3)

- [四 springboot启动流程概览图](https://www.cnblogs.com/theRhyme/p/11057233.html#_label5)

- [五 深入探索SpringApplication执行流程](https://www.cnblogs.com/theRhyme/p/11057233.html#_label6)

- [简单了解下Bean的生命周期](https://www.cnblogs.com/theRhyme/p/11057233.html#_label7)

- [BeanFactory 和ApplicationContext的区别](https://www.cnblogs.com/theRhyme/p/11057233.html#_label8)

- [SpringMVC处理请求的流程](https://www.cnblogs.com/theRhyme/p/11057233.html#_label9)

- [BEANFACTORY和FACTORYBEAN的区别与联系](https://www.cnblogs.com/theRhyme/p/11057233.html#_label10)

- [Bean的循环依赖](https://www.cnblogs.com/theRhyme/p/11057233.html#_label11)

- [@Import注解介绍](https://www.cnblogs.com/theRhyme/p/11057233.html#_label12)

- [同一个类中调用 @Transaction注解的方法会有事务效果吗？](https://www.cnblogs.com/theRhyme/p/11057233.html#_label13)

- [来源](https://www.cnblogs.com/theRhyme/p/11057233.html#_label14)

# SpringBoot启动原理精简版

https://www.cnblogs.com/theRhyme/p/how-does-springboot-start.html

 

# Spring Boot、Spring MVC 和 Spring 有什么区别？

 

分别描述各自的特征：

 

Spring 框架就像一个家族，有众多衍生产品例如 boot、security、jpa等等；但他们的基础都是Spring 的ioc和 aop，ioc 提供了依赖注入的容器， aop解决了面向切面编程，然后在此两者的基础上实现了其他延伸产品的高级功能。

 

Spring MVC提供了一种轻度耦合的方式来开发web应用；它是Spring的一个模块，是一个web框架；通过DispatcherServlet, ModelAndView 和 View Resolver，开发web应用变得很容易；解决的问题领域是网站应用程序或者服务开发——URL路由、Session、模板引擎、静态Web资源等等。

 

Spring Boot实现了auto-configuration**自动配置**（另外三大神器actuator监控，cli命令行接口，starter依赖），降低了项目搭建的复杂度。它主要是为了解决使用Spring框架需要进行大量的配置太麻烦的问题，所以它并不是用来替代Spring的解决方案，而是和Spring框架紧密结合用于提升Spring开发者体验的工具；同时它集成了大量常用的第三方库配置(例如Jackson, JDBC, Mongo, Redis, Mail等等)，Spring Boot应用中这些第三方库几乎可以零配置的开箱即用(out-of-the-box)。

 

所以，用最简练的语言概括就是:

 

Spring 是一个“引擎”;

 

Spring MVC 是基于Spring的一个 MVC 框架;

 

Spring Boot 是基于Spring4的条件注册的一套快速开发整合包。

 

# 一 springboot启动原理及相关流程概览

　　springboot是基于spring的新型的轻量级框架，最厉害的地方当属***自动配置。***那我们就可以根据启动流程和相关原理来看看，如何实现传奇的自动配置。

![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190707171658626-1389392187.png)

 

 

# 二 springboot的启动类入口

用过springboot的技术人员很显而易见的两者之间的差别就是视觉上很直观的：springboot有自己独立的启动类（独立程序）

```
@SpringBootApplication``public` `class` `Application {``  ``public` `static` `void` `main(String[] args) {``    ``SpringApplication.run(Application.``class``, args);``  ``}``}
```

从上面代码可以看出，Annotation定义（@SpringBootApplication）和类定义（SpringApplication.run）最为耀眼，所以要揭开SpringBoot的神秘面纱，我们要从这两位开始就可以了。

 

# 三 单单是SpringBootApplication接口用到了这些注解

```java
@Target(ElementType.TYPE)// 注解的适用范围，其中TYPE用于描述类、接口（包括包注解类型）或enum声明
@Retention(RetentionPolicy.RUNTIME) // 注解的生命周期，保留到class文件中（三个生命周期）
@Documented// 表明这个注解应该被javadoc记录
@Inherited// 子类可以继承该注解
@SpringBootConfiguration// 继承了Configuration，表示当前是注解类
@EnableAutoConfiguration// 开启springboot的注解功能，springboot的四大神器之一，其借助@import的帮助
@ComponentScan(excludeFilters = {// 扫描路径设置
@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter. class) })
public @interface SpringBootApplication {...}　　
```

![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190709114128801-171612088.png)

 

　　

 

在其中比较重要的有三个注解，分别是：

　　1）@SpringBootConfiguration // 继承了Configuration，表示当前是注解类

　　2）@EnableAutoConfiguration // 开启springboot的注解功能，springboot的四大神器之一，其借助@import的帮助

　　3）@ComponentScan(excludeFilters = { // 扫描路径设置（具体使用待确认）

 　接下来对三个注解一一详解，增加对springbootApplication的理解：

## 　　1）@Configuration注解

　　按照原来xml配置文件的形式，在springboot中我们大多用配置类来解决配置问题

　　　配置bean方式的不同：　

　　　　a）xml配置文件的形式配置bean

```
<?xml version=``"1.0"` `encoding=``"UTF-8"``?>``<beans xmlns=``"http://www.springframework.org/schema/beans"``xmlns:xsi=``"http://www.w3.org/2001/XMLSchema-instance"``xsi:schemaLocation=``"http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"``default``-lazy-init=``"true"``>``<!--bean定义-->``</beans>
```

　　b）java configuration的配置形式配置bean

```
@Configuration``public` `class` `MockConfiguration{``  ``//bean定义``}
```

　　

　　注入bean方式的不同：

　　　　a）xml配置文件的形式注入bean

```
<bean id=``"mockService"` `class``=``"..MockServiceImpl"``>``...``</bean>
```

　　b）java configuration的配置形式注入bean

```
@Configuration``public` `class` `MockConfiguration{``  ``@Bean``  ``public` `MockService mockService(){``    ``return` `new` `MockServiceImpl();``  ``}``}
```

　　

任何一个标注了@Bean的方法，其返回值将作为一个bean定义注册到Spring的IoC容器，方法名将默认成该bean定义的id。

　　表达**bean之间依赖关系**的不同：

　　　　a）xml配置文件的形式表达依赖关系

```
<bean id=``"mockService"` `class``=``"..MockServiceImpl"``>``　　<propery name =``"dependencyService"` `ref=``"dependencyService"` `/>``</bean>``<bean id=``"dependencyService"` `class``=``"DependencyServiceImpl"``></bean>
```

　　　　b）java configuration配置的形式表达**依赖关系（重点）**

　　　　**如果一个bean A的定义依赖其他bean B,则直接调用对应的JavaConfig类中依赖bean B的创建方法就可以了。**

```
@Configuration``public` `class` `MockConfiguration{``　　``@Bean``　　``public` `MockService mockService(){``  ``　　``return` `new` `MockServiceImpl(dependencyService());``　　}``　　``@Bean``　　``public` `DependencyService dependencyService(){``  ``　　``return` `new` `DependencyServiceImpl();``　　}``}
```

　　

## 　　2） @ComponentScan注解

　　作用：a）对应xml配置中的元素；

　　　　　b） **（重点）**ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义;

　　　　　c) 将这些bean定义加载到**IoC**容器中.

　　 我们可以通过basePackages等属性来**细粒度**的定制@ComponentScan自动扫描的范围，如果不指定，则**默认**Spring框架实现会从声明@ComponentScan所在类的package进行扫描。

　　`注：所以SpringBoot的启动类最好是放在root ``package``下，因为默认不指定basePackages`

 　

## 3) **@EnableAutoConfiguration**

　　　　此注解顾名思义是可以自动配置，所以应该是springboot中最为重要的注解。

　　　　在spring框架中就提供了各种以@Enable开头的注解，例如： @EnableScheduling、@EnableCaching、@EnableMBeanExport等； @EnableAutoConfiguration的理念和做事方式其实一脉相承简单概括一下就是，借助@Import的支持，收集和注册特定场景相关的bean定义。　　

- - 　　@EnableScheduling是通过@Import将Spring调度框架相关的bean定义都加载到IoC容器【定时任务、时间调度任务】
  - 　　@EnableMBeanExport是通过@Import将JMX相关的bean定义加载到IoC容器【监控JVM运行时状态】

　　　　 @EnableAutoConfiguration也是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IoC容器。

　　　 　@EnableAutoConfiguration作为一个复合Annotation,其自身定义关键信息如下：

 

```java
@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
【重点注解】
 @Import (AutoConfigurationImportSelector.class)【重点注解】
public @interface EnableAutoConfiguration {...}
```

 

　　

其中最重要的两个注解已经标注：1、@AutoConfigurationPackage【重点注解】2、@Import(AutoConfigurationImportSelector.class)【重点注解】

  当然还有其中比较重要的一个类就是：AutoConfigurationImportSelector.class

### AutoConfigurationPackage注解：

```java
@Target(ElementType.TYPE)
@Retention``(RetentionPolicy.RUNTIME)@Documented``@Inherited``@Import``(AutoConfigurationPackages.Registrar.``class``)``public` `@interface` `AutoConfigurationPackage {` `}
```

通过@Import(AutoConfigurationPackages.**Registrar**.class)

```
static` `class` `Registrar ``implements` `ImportBeanDefinitionRegistrar, DeterminableImports {` `    ``@Override``    ``public` `void` `registerBeanDefinitions(AnnotationMetadata metadata,``        ``BeanDefinitionRegistry registry) {``      ``register(registry, ``new` `PackageImport(metadata).getPackageName());``    ``}` `    ``……` `  ``}
```

注册**当前启动类的根package**；

注册org.springframework.boot.autoconfigure.**AutoConfigurationPackages**的**BeanDefinition**。

 

### Import(AutoConfigurationImportSelector.class)注解

![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190707175201406-322411811.png)

 

（重点）可以从图中看出 **AutoConfigurationImportSelector 实现**了 DeferredImportSelector 从 ImportSelector继承的方法：**selectImports**。

```
@Override``  ``public` `String[] selectImports(AnnotationMetadata annotationMetadata) {``    ``if` `(!isEnabled(annotationMetadata)) {``      ``return` `NO_IMPORTS;``    ``}``    ``AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader``        ``.loadMetadata(``this``.beanClassLoader);``    ``AnnotationAttributes attributes = getAttributes(annotationMetadata);``    ``List<String> configurations = getCandidateConfigurations(annotationMetadata,``        ``attributes);``    ``configurations = removeDuplicates(configurations);``    ``Set<String> exclusions = getExclusions(annotationMetadata, attributes);``    ``checkExcludedClasses(configurations, exclusions);``    ``configurations.removeAll(exclusions);``    ``configurations = filter(configurations, autoConfigurationMetadata);``    ``fireAutoConfigurationImportEvents(configurations, exclusions);``    ``return` `StringUtils.toStringArray(configurations);``  ``}
```

　　

`第9行List<String> configurations = getCandidateConfigurations(annotationMetadata,`attributes);其实是去加载各个组件jar下的  public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";外部文件。

该方法在springboot启动流程——bean实例化前被执行，返回要实例化的类信息列表；

如果获取到类信息，spring可以通过类加载器将类加载到jvm中，现在我们已经通过spring-boot的starter依赖方式依赖了我们需要的组件，那么这些组件的类信息在select方法中就可以被获取到。

 

```
protected` `List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {`` ``List<String> configurations = SpringFactoriesLoader.loadFactoryNames(``this``.getSpringFactoriesLoaderFactoryClass(), ``this``.getBeanClassLoader());`` ``Assert.notEmpty(configurations, ``"No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct."``);`` ``return` `configurations;`` ``}
```

 

　　

 

其返回一个自动配置类的类名列表，方法调用了loadFactoryNames方法，查看该方法

 

```
public` `static` `List<String> loadFactoryNames(Class<?> factoryClass, ``@Nullable` `ClassLoader classLoader) {`` ``String factoryClassName = factoryClass.getName();`` ``return` `(List)loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());`` ``}
```

 

自动配置器会跟根据传入的factoryClass.getName()到项目系统路径下所有的spring.factories文件中找到相应的key，从而加载里面的类。

 

这个外部文件，有很多自动配置的类。如下：

![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190707180956760-1536038503.png)

 

**（重点）**其中，最关键的要属@Import(AutoConfigurationImportSelector.class)，借助AutoConfigurationImportSelector，@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件(spring.factories)的**bean定义**（如Java Config@Configuration配置）都加载到当前SpringBoot创建并使用的IoC容器。 

 

## SpringFactoriesLoader详解

借助于Spring框架**原有**的一个工具类：**SpringFactoriesLoader**的支持，@EnableAutoConfiguration可以智能的自动配置功效才得以大功告成！

SpringFactoriesLoader属于Spring框架私有的一种扩展方案，其主要功能就是从**指定的配置文件META-INF/spring.factories**加载配置,**加载工厂类**。

SpringFactoriesLoader为Spring工厂加载器，该对象提供了loadFactoryNames方法，入参为factoryClass和classLoader即需要传入**工厂类**名称和对应的类加载器，方法会根据指定的classLoader，加载该类加器搜索路径下的指定文件，即spring.factories文件；

传入的工厂类为接口，而文件中对应的类则是接口的实现类，或最终作为实现类。

```
public` `abstract` `class` `SpringFactoriesLoader {``//...``　　``public` `static` `<T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {``　　　　...``　　}`` ` ` ` `　　``public` `static` `List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {``　　　　....``　　}``}
```

配合@EnableAutoConfiguration使用的话，它更多是提供一种配置查找的功能支持，即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfigure.EnableAutoConfiguration作为查找的Key,获取对应的一组@Configuration类　　

![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190707180956760-1536038503.png)

上图就是从SpringBoot的autoconfigure依赖包中的META-INF/spring.factories配置文件中摘录的一段内容，可以很好地说明问题。

（重点）所以，@EnableAutoConfiguration自动配置的魔法其实就变成了：

从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableAutoConfiguration对应的**配置项**通过**反射（Java Refletion）**实例化为对应的标注了**@Configuration**的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。

 

![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190708145522504-1677532764.png)

 

 

 

# 四 springboot启动流程概览图

 ![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190707203656740-1805253848.png)

![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190707203726850-831777585.png)

 

 

 

# 五 深入探索SpringApplication执行流程

 

 ![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190716164535793-477823131.png)

 

![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190717114952544-1705075996.jpg)

 ![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190717115009703-517805589.jpg)

 

 

```
public` `class` `EventPublishingRunListener ``implements` `SpringApplicationRunListener, Ordered
```

SpringApplicationRunListener接口的唯一实现是EventPublishingRunListener；

实现了方法，下面是部分方法源码：

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {

 private final SpringApplication application;

 private final String[] args;

 private final SimpleApplicationEventMulticaster initialMulticaster;

 public EventPublishingRunListener(SpringApplication application, String[] args) {
  this.application = application;
  this.args = args;
  this.initialMulticaster = new SimpleApplicationEventMulticaster();
  for (ApplicationListener<?> listener : application.getListeners()) {
   this.initialMulticaster.addApplicationListener(listener);
  }
 }

 @Override
 public int getOrder() {
  return 0;
 }

    // SpringBoot启动事件
 @Override
 public void starting() {
  this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
 }

    // 创建和配置环境
 @Override
 public void environmentPrepared(ConfigurableEnvironment environment) {
  this.initialMulticaster
    .multicastEvent(new ApplicationEnvironmentPreparedEvent(this.application, this.args, environment));
 }

    // 准备ApplicationContext
 @Override
 public void contextPrepared(ConfigurableApplicationContext context) {
  this.initialMulticaster
    .multicastEvent(new ApplicationContextInitializedEvent(this.application, this.args, context));
 }

    // 发布ApplicationContext已经refresh事件，标志着ApplicationContext初始化完成
 @Override
 public void contextLoaded(ConfigurableApplicationContext context) {
  for (ApplicationListener<?> listener : this.application.getListeners()) {
   if (listener instanceof ApplicationContextAware) {
    ((ApplicationContextAware) listener).setApplicationContext(context);
   }
   context.addApplicationListener(listener);
  }
  this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
 }

    // SpringBoot已启动事件
 @Override
 public void started(ConfigurableApplicationContext context) {
  context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context));
  AvailabilityChangeEvent.publish(context, LivenessState.CORRECT);
 }

    // "SpringBoot现在可以处理接受的请求"事件
 @Override
 public void running(ConfigurableApplicationContext context) {
  context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context));
  AvailabilityChangeEvent.publish(context, ReadinessState.ACCEPTING_TRAFFIC);
 }

 @Override
 public void failed(ConfigurableApplicationContext context, Throwable exception) {
  ApplicationFailedEvent event = new ApplicationFailedEvent(this.application, this.args, context, exception);
  if (context != null && context.isActive()) {
   // Listeners have been registered to the application context so we should
   // use it at this point if we can
   context.publishEvent(event);
  }
  else {
   // An inactive context may not have a multicaster so we use our multicaster to
   // call all of the context's listeners instead
   if (context instanceof AbstractApplicationContext) {
    for (ApplicationListener<?> listener : ((AbstractApplicationContext) context)
      .getApplicationListeners()) {
     this.initialMulticaster.addApplicationListener(listener);
    }
   }
   this.initialMulticaster.setErrorHandler(new LoggingErrorHandler());
   this.initialMulticaster.multicastEvent(event);
  }
 }

 private static class LoggingErrorHandler implements ErrorHandler {

  private static final Log logger = LogFactory.getLog(EventPublishingRunListener.class);

  @Override
  public void handleError(Throwable throwable) {
   logger.warn("Error calling ApplicationEventListener", throwable);
  }

 }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

 

　　

# 简单了解下Bean的生命周期

![img](https://img2018.cnblogs.com/blog/1158841/201907/1158841-20190724210248256-956610315.png)

 

一个Bean的构造函数初始化时是最先执行的，这个时候，**bean属性还没有被注入**；

@PostConstruct注解的方法优先于InitializingBean的afterPropertiesSet执行，这时Bean的属性竟然被注入了；

spring很多组件的初始化都放在afterPropertiesSet做,想和spring一起启动，可以放在这里启动;

spring为bean提供了两种初始化bean的方式，实现InitializingBean接口，实现afterPropertiesSet方法，或者在配置文件中同过init-method指定，两种方式可以同时使用；

实现InitializingBean接口是直接调用afterPropertiesSet方法，比通过反射调用init-method指定的方法效率相对来说要高点；但是init-method方式消除了对spring的依赖；

如果调用afterPropertiesSet方法时出错，则不调用init-method指定的方法。

Bean在实例化的过程中：

Constructor > @PostConstruct > InitializingBean > init-method

 

# **BeanFactory 和ApplicationContext的区别**

BeanFactory和ApplicationContext都是接口，并且ApplicationContext间接继承了BeanFactory。

BeanFactory是Spring中最底层的接口，提供了最简单的容器的功能，只提供了实例化对象和获取对象的功能，而ApplicationContext是Spring的一个更高级的容器，提供了更多的有用的功能。 

ApplicationContext提供的额外的功能：获取Bean的详细信息(如定义、类型)、国际化的功能、统一加载资源的功能、强大的事件机制、对Web应用的支持等等。

加载方式的区别：BeanFactory采用的是**延迟加载的**形式来注入Bean；ApplicationContext则相反的，它是在Ioc**启动**时就一次性创建**所有的Bean,**好处是可以马上发现Spring配置文件中的错误，坏处是造成浪费。

 

```
public` `interface` `ApplicationContext ``extends` `EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,``    ``MessageSource, ApplicationEventPublisher, ResourcePatternResolver
```

　　

# **SpringMVC处理请求的流程**

1、用户发送请求至前端控制器DispatcherServlet

2、DispatcherServlet收到请求调用HandlerMapping处理器映射器。

3、处理器映射器根据请求url找到具体的处理器，生成处理器对象Handler及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

4、DispatcherServlet通过HandlerAdapter（让Handler实现更加灵活）处理器适配器调用处理器

5、执行处理器(Controller，也叫后端控制器)。

6、Controller执行完成返回ModelAndView（连接业务逻辑层和展示层的桥梁，持有一个ModelMap对象和一个View对象）。

7、HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

8、DispatcherServlet将ModelAndView传给ViewReslover视图解析器

9、ViewReslover解析后返回具体View

10、DispatcherServlet对View进行渲染视图（将ModelMap模型数据填充至视图中）。

11、DispatcherServlet响应用户

 

# [BEANFACTORY和FACTORYBEAN的区别与联系](https://www.cnblogs.com/theRhyme/p/9139673.html)

1. 两者都是接口；
2. BeanFactory主要是用来创建Bean和获得Bean的；
3. FactoryBean跟普通Bean不同，其返回的对象不是指定类的一个实例，而是该FactoryBean的**getObject**方法所返回的对象；
4. 通过BeanFactory和beanName获取bean时，如果beanName不加&则获取到对应bean的实例；如果beanName加上&，则获取到FactoryBean本身的实例
5. FactoryBean 通常是用来**创建****比较复杂的bean****(如创建mybatis的SqlSessionFactory很复杂)**，一般的bean 直接用xml配置即可，但如果创建一个bean的创建过程中涉及到很多**其他的bean** 和复杂的逻辑，用xml配置比较困难，这时可以考虑用FactoryBean。

 

# Bean的循环依赖

https://blog.csdn.net/itmrchen/article/details/90201279

对于Spring中Bean的管理，下图一目了然： 

![img](https://img2018.cnblogs.com/blog/1158841/201910/1158841-20191031165056809-1219399286.png)

 

 

先调用构造函数进行实例化，然后填充属性，再接着进行其他附加操作和初始化，正是这样的生命周期，才有了Spring的解决循环依赖，这样的解决机制是根据Spring框架内定义的三级缓存来实现的，也就是说：三级缓存解决了Bean之间的循环依赖。我们从源码中来说明。

先来看Spring中Bean工厂是怎么获取Bean的（AbstractBeanFactory中）：
![img](https://img2018.cnblogs.com/blog/1158841/201910/1158841-20191031165137246-994866848.png)

 

 ![img](https://img2018.cnblogs.com/blog/1158841/201910/1158841-20191031165143002-1751267656.png)

 

 ![img](https://img2018.cnblogs.com/blog/1158841/201910/1158841-20191031165150173-1161185844.png)

 

 ![img](https://img2018.cnblogs.com/blog/1158841/201910/1158841-20191031165155277-1290927126.png)

 

 一级一级向下寻找，找出了前面提到的三级缓存，也就是三个Map集合类：

singletonObjects：第一级缓存，里面放置的是已经实例化好的单例对象；

earlySingletonObjects：第二级缓存，里面存放的是提前曝光的单例对象；

singletonFactories：第三级缓存，里面存放的是将要被实例化的对象的对象工厂。

所以当一个Bean调用构造函数进行实例化后，即使set属性还未填充，就可以通过**三级缓存**向外暴露依赖的引用值进行set（所以循环依赖问题的解决也是基于Java的引用传递），这也说明了另外一点，基于构造函数的注入，如果有循环依赖，Spring是不能够解决的。

还要说明一点，Spring默认的Bean Scope是**单例**的，而三级缓存中都包含singleton，可见是对于单例Bean之间的循环依赖的解决，Spring是通过三级缓存来实现的。

 

# @Import注解介绍

https://www.cnblogs.com/theRhyme/p/13887223.html 

 

# 同一个类中调用 @Transaction注解的方法会有事务效果吗？

没有，可以Autowired注入自己，然后再调用注入的类中的方法，即自己依赖自己，循环依赖；

这里在一个内部调用应该是相当于单纯的调用方法this.methodName()，并没有AOP代理。

 

 

# 来源

https://www.cnblogs.com/l3306/p/10752907.html

https://www.toutiao.com/a6704225799840465422/?timestamp=1560991132&app=news_article&group_id=6704225799840465422&req_id=201906200838520100220620357346BE4

https://blog.csdn.net/qq_41063141/article/details/83239941

https://www.toutiao.com/a6646351921307189764/?tt_from=weixin&utm_campaign=client_share&wxshare_count=1&timestamp=1547529100&app=news_article_lite&utm_source=weixin&iid=57655139443&utm_medium=toutiao_android&group_id=6646351921307189764

https://blog.51cto.com/luecsc/1964056

https://www.cnblogs.com/javazhiyin/p/xiaozhi.html

https://blog.csdn.net/qq_38409944/article/details/82663361

https://www.cnblogs.com/theRhyme/p/how-does-springboot-start.html