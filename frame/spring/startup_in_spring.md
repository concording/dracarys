# Guide To Running Logic on Startup in Spring

## **1\. Introduction**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#Introduction)

In this article we'll focus on how to **run logic at the startup of a Spring application**.

## **2\. Running Logic on Startup**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#Running)

Running logic during/after Spring application's startup is a common scenario, but one that causes multiple problems.

In order to benefit from Inverse of Control, we naturally need to renounce partial control over the application's flow to the container – which is why instantiation, setup logic on startup, etc needs special attention.

We can't simply include our logic in the beans' constructors or call methods after instantiation of any object; we are simply not in control during those processes.

Let's look at the real-life example:

```java
@Component
public class InvalidInitExampleBean {

    @Autowired
    private Environment env;

    public InvalidInitExampleBean() {
        env.getActiveProfiles();
    }
}
```

Here, we're trying to access an _autowired_ field in the constructor. When the constructor is called, the Spring bean is not yet fully initialized. This is problematic because **calling not yet initialized fields will of course result in _NullPointerException_s**.

Spring gives us a few ways of managing this situation.

### **2.1\. The _@PostConstruct_ Annotation**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#1-the-postconstruct-annotation)

Javax's_ @PostConstruct_ annotation can be used for annotating a method that should be run **once immediately after the bean's initialization**. Keep in mind that the annotated method will be executed by Spring even if there is nothing to inject.

Here's _@PostConstruct_ in action:

```java
@Component
public class PostConstructExampleBean {

    private static final Logger LOG 
      = Logger.getLogger(PostConstructExampleBean.class);

    @Autowired
    private Environment environment;

    @PostConstruct
    public void init() {
        LOG.info(Arrays.asList(environment.getDefaultProfiles()));
    }
}
```

In the example above you can see that the _Environment_ instance was safely injected and then called in the _@PostConstruct_ annotated method without throwing a _NullPointerException_.

### **2.2\. The _InitializingBean _Interface**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#2-the-initializingbean-interface)

The_ InitializingBean_ approach works pretty similarly to the previous one. Instead of annotating a method, you need to implement the _InitializingBean_ interface and the _afterPropertiesSet()_ method.

Here you can see the previous example implemented using the _InitializingBean_ interface:

```java
@Component
public class InitializingBeanExampleBean implements InitializingBean {

    private static final Logger LOG 
      = Logger.getLogger(InitializingBeanExampleBean.class);

    @Autowired
    private Environment environment;

    @Override
    public void afterPropertiesSet() throws Exception {
        LOG.info(Arrays.asList(environment.getDefaultProfiles()));
    }
}
```

### **2.3\. An _ApplicationListener_**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#3-an-applicationlistener)

This approach can be used for **running logic after the Spring context has been initialized**, so we are not focusing on any particular bean, but waiting for all of them to initialize.

In order to achieve this you need to create a bean that implements the _ApplicationListener<ContextRefreshedEvent>_ interface:

```java
@Component
public class StartupApplicationListenerExample implements 
  ApplicationListener<ContextRefreshedEvent> {

    private static final Logger LOG 
      = Logger.getLogger(StartupApplicationListenerExample.class);

    public static int counter;

    @Override public void onApplicationEvent(ContextRefreshedEvent event) {
        LOG.info("Increment counter");
        counter++;
    }
}

```

The same results can be achieved by using the newly-introduced _@EventListener_ annotation:

```java
@Component
public class EventListenerExampleBean {

    private static final Logger LOG 
      = Logger.getLogger(EventListenerExampleBean.class);

    public static int counter;

    @EventListener
    public void onApplicationEvent(ContextRefreshedEvent event) {
        LOG.info("Increment counter");
        counter++;
    }
}
```

In this example we chose the _ContextRefreshedEvent._ Make sure to pick an appropriate event that suits your needs.

### **2.4\. The _@Bean_ Initmethod Attribute**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#4-the-bean-initmethod-attribute)

The_ initMethod_ property can be used to execute a method after a bean's initialization.

Here's what a bean looks like:

```java
public class InitMethodExampleBean {

    private static final Logger LOG = Logger.getLogger(InitMethodExampleBean.class);

    @Autowired
    private Environment environment;

    public void init() {
        LOG.info(Arrays.asList(environment.getDefaultProfiles()));
    }
}
```

