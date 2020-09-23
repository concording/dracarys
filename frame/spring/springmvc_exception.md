## [Exception Handling in Spring MVC](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc)

[ENGINEERING](https://spring.io/blog/category/engineering) 

[](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#disqus_thread)

**NOTE:** _[Revised April 2018](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#recent-updates)_

Spring MVC provides several complimentary approaches to exception handling but, when teaching Spring MVC, I often find that my students are confused or not comfortable with them.

Today I’m going to show you the various options available. Our goal is to _not_ handle exceptions explicitly in Controller methods where possible. They are a cross-cutting concern better handled separately in dedicated code.

There are three options: per exception, per controller or globally.

_A demonstration application that shows the points discussed here can be found at
[](https://github.com/paulc4/mvc-exceptions)[http://github.com/paulc4/mvc-exceptions](https://github.com/paulc4/mvc-exceptions). See [Sample Application](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#sample-application) below for details._

**NOTE:** _The demo applications has been revamped and updated (April 2018) to use Spring Boot 2.0.1 and is (hopefully) easier to use and understand. I also fixed some broken links (thanks for the feedback, sorry it took a while)._
## Spring Boot

[Spring Boot](https://spring.io/spring-boot) allows a Spring project to be setup with minimal configuration and it is likely that you are using it if your application is less than a few years old.

Spring MVC offers no default (fall-back) error page out-of-the-box. The most common way to set a default error page has always been the `SimpleMappingExceptionResolver` (since Spring V1 in fact). We will discuss this later.

However Spring Boot _does_ provide for a fallback error-handling page.

At start-up, Spring Boot tries to find a mapping for `/error`. By convention, a URL ending in `/error` maps to a logical view of the same name: `error`. In the demo application this view maps in turn to the `error.html` Thymeleaf template. (If using JSP, it would map to `error.jsp` according to the setup of your `InternalResourceViewResolver`). The actual mapping will depend on what `ViewResolver` (if any) that you or Spring Boot has setup.

If no view-resolver mapping for `/error` can be found, Spring Boot defines its own fall-back error page - the so-called “Whitelabel Error Page” (a minimal page with just the HTTP status information and any error details, such as the message from an uncaught exception). In the sample applicaiton, if you rename the `error.html` template to, say, `error2.html` then restart, you will see it being used.

If you are making a RESTful request (the HTTP request has specified a desired response type other than HTML) Spring Boot returns a JSON representation of the same error information that it puts in the “Whitelabel” error page.
```
$> curl -H "Accept: application/json" http://localhost:8080/no-such-page

{"timestamp":"2018-04-11T05:56:03.845+0000","status":404,"error":"Not Found","message":"No message available","path":"/no-such-page"}
```
Spring Boot also sets up a default error-page for the container, equivalent to the
`<error-page>` directive in `web.xml` (although implemented very differently). Exceptions thrown outside the Spring MVC framework, such as from a servlet Filter, are still reported by the Spring Boot fallback error page. The sample application also shows an example of this.

A more in-depth discussion of Spring Boot error-handling can be found at the end of this article.

_The rest of this article applies regardless of whether you are using Spring with or without Spring Boot_.

_Impatient REST developers may choose to skip directly to the [section](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#errors-and-rest) on custom REST error responses. However they should then read the full article as most of it applies equally to all web applications, REST or otherwise._

## [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#using-http-status-codes)Using HTTP Status Codes

Normally any unhandled exception thrown when processing a web-request causes the server to return an HTTP 500 response. However, any exception that you write yourself can be annotated with the `@ResponseStatus` annotation (which supports all the HTTP status codes defined by the HTTP specification). When an _annotated_ exception is thrown from a controller method, and not handled elsewhere, it will automatically cause the appropriate HTTP response to be returned with the specified status-code.

For example, here is an exception for a missing order.
```
 @ResponseStatus(value=HttpStatus.NOT_FOUND, reason="No such Order")  // 404
 public class OrderNotFoundException extends RuntimeException {
     // ...
 }
```
And here is a controller method using it:
```
 @RequestMapping(value="/orders/{id}", method=GET)
 public String showOrder(@PathVariable("id") long id, Model model) {
     Order order = orderRepository.findOrderById(id);

     if (order == null) throw new OrderNotFoundException(id);

     model.addAttribute(order);
     return "orderDetail";
 }
```
A familiar HTTP 404 response will be returned if the URL handled by this method includes an unknown order id.

## [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#controller-based-exception-handling)Controller Based Exception Handling

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#using-exceptionhandler)Using @ExceptionHandler

You can add extra (`@ExceptionHandler`) methods to any controller to specifically handle exceptions thrown by request handling (`@RequestMapping`) methods in the same controller. Such methods can:

1.  Handle exceptions without the `@ResponseStatus` annotation (typically predefined exceptions that you didn’t write)
2.  Redirect the user to a dedicated error view
3.  Build a totally custom error response

The following controller demonstrates these three options:
```
@Controller
public class ExceptionHandlingController {

  // @RequestHandler methods
  ...
  
  // Exception handling methods
  
  // Convert a predefined exception to an HTTP Status code
  @ResponseStatus(value=HttpStatus.CONFLICT,
                  reason="Data integrity violation")  // 409
  @ExceptionHandler(DataIntegrityViolationException.class)
  public void conflict() {
    // Nothing to do
  }
  
  // Specify name of a specific view that will be used to display the error:
  @ExceptionHandler({SQLException.class,DataAccessException.class})
  public String databaseError() {
    // Nothing to do.  Returns the logical view name of an error page, passed
    // to the view-resolver(s) in usual way.
    // Note that the exception is NOT available to this view (it is not added
    // to the model) but see "Extending ExceptionHandlerExceptionResolver"
    // below.
    return "databaseError";
  }

  // Total control - setup a model and return the view name yourself. Or
  // consider subclassing ExceptionHandlerExceptionResolver (see below).
  @ExceptionHandler(Exception.class)
  public ModelAndView handleError(HttpServletRequest req, Exception ex) {
    logger.error("Request: " + req.getRequestURL() + " raised " + ex);

    ModelAndView mav = new ModelAndView();
    mav.addObject("exception", ex);
    mav.addObject("url", req.getRequestURL());
    mav.setViewName("error");
    return mav;
  }
}
```

In any of these methods you might choose to do additional processing - the most common example is to log the exception.

Handler methods have flexible signatures so you can pass in obvious servlet-related objects such as `HttpServletRequest`, `HttpServletResponse`, `HttpSession` and/or `Principle`.

**Important Note:** The `Model` may **not** be a parameter of any `@ExceptionHandler` method. Instead, setup a model inside the method using a `ModelAndView` as shown by `handleError()` above.

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#exceptions-and-views)Exceptions and Views

Be careful when adding exceptions to the model. Your users do not want to see web-pages containing Java exception details and stack-traces. You may have security policies that expressly _forbid_ putting _any_ exception information in the error page. Another reason to make sure you override the Spring Boot white-label error page.

Make sure exceptions are logged usefully so they can be analyzed after the event by your support and development teams.

_Please remember the following may be convenient but it is **not** best practice in production_.

It can be useful to hide exception details in the page _source_ as a comment, to assist _testing_. If using JSP, you could do something like this to output the exception and the corresponding stack-trace (using a hidden `<div>` is another option).

```
  <h1>Error Page</h1>
  <p>Application has encountered an error. Please contact support on ...</p>
    
  <!--
    Failed URL: ${url}
    Exception:  ${exception.message}
        <c:forEach items="${exception.stackTrace}" var="ste">    ${ste} 
    </c:forEach>
  -->
```

For the Thymeleaf equivalent see [support.html](https://github.com/paulc4/mvc-exceptions/blob/master/src/main/resources/templates/support.html) in the demo application. The result looks like this.

![Example of an error page with a hidden exception for support](https://assets.spring.io/wp/wp-content/uploads/2013/10/support-page-example.png "Error Page with Hidden Exception")

## [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#global-exception-handling)Global Exception Handling

### Using @ControllerAdvice Classes

A controller advice allows you to use exactly the same exception handling techniques but apply them across the whole application, not just to an individual controller. You can think of them as an annotation driven interceptor.

Any class annotated with `@ControllerAdvice` becomes a controller-advice and three types of method are supported:

*   Exception handling methods annotated with `@ExceptionHandler`.
*   Model enhancement methods (for adding additional data to the model) annotated with
    `@ModelAttribute`. Note that these attributes are _not_ available to the exception handling views.
*   Binder initialization methods (used for configuring form-handling) annotated with
    `@InitBinder`.

We are only going to look at exception handling - search the [online manual](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-controller) for more on `@ControllerAdvice` methods.

Any of the exception handlers you saw above can be defined on a controller-advice class - but now they apply to exceptions thrown from _any_ controller. Here is a simple example:

```
@ControllerAdvice
class GlobalControllerExceptionHandler {
    @ResponseStatus(HttpStatus.CONFLICT)  // 409
    @ExceptionHandler(DataIntegrityViolationException.class)
    public void handleConflict() {
        // Nothing to do
    }
}
```
If you want to have a default handler for _any_ exception, there is a slight wrinkle. You need to ensure annotated exceptions are handled by the framework. The code looks like this:
```
@ControllerAdvice
class GlobalDefaultExceptionHandler {
  public static final String DEFAULT_ERROR_VIEW = "error";

  @ExceptionHandler(value = Exception.class)
  public ModelAndView
  defaultErrorHandler(HttpServletRequest req, Exception e) throws Exception {
    // If the exception is annotated with @ResponseStatus rethrow it and let
    // the framework handle it - like the OrderNotFoundException example
    // at the start of this post.
    // AnnotationUtils is a Spring Framework utility class.
    if (AnnotationUtils.findAnnotation
                (e.getClass(), ResponseStatus.class) != null)
      throw e;

    // Otherwise setup and send the user to a default error-view.
    ModelAndView mav = new ModelAndView();
    mav.addObject("exception", e);
    mav.addObject("url", req.getRequestURL());
    mav.setViewName(DEFAULT_ERROR_VIEW);
    return mav;
  }
}
```
## Going Deeper

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#handlerexceptionresolver)HandlerExceptionResolver

Any Spring bean declared in the `DispatcherServlet`’s application context that implements `HandlerExceptionResolver` will be used to intercept and process any exception raised in the MVC system and not handled by a Controller. The interface looks like this:
```
public interface HandlerExceptionResolver {
    ModelAndView resolveException(HttpServletRequest request, 
            HttpServletResponse response, Object handler, Exception ex);
}
```
The `handler` refers to the controller that generated the exception (remember that `@Controller` instances are only one type of handler supported by Spring MVC. For example: `HttpInvokerExporter` and the WebFlow Executor are also types of handler).

Behind the scenes, MVC creates three such resolvers by default. It is these resolvers that implement the behaviours discussed above:

*   `ExceptionHandlerExceptionResolver` matches uncaught exceptions against suitable `@ExceptionHandler` methods on both the handler (controller) and on any controller-advices.
*   `ResponseStatusExceptionResolver` looks for uncaught exceptions annotated by `@ResponseStatus` (as described in Section 1)
*   `DefaultHandlerExceptionResolver` converts standard Spring exceptions and converts them to HTTP Status Codes (I have not mentioned this above as it is internal to Spring MVC).

These are chained and processed in the order listed - internally Spring creates a dedicated bean (the `HandlerExceptionResolverComposite`) to do this.

Notice that the method signature of `resolveException` does not include the `Model`. This is why `@ExceptionHandler` methods cannot be injected with the model.

You can, if you wish, implement your own `HandlerExceptionResolver` to setup your own custom exception handling system. Handlers typically implement Spring’s `Ordered` interface so you can define the order that the handlers run in.

### SimpleMappingExceptionResolver

Spring has long provided a simple but convenient implementation of `HandlerExceptionResolver` that you may well find being used in your appication already - the `SimpleMappingExceptionResolver`. It provides options to:

*   Map exception class names to view names - just specify the classname, no package needed.
*   Specify a default (fallback) error page for any exception not handled anywhere else
*   Log a message (this is not enabled by default).
*   Set the name of the `exception` attribute to add to the Model so it can be used inside a View
    (such as a JSP). By default this attribute is named `exception`. Set to `null` to disable. Remember that views returned from `@ExceptionHandler` methods _do not_ have access to the exception but views defined to `SimpleMappingExceptionResolver` _do_.

Here is a typical configuration using Java Configuration:
```
@Configuration
@EnableWebMvc  // Optionally setup Spring MVC defaults (if you aren't using
               // Spring Boot & haven't specified @EnableWebMvc elsewhere)
public class MvcConfiguration extends WebMvcConfigurerAdapter {
  @Bean(name="simpleMappingExceptionResolver")
  public SimpleMappingExceptionResolver
                  createSimpleMappingExceptionResolver() {
    SimpleMappingExceptionResolver r =
                new SimpleMappingExceptionResolver();

    Properties mappings = new Properties();
    mappings.setProperty("DatabaseException", "databaseError");
    mappings.setProperty("InvalidCreditCardException", "creditCardError");

    r.setExceptionMappings(mappings);  // None by default
    r.setDefaultErrorView("error");    // No default
    r.setExceptionAttribute("ex");     // Default is "exception"
    r.setWarnLogCategory("example.MvcLogger");     // No default
    return r;
  }
  ...
}
```
Or using XML Configuration:
```
  <bean id="simpleMappingExceptionResolver" class=
     "org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
      <map>
         <entry key="DatabaseException" value="databaseError"/>
         <entry key="InvalidCreditCardException" value="creditCardError"/>
      </map>
    </property>

    <!-- See note below on how this interacts with Spring Boot -->
    <property name="defaultErrorView" value="error"/>
    <property name="exceptionAttribute" value="ex"/>
        
    <!-- Name of logger to use to log exceptions. Unset by default, 
           so logging is disabled unless you set a value. -->
    <property name="warnLogCategory" value="example.MvcLogger"/>
  </bean>
```

The _defaultErrorView_ property is especially useful as it ensures any uncaught exception generates a suitable application defined error page. (The default for most application servers is to display a Java stack-trace - something your users should _never_ see). Spring Boot provides another way to do the same thing with its “white-label” error page.

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#extending-simplemappingexceptionresolver)Extending SimpleMappingExceptionResolver

It is quite common to extend `SimpleMappingExceptionResolver` for several reasons:

*   You can use the constructor to set properties directly - for example to enable exception logging and set the logger to use
*   Override the default log message by overriding `buildLogMessage`. The default implementation always returns this fixed text:
    _Handler execution resulted in exception_
*   To make additional information available to the error view by overriding `doResolveException`

For example:
```
public class MyMappingExceptionResolver extends SimpleMappingExceptionResolver {
  public MyMappingExceptionResolver() {
    // Enable logging by providing the name of the logger to use
    setWarnLogCategory(MyMappingExceptionResolver.class.getName());
  }

  @Override
  public String buildLogMessage(Exception e, HttpServletRequest req) {
    return "MVC exception: " + e.getLocalizedMessage();
  }
    
  @Override
  protected ModelAndView doResolveException(HttpServletRequest req,
        HttpServletResponse resp, Object handler, Exception ex) {
    // Call super method to get the ModelAndView
    ModelAndView mav = super.doResolveException(req, resp, handler, ex);
        
    // Make the full URL available to the view - note ModelAndView uses
    // addObject() but Model uses addAttribute(). They work the same. 
    mav.addObject("url", request.getRequestURL());
    return mav;
  }
}
```
This code is in the demo application as [ExampleSimpleMappingExceptionResolver](https://github.com/paulc4/mvc-exceptions/blob/master/src/main/java/demo/example/ExampleSimpleMappingExceptionResolver.java)

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#extending-exceptionhandlerexceptionresolver)Extending ExceptionHandlerExceptionResolver

It is also possible to extend `ExceptionHandlerExceptionResolver` and override its
`doResolveHandlerMethodException` method in the same way. It has almost the same signature (it just takes the new `HandlerMethod` instead of a `Handler`).

To make sure it gets used, also set the inherited order property (for example in the constructor of your new class) to a value less than `MAX_INT` so it runs _before_ the default ExceptionHandlerExceptionResolver instance (it is easier to create your own handler instance than try to modify/replace the one created by Spring). See [ExampleExceptionHandlerExceptionResolver](https://github.com/paulc4/mvc-exceptions/blob/master/src/main/java/demo/example/ExampleExceptionHandlerExceptionResolver.java) in the demo app for more.

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#errors-and-rest)Errors and REST

RESTful GET requests may also generate exceptions and we have already seen how we can return standard HTTP Error response codes. However, what if you want to return information about the error? This is very easy to do. Firstly define an error class:
```
public class ErrorInfo {
    public final String url;
    public final String ex;

    public ErrorInfo(String url, Exception ex) {
        this.url = url;
        this.ex = ex.getLocalizedMessage();
    }
}
```
Now we can return an instance from a handler as the `@ResponseBody` like this:
```
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(MyBadDataException.class)
@ResponseBody ErrorInfo
handleBadRequest(HttpServletRequest req, Exception ex) {
    return new ErrorInfo(req.getRequestURL(), ex);
} 
```
## What to Use When?

As usual, Spring likes to offer you choice, so what should you do? Here are some rules of thumb. However if you have a preference for XML configuration or Annotations, that’s fine too.

*   For exceptions you write, consider adding `@ResponseStatus` to them.
*   For all other exceptions implement an `@ExceptionHandler` method on a `@ControllerAdvice` class or use an instance of `SimpleMappingExceptionResolver`. You may well have `SimpleMappingExceptionResolver` configured for your application already, in which case it may be easier to add new exception classes to it than implement a `@ControllerAdvice`.
*   For Controller specific exception handling add `@ExceptionHandler` methods to your controller.
*   **Warning:** Be careful mixing too many of these options in the same application. If the same exception can be handed in more than one way, you may not get the behavior you wanted. `@ExceptionHandler` methods on the Controller are always selected before those on any `@ControllerAdvice` instance. It is _undefined_ what order controller-advices are processed.

## [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#sample-application)Sample Application

A demonstration application can be found at [github](https://github.com/paulc4/mvc-exceptions). It uses Spring Boot and Thymeleaf to build a simple web application.

The application has been revised twice (Oct 2014, April 2018) and is (hopefully) better and easier to understand. The fundamentals stay the same. It uses Spring Boot V2.0.1 and Spring V5.0.5 but the code is applicable to Spring 3.x and 4.x also.

The demo is running on Cloud Foundry at [](https://mvc-exceptions-v2.cfapps.io/)[http://mvc-exceptions-v2.cfapps.io/](https://mvc-exceptions-v2.cfapps.io/).

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#about-the-demo)About the Demo

The application leads the user through 5 demo pages, highlighting different exception handling techniques:

1.  A controller with `@ExceptionHandler` methods to handle its own exceptions
2.  A contoller that throws exceptions for a global ControllerAdvice to handle
3.  Using a `SimpleMappingExceptionResolver` to handle exceptions
4.  Same as demo 3 but with the `SimpleMappingExceptionResolver` disabled for comparison
5.  Shows how Spring Boot generates its error page

A description of the most important files in the application and how they relate to each demo can be found in the project’s [README.md](https://github.com/paulc4/mvc-exceptions/blob/master/README.md).

The home web-page is [index.html](https://github.com/paulc4/mvc-exceptions/blob/master/src/main/resources/templates/index.html) which:

*   Links to each demo page
*   Links (bottom of the page) to Spring Boot endpoints for those interested in Spring Boot.

Each demo page contains several links, all of which deliberately raise exceptions. You will need to use the back-button on your browser each time to return to the demo page.

Thanks to Spring Boot, you can run this demo as a Java application (it runs an embedded Tomcat container). To run the application, you can use one of the following (the second is thanks to the Spring Boot maven plugin):

*   `mvn exec:java`
*   `mvn spring-boot:run`

Your choice. The home page URL will be [](http://localhost:8080/)[http://localhost:8080](http://localhost:8080/).

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#error-page-contents)Error Page Contents

Also in the demo application I show how to create a “support-ready” error page with a stack-trace hidden in the HTML source (as a comment). Ideally support should get this information from the logs, but life isn’t always ideal. Regardless, what this page _does_ show is how the underlying error-handling method `handleError` creates its own `ModelAndView` to provide extra information in the error page. See:

*   `ExceptionHandlingController.handleError()` on [github](https://github.com/paulc4/mvc-exceptions/blob/master/src/main/java/demo1/web/ExceptionHandlingController.java)
*   `GlobalControllerExceptionHandler.handleError()` on [github](https://github.com/paulc4/mvc-exceptions/blob/master/src/main/java/demo2/web/GlobalExceptionHandlingControllerAdvice.java)

## [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#spring-boot-and-error-handling)Spring Boot and Error Handling

[Spring Boot](https://spring.io/spring-boot) allows a Spring project to be setup with minimal configuration. Spring Boot creates sensible defaults automatically when it detects certain key classes and packages on the classpath. For example if it sees that you are using a Servlet environment, it sets up Spring MVC with the most commonly used view-resolvers, hander mappings and so forth. If it sees JSP and/or Thymeleaf, it sets up these view-technologies.

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#fallback-error-page)Fallback Error Page

How does Spring Boot support the default error-handling described at the beginning of this article?

1.  In the event of any unhanded error, Spring Boot forwards internally to `/error`.
2.  Boot sets up a `BasicErrorController` to handle any request to `/error`. The controller adds error information to the internal Model and returns `error` as the logical view name.
3.  If any view-resolver(s) are configured, they will try to use a corresponding error-view.
4.  Otherwise, a default error page is provided using a dedicated `View` object (making it independent of any view-resolution system you may be using).
5.  Spring Boot sets up a `BeanNameViewResolver` so that `/error` can be mapped to a `View` of the same name.
6.  If you look in Boot’s [`ErrorMvcAutoConfiguration`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration.java) class you will see that the `defaultErrorView` is returned as a bean called `error`. This is the View bean found by the `BeanNameViewResolver`.

The “Whitelabel” error page is deliberately minimal and ugly. You can override it:

1.  By defining an error template - in our demo we are using Thymeleaf so the error template is in `src/main/resources/templates/error.html` (this location is set by the Spring Boot property `spring.thymeleaf.prefix` - similar properties exist for other supported server-side view technologies such as JSP or Mustache).
2.  If you _aren’t_ using server-side rendering
    2.1 Define your own error View as a bean called `error`.
    2.1 Or disable Spring boot’s “Whitelabel” error page by setting the property
    `server.error.whitelabel.enabled` to `false`. Your container’s default error page is used instead.

By convention, Spring Boot properties are normally set in `application.properties` or `application.yml`.

### Integration with _SimpleMappingExceptionResolver_

What if you are already using `SimpleMappingExceptionResolver` to setup a default
error view? Simple, use `setDefaultErrorView()` to define the same view that Spring Boot uses: `error`.

Note that in the demo, the `defaultErrorView` property of the `SimpleMappingExceptionResolver` is deliberately set not to `error` but to `defaultErrorPage` so you can see when the handler is generating the error page and when Spring Boot is responsible. Normally _both_ would be set to `error`.

### [](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc#container-wide-exception-handling)Container-Wide Exception Handling

Exceptions thrown outside the Spring Framework, such as from a servlet Filter, are also reported by Spring Boot’s fallback error page.

To do this Spring Boot has to register a default error page for the container. In Servlet 2, there is an `<error-page>` directive that you can add to your `web.xml` to do this. Sadly Servlet 3 does not offer a Java API equivalent. Instead Spring Boot does the following:

*   For a Jar application, with an embedded container, it registers a default error page using Container specific API.
*   For a Spring Boot application deployed as a traditional WAR file, a Servlet Filter is used to
    catch exceptions raised further down the line and handle it.

[comments powered by Disqus](https://disqus.com/)
