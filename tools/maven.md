
> Life cycle is a sequence of named **phases**.
> Phases executes sequentially. Executing a phase means executes all previous phases.
> 
> Plugin is a collection of **goals** also called MOJO (**M**aven **O**ld **J**ava **O**bject).
> Analogy : Plugin is a class and goals are methods within the class.

**Maven** is based around the central concept of a build lifecycle.
There are three built-in build lifecycles:

1.  default
2.  clean
3.  site

[**Each Build Lifecycle is Made Up of Phases**](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

For example the `default` lifecycle comprises of the following **Build Phases**:

```
◾validate - validate the project is correct and all necessary information is available
◾compile - compile the source code of the project
◾test - test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
◾package - take the compiled code and package it in its distributable format, such as a JAR.
◾integration-test - process and deploy the package if necessary into an environment where integration tests can be run
◾verify - run any checks to verify the package is valid and meets quality criteria
◾install - install the package into the local repository, for use as a dependency in other projects locally
◾deploy - done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.
```

So to go through the above phases, we just have to call one command:

```
mvn <phase> { Ex: mvn install }
```
For the above command, starting from the first phase, all the phases are executed sequentially till the ‘install’ phase. `mvn` can either execute a goal or a phase (or even multiple goals or multiple phases) as follows:

```
mvn clean install plugin:goal  
```

However, if you want to customize the prefix used to reference your plugin, you can specify the prefix directly through a configuration parameter on the `maven-plugin-plugin` in your [plugin's POM.](https://maven.apache.org/guides/introduction/introduction-to-plugin-prefix-mapping.html)

**A Build Phase is Made Up of [Plugin](https://maven.apache.org/plugins/index.html) Goals**

Most of Maven's functionality is in plugins. A plugin provides a set of **goals** that can be executed using the following syntax:

```
 mvn [plugin-name]:[goal-name]
```

For example, a Java project can be compiled with the compiler-plugin's compile-goal by running `mvn compiler:compile`.

Build lifecycle is a list of named phases that can be used to give order to goal execution.

**Goals provided by plugins can be associated with different phases of the lifecycle.** For example, by default, the _goal_ `compiler:compile` is associated with the `compile` _phase_, while the _goal_ `surefire:test` is associated with the `test` _phase_. Consider the following command:

```
mvn test
```

When the preceding command is executed, Maven runs all goals associated with each of the phases up to and including the `test` phase. In such a case, Maven runs the `resources:resources` goal associated with the `process-resources` phase, then `compiler:compile`, and so on until it finally runs the `surefire:test` goal.

However, even though a build phase is responsible for a specific step in the build lifecycle, the manner in which it carries out those responsibilities may vary. And this is done by declaring the plugin goals bound to those build phases.

A plugin goal represents a specific task (finer than a build phase) which contributes to the building and managing of a project. It may be bound to zero or more build phases. A goal not bound to any build phase could be executed outside of the build lifecycle by direct invocation. The order of execution depends on the order in which the goal(s) and the build phase(s) are invoked. For example, consider the command below. The `clean` and `package` arguments are build phases, while the `dependency:copy-dependencies` is a goal (of a plugin).

```
mvn clean dependency:copy-dependencies package
```

If this were to be executed, the `clean` phase will be executed first (meaning it will run all preceding phases of the clean lifecycle, plus the `clean` phase itself), and then the `dependency:copy-dependencies` goal, before finally executing the `package` phase (and all its preceding build phases of the default lifecycle).

Moreover, if a goal is bound to one or more build phases, that goal will be called in all those phases.

Furthermore, a build phase can also have zero or more goals bound to it. If a build phase has no goals bound to it, that build phase will not execute. But if it has one or more goals bound to it, it will execute all those goals.

**[Built-in Lifecycle Bindings](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings)**
Some phases have goals bound to them by default. And for the default lifecycle, these bindings depend on the packaging value.

Maven Architecture:

![enter image description here](https://i.stack.imgur.com/wZpms.png)


[Reference 1](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Build_Lifecycle_Basics)
[Reference 2](https://en.wikipedia.org/wiki/Apache_Maven#Build_lifecycles)