You can notice that there are no special interfaces implemented nor any special annotations used.

Then, we can define the bean using the _@Bean_ annotation:

```java
@Bean(initMethod="init")
public InitMethodExampleBean initMethodExampleBean() {
    return new InitMethodExampleBean();
}
```

And this is how a bean definition looks in an XML config:

```xml
<bean id="initMethodExampleBean"
  class="com.baeldung.startup.InitMethodExampleBean"
  init-method="init">
</bean>
```

### **2.5\. Constructor Injection**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#5-constructor-injection)

If you are injecting fields using Constructor Injection, you can simply include your logic in a constructor:

```java
@Component 
public class LogicInConstructorExampleBean {

    private static final Logger LOG 
      = Logger.getLogger(LogicInConstructorExampleBean.class);

    private final Environment environment;

    @Autowired
    public LogicInConstructorExampleBean(Environment environment) {
        this.environment = environment;
        LOG.info(Arrays.asList(environment.getDefaultProfiles()));
    }
}
```

### **2.6\. Spring Boot _CommandLineRunner_**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#6-spring-boot-commandlinerunner)

Spring boot provides a _CommandLineRunner_ interface with a callback _run()_ method which can be invoked at application startup after the Spring application context is instantiated.

Let's look at an example:

```java
@Component
public class CommandLineAppStartupRunner implements CommandLineRunner {
    private static final Logger LOG =
      LoggerFactory.getLogger(CommandLineAppStartupRunner.class);

    public static int counter;

    @Override
    public void run(String...args) throws Exception {
        LOG.info("Increment counter");
        counter++;
    }
}
```

**Note**: As mentioned in the [documentation](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html), multiple _CommandLineRunner_ beans can be defined within the same application context and can be ordered using the _@Ordered_ interface or _@Order_ annotation.

### **2.7\. Spring Boot _ApplicationRunner_**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#7-spring-boot-applicationrunner)

Similar to _CommandLineRunner,_ Spring boot also provides an _ApplicationRunner _interface with a _run()_ method to be invoked at application startup. However, instead of raw _String_ arguments passed to the callback method, we have an instance of the [_ApplicationArguments_ ](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationArguments.html)class.

The _ApplicationArguments _interface has methods to get argument values that are options and plain argument values. An argument that is prefixed with – – is an option argument.

Let's look at an example:

```java
@Component
public class AppStartupRunner implements ApplicationRunner {
    private static final Logger LOG =
      LoggerFactory.getLogger(AppStartupRunner.class);

    public static int counter;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        LOG.info("Application started with option names : {}", 
          args.getOptionNames());
        LOG.info("Increment counter");
        counter++;
    }
}
```

## **3\. Combining Mechanisms**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#combining-mechanisms)

In order to achieve full control over your beans, you might want to combine the above mechanisms together.

The order of execution is as follows:

1.  The constructor
2.  the_ @PostConstruct_ annotated methods
3.  the InitializingBean's _afterPropertiesSet()_ method
4.  the initialization method specified as _init-method_ in XML

Let's create a Spring bean that combines all mechanisms:

```java
@Component
@Scope(value = "prototype")
public class AllStrategiesExampleBean implements InitializingBean {

    private static final Logger LOG 
      = Logger.getLogger(AllStrategiesExampleBean.class);

    public AllStrategiesExampleBean() {
        LOG.info("Constructor");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        LOG.info("InitializingBean");
    }

    @PostConstruct
    public void postConstruct() {
        LOG.info("PostConstruct");
    }

    public void init() {
        LOG.info("init-method");
    }
}
```

If you try to instantiate this bean, you will be able to see logs that match the order specified above:

```plaintext
[main] INFO o.b.startup.AllStrategiesExampleBean - Constructor
[main] INFO o.b.startup.AllStrategiesExampleBean - PostConstruct
[main] INFO o.b.startup.AllStrategiesExampleBean - InitializingBean
[main] INFO o.b.startup.AllStrategiesExampleBean - init-method
```

## **4\. Conclusion**[](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#conclusion)

In this article we illustrated multiple ways of executing logic on Spring's application startup.

Code samples can be found on [GitHub](https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data).
