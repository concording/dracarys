# [Why use @PostConstruct?](https://stackoverflow.com/questions/3406555/why-use-postconstruct)

*   because when the constructor is called, the bean is not yet initialized - i.e. no dependencies are injected. In the `@PostConstruct` method the bean is fully initialized and you can use the dependencies.

*   because this is the contract that guarantees that this method will be invoked only once in the bean lifecycle. It may happen (though unlikely) that a bean is instantiated multiple times by the container in its internal working, but it guarantees that `@PostConstruct` will be invoked only once.


# [Spring: Why is afterPropertiesSet() of InitializingBean needed when there are static and non-static initializers in Java?](https://stackoverflow.com/questions/30726189/spring-why-is-afterpropertiesset-of-initializingbean-needed-when-there-are-st)

Given the following class

```
public class MyClass implements InitializingBean {

    static { ... } // static initializer
    { ... }  // non-static initializer

    public void afterPropertiesSet() throws Exception { ... }
}
```

The **static** initializer block is only executed when the class is loaded by the class loader. There is no instance of that class at that moment and you will only be able to access class level (`static`) variables at that point and not instance variables.

The **non-static** initializer block is when the object is constructed but before any properties are injected. The non-static initializer block is actually copied to the constructor.

> The Java compiler copies initializer blocks into every constructor. Therefore, this approach can be used to share a block of code between multiple constructors.

See also [Static Initialization Blocks](https://stackoverflow.com/questions/2420389/static-initialization-blocks) and [http://docs.oracle.com/javase/tutorial/java/javaOO/initial.html](http://docs.oracle.com/javase/tutorial/java/javaOO/initial.html)

The `afterPropertiesSet` or `@PostConstruct` annotated method is called after an instance of class is created and all the properties have been set. For instance if you would like to preload some data that can be done in this method as all the dependencies have been set.

If you only have mandatory dependencies you might be better of using constructor injection and instead of using `InitializingBean` or `@PostConstruct` put the initializing logic in the constructor. This will only work if all the dependencies are injected through the constructor, if you have optional dependencies set by set methods then you have no choice but to use `@PostConstruct` or `InitializingBean`.
