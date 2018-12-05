# Limiting Repetitive Log Messages With Logback

### Learn about Logback's method of limiting repetitive log messages and how to use it to keep your log files clean and readable.


**Read why times series is the [fastest growing database](https://dzone.com/go?i=283423&u=https%3A%2F%2Fwww.influxdata.com%2Ftime-series-database%2F) category.**

Keeping log files clean and readable is paramount to being able to see what’s happening in an environment.

One source of hard-to-read log files is having many unnecessary repetitive lines. If there is no limit, then someone could unintentionally or intentionally generate hundreds of log lines of the exact same message, thus polluting the log files.

In this article, I am going to look at how Logback can assist with limiting unwanted repetitive log messages.

The associated code can be found [here](https://github.com/mahanhz/clean-architecture-example).

## The Default DuplicateMessageFilter

[Logback](https://logback.qos.ch/manual/filters.html#DuplicateMessageFilter) provides an out of the box solution. However, it comes with some limitations.

It can be defined in [logback.xml](https://github.com/mahanhz/clean-architecture-example/blob/master/application/configuration/src/main/resources/logback-spring.xml), like so:

`<turboFilter class="ch.qos.logback.classic.turbo.DuplicateMessageFilter"/>`

### Limitation #1

The first limitation is that only the raw message is taken into consideration, which means that parameterized log messages like the one below ignores the _customerId _parameter.

`LOGGER.info("Getting customer {}", customerId);`

Let’s say that I had eight customers (with sequential ids from 1 to 8) and every time I viewed a customer a log message like the above should be printed.

What I would get are 6 log lines rather than 8\. This is because the default configuration has 5 allowed repetitions (the first log line is of course not considered a repetition hence 6 log lines are printed) and the repetition count is based on the “Getting customer {}” string irrespective of the customerId.

#### View the Behavior in a Unit Test

This behavior can be seen in the _[DuplicateMessageFilterTest ](https://github.com/mahanhz/clean-architecture-example/blob/master/application/configuration/src/test/java/com/example/clean/app/configuration/log/DuplicateMessageFilterTest.java)_unit test.

```
 @Test

 public void shouldOnlyConsiderRawMessage() {

 final DuplicateMessageFilter dmf = new DuplicateMessageFilter();

 dmf.setAllowedRepetitions(5);

 dmf.start();

 assertThat(logMessage(dmf, "Getting customer {}", new String[]{"1"})).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, "Getting customer {}", new String[]{"2"})).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, "Getting customer {}", new String[]{"3"})).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, "Getting customer {}", new String[]{"4"})).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, "Getting customer {}", new String[]{"5"})).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, "Getting customer {}", new String[]{"6"})).isEqualTo(FilterReply.NEUTRAL);

 // messages are denied since the repetitions have been exceeded

 // due to customerId being ignored

 // resulting in the messages being considered the same

 assertThat(logMessage(dmf, "Getting customer {}", new String[]{"7"})).isEqualTo(FilterReply.DENY);

 assertThat(logMessage(dmf, "Getting customer {}", new String[]{"8"})).isEqualTo(FilterReply.DENY);

 }

 private FilterReply logMessage(final TurboFilter dmf, final String message, final String[] params) {

 return dmf.decide(null, null, null, message, params, null);

 }
```

#### View the Behavior by Running the Example Application

To see it in practice, run the application with spring profile _duplicate-default._

For example, run the following on the command line:

`gradlew bootRun -Dspring.profiles.active=duplicate-default`

Then go to the URL [http://localhost:13001/api/customers/1](http://localhost:13001/api/customers/1).

The output should be the log message **Getting customer 1.**

Repeat accessing the URL all the way to customer 8, and only the first 6 log lines should be printed.

### Limitation #2

Another limitation is that there is no time limit on the cache (LRU eviction with a default size of 100) that holds the messages and their respective count, which means that once I hit the limit of 5 repetitions, then I may never see those log messages appearing again if they are frequently hit.

These limitations do not meet my needs, but thankfully I can write a custom filter.

## Creating a Custom Filter

My requirements are:

*   Maximum 4 repetitions of a log message over a 30 second period.
*   Log messages should be based on similarity rather than an exact match of the raw message.
*   Certain log messages like those marked as audit or security should not be limited.

The custom filter will be created by extending from Logback’s [TurboFilter](https://logback.qos.ch/manual/filters.html#TurboFilters).

To make it configurable there will be four adjustable properties:

*   Number of allowed repetitions
*   Cache size
*   Expire from cache in seconds
*   Exclude markers

It can be defined in [logback.xml](https://github.com/mahanhz/clean-architecture-example/blob/master/application/configuration/src/main/resources/logback-spring.xml) like so:

```
<turboFilter class="com.example.clean.app.configuration.log.ExpiringDuplicateMessageFilter">

 <allowedRepetitions>4</allowedRepetitions>

 <cacheSize>200</cacheSize>

 <expireAfterWriteSeconds>30</expireAfterWriteSeconds>

 <excludeMarkers>AUDIT,SECURITY</excludeMarkers>

</turboFilter>
```

The custom TurboFilter ([ExpiringDuplicateMessageFilter](https://github.com/mahanhz/clean-architecture-example/blob/master/application/configuration/src/main/java/com/example/clean/app/configuration/log/ExpiringDuplicateMessageFilter.java)) will override the following methods:

*   _start_ – for initializing the cache
*   _stop_ – for invalidating the cache
*   _decide_ – contains the logic

I will use [Caffeine](https://github.com/ben-manes/caffeine) to implement the in-memory cache containing the messages and their respective repetition count.

#### View the Behavior in a Unit Test

This behavior can be seen in the _[ExpiringDuplicateMessageFilterTest ](https://github.com/mahanhz/clean-architecture-example/blob/master/application/configuration/src/test/java/com/example/clean/app/configuration/log/ExpiringDuplicateMessageFilterTest.java)_unit test.

```
 private static final String MESSAGE = "Getting customer {}";

 @Test

 public void shouldConsiderMessageSimilarity() {

 final ExpiringDuplicateMessageFilter dmf = new ExpiringDuplicateMessageFilter();

 dmf.setAllowedRepetitions(4);

 dmf.start();

 // customerId parameter is taken into consideration

 // so all are different and allowed

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("2"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("3"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("4"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("5"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("6"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("7"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("8"))).isEqualTo(FilterReply.NEUTRAL);

 }

 @Test

 public void shouldAllowOnlyFourRepetitions() {

 final ExpiringDuplicateMessageFilter dmf = new ExpiringDuplicateMessageFilter();

 dmf.setAllowedRepetitions(4);

 dmf.start();

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("2"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("3"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 // messages are denied as the repetition count for customer 1 has been exceeded

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.DENY);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.DENY);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.DENY);

 }

 @Test

 public void shouldReappearAfterCacheHasExpired() throws Exception {

 final ExpiringDuplicateMessageFilter dmf = new ExpiringDuplicateMessageFilter();

 dmf.setAllowedRepetitions(4);

 dmf.setExpireAfterWriteSeconds(2);

 dmf.start();

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 // messages are denied as the repetition count for customer 1 has been exceeded

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.DENY);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.DENY);

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.DENY);

 Thread.sleep(3000);

 // messages are allowed as the cache for customer 1 has expired

 assertThat(logMessage(dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 }

 private String[] customerId(final String id) {

 return new String[]{id};

 }

 private FilterReply logMessage(final Marker marker, final TurboFilter dmf, final String message, final String[] params) {

 return dmf.decide(marker, logger(), null, message, params, null);

 }

 private FilterReply logMessage(final TurboFilter dmf, final String message, final String[] params) {

 return logMessage(null, dmf, message, params);

 }

 private Logger logger() {

 return (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.example");

 }

```

#### View the Behavior by Running the Example Application

To see it in practice, run the application with spring profile _duplicate-custom._

For example, run the following on the command line:

gradlew bootRun -Dspring.profiles.active=duplicate-custom

Now access the URL ([http://localhost:13001/api/customers/1](http://localhost:13001/api/customers/1)) for all customers from 1 to 8.

The output should print all 8 log lines, as the customerId parameter is now being taken into account, so the log messages are no longer considered the same.

If I access [http://localhost:13001/api/customers/1](http://localhost:13001/api/customers/1) ten times then only the first 5 log lines will be printed (the first log line and 4 repetitions).

If I then wait for 30 seconds (for the cache entry to expire), and access customer 1 again then the log lines reappear.

The above now meets the expected behavior defined in the configuration.

#### Exceptions to the Rule

I don't want a blanket rule throughout my application. For certain situations, I do not want to apply any limitation- for example, if the log messages relate to auditing or security.

To allow for these exceptions the custom filter configuration has an excludeMarkers property defined with the value AUDIT,SECURITY.

This indicates that any log messages marked with these should not be limited.

The behavior can be seen in the following [unit test](https://github.com/mahanhz/clean-architecture-example/blob/master/application/configuration/src/test/java/com/example/clean/app/configuration/log/ExpiringDuplicateMessageFilterTest.java).

```
 private static final String SECURITY_MARKER = "SECURITY";

 private static final String NORMAL_MARKER   = "NORMAL";

 @Test

 public void shouldExcludeMarkers() throws Exception {

 final ExpiringDuplicateMessageFilter dmf = new ExpiringDuplicateMessageFilter();

 dmf.setAllowedRepetitions(4);

 dmf.setExcludeMarkers(SECURITY_MARKER);

 dmf.start();

 // There is no limitation on security markers as they are excluded

 assertThat(logMessage(marker(SECURITY_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(SECURITY_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(SECURITY_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(SECURITY_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(SECURITY_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(SECURITY_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 // The limit of 4 repetitions still applies to non security markers

 assertThat(logMessage(marker(NORMAL_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(NORMAL_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(NORMAL_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(NORMAL_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(NORMAL_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.NEUTRAL);

 assertThat(logMessage(marker(NORMAL_MARKER), dmf, MESSAGE, customerId("1"))).isEqualTo(FilterReply.DENY);

 }
```

It can also be seen by running the application and accessing the URLs [http://localhost:13001/audit](http://localhost:13001/audit)and [http://localhost:13001/security](http://localhost:13001/security) which will print the log messages "This is an auditable resource" and "This is a secure resource," respectively.

I can play around with the configuration by removing AUDIT or SECURITY from the excludeMarkers property to see that repetitive messages are being limited again.

## Conclusion

I now have greater flexibility in trying to keep my log files from being polluted with unwanted repetitive data, all of which is wrapped in a single class with a little bit of configuration.
