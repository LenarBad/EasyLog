![GitHub](https://img.shields.io/github/license/lenarbad/easylog)
[![Maven Central](https://img.shields.io/maven-central/v/io.lenar/easy-log.svg)](https://maven-badges.herokuapp.com/maven-central/io.lenar/easy-log)

![Build Status](https://travis-ci.com/LenarBad/easylog.svg?branch=master)
[![Hits of Code](https://hitsofcode.com/github/lenarbad/easylog)](https://hitsofcode.com/view/github/lenarbad/easylog)
[![Lines of Code](https://tokei.rs/b1/github/lenarbad/easylog)](https://github.com/lenarbad/easylog)

[https://lenar.io/easylog/](https://lenar.io/easylog/)

# EasyLog 

EasyLog is an open source library for logging/debugging in Java projects.

## Table of Contents

* [Overview](#overview)
* [Quick Start](#quick-start)
  * [Spring Projects](#spring-projects)
  * [Non-Spring Projects](non-spring-projects)
* [@LogIt annotation](#logit-annotation)
  * [Logging Level](#logging-level)
  * [Labels](#labels)
  * [Exclude parameters from logging](#exclude-parameters-from-logging)
  * [Mask fields](#mask-fields)
  * [Logging Styles](#logging-styles)
* [Exceptions](#exceptions)
  * [WebApplicationException](#webapplicationexception)
  * [Retry on Exception](#retry-on-exception)
* [Interfaces](#interfaces)
  * [RESTEasy Client Interfaces. JAX-RS](#resteasy-client-interfaces)
* [Examples](#examples)
* [Warning](#warning)
* [Issues and suggestions](#issues-and-suggestions)
* [Contributions](#contributions)


## Overview

EasyLog allows you to start logging any method or all class's methods by adding just one annotation <code>@LogIt</code>. 

```java
    @LogIt
    public Universe bigBang(int numberOfStars) {
        ...
    }
```

You'll get following

```text
13:36:06.021 [main] INFO  UneasyLogger - 
-> public Universe Universe.bigBang(int numberOfStars)
"numberOfStars": 1

13:36:06.205 [main] INFO  UneasyLogger - 
Execution/Response time:  162ms
<- Universe Universe.bigBang(int numberOfStars)
{
  "stars": [
    {
      "name": "Star-b90637a4-81bb-4c46-9c05-99ecf2dc0502",
      "type": "RED_GIANT",
      "planets": [
        {
          "name": "Planet-c5308178-4ebe-46c6-a02a-f78d489afc99",
          "haveSatellites": true
        }
      ]
    }
  ],
  "dateOfCreation": "Jun 29, 2018 1:36:06 PM"
}
```

You can use many features to customize your log output. 

For example you can mask one or several field or subfield values (<code>@LogIt(maskFields = {"dateOfCreation"})</code>) then every time when <code>dateOfCreation</code> appears in logs it's value will be replaced with <code>XXXMASKEDXXX</code>


## Quick Start

See how to setup EasyLog in example projects
- [EasyLog for non-Spring projects - example](https://github.com/LenarBad/EasyLog-no-Spring-Example)
- [EasyLog for Spring projects - example](https://github.com/LenarBad/EasyLog-Spring-Example)

### Spring Projects

#### pom.xml

First, you need to setup your parent project

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.1.RELEASE</version>
    </parent>
```

Second, you need to add 2 dependencies

```xml
    <dependency>
        <groupId>io.lenar</groupId>
        <artifactId>easy-log</artifactId>
        <version>1.3.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
        <version>2.0.3.RELEASE</version>
    </dependency>
```

If your project already has a different parent project (that project also has to pe a Spring project as well) you might not need ```spring-boot-starter-aop``` - just check

#### Extend EasyLogger

In your project create the class that extends the <code>EasyLogger</code> aspect and add the <code>@Component</code> annotation.

```java
import io.lenar.easy.log.EasyLogger;
import org.springframework.stereotype.Component;

@Component
public class MyLogger extends EasyLogger {
}
```

[back to Table of Contents](#table-of-contents)

### Non-Spring Projects

#### pom.xml

First, you need to add <code>io.lenar:easy-log:{{ site.easylog_version }}</code> dependency

```xml
    <dependency>
        <groupId>io.lenar</groupId>
        <artifactId>easy-log</artifactId>
        <version>1.3.0</version>
    </dependency>
```

Second setup <code>build:plugins</code>

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>aspectj-maven-plugin</artifactId>
                <version>1.11</version>
                <configuration>
                    <complianceLevel>1.8</complianceLevel>
                    <source>1.8</source>
                    <target>1.8</target>
                    <verbose>true</verbose>
                    <Xlint>ignore</Xlint>
                    <encoding>1.8</encoding>
                </configuration>
                <executions>
                    <execution>
                        <id>aspectj-compile</id>
                        <goals>
                            <goal>compile</goal>
                            <goal>test-compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

#### Extend EasyLoggerNoSpring

```java
@Aspect
public class MyLogger extends EasyLoggerNoSpring {

    @Around("execution(* *(..)) && @within(annotation)")
    public Object classLog(ProceedingJoinPoint jp, LogIt annotation) throws Throwable {
        return logItClassLevel(jp, annotation);
    }

    @Around("execution(* *(..)) && @annotation(annotation)")
    public Object methodLog(ProceedingJoinPoint jp, LogIt annotation) throws Throwable {
        return logItMethodLevel(jp, annotation);
    }

    @AfterThrowing(pointcut = "execution(* *(..)) && @within(annotation)", throwing = "e")
    public void classExceptionLog(JoinPoint jp, LogIt annotation, Throwable e) {
        logExceptionClassLevel(jp, annotation, e);
    }

    @AfterThrowing(pointcut = "execution(* *(..)) && @annotation(annotation)", throwing = "e")
    public void methodExceptionLog(JoinPoint jp, LogIt annotation, Throwable e) {
        logExceptionMethodLevel(jp, annotation, e);
    }

}
```

Just copy and past.

If you use IntelliJ make sure that you use AJC compiler if this wasn't set automatically.
Also you may need to clear ```target``` folder (```mvn clean``` or just right click and delete on it).

Maven commands work as expected.

[back to Table of Contents](#table-of-contents)

## @LogIt annotation 

Annotate the methods that you want to log with <code>@LogIt</code> annotation 

```java
@LogIt
public User createUser(CreateUserForm form) {
    ...
}
```

If you need to log all methods of a class you can annotate the class with <code>@LogIt</code> annotation

 ```java
@LogIt
public class ClassWithMethods {
    ...
}
```

_Note: If you use ```@LogIt``` for a method and for a class, class's one will be ignored._

[back to Table of Contents](#table-of-contents)

### Logging Level

You can set the logging level by passing the parameter <code>level</code> with ```@LogIt``` annotation.

Available options:  ```DEBUG```, ```INFO```, ```WARN```, ```ERROR```

By default ```level=Level.INFO```

[back to Table of Contents](#table-of-contents)

### Labels

Labels help you to simplify a search for specific entries in the logs.
Just pass another annotation parameter ```String label```. 

### Exclude parameters from logging

You can skip some parameters and not log them with ```String[] ignoreParameters```. 

By default there is no ignored parameter.

### Mask fields

```String[] maskFields() default {}``` - allows to replace actual values for field names for the results returned by the method with ```"XXXMASKEDXXX"```.

_Note: We don't modify returned results, just customize how the results look in the logs._

By default there is no masked field.

Might be used for: 
 - masking any sensitive information that shouldn't be logged
 - decreasing the amount of logged info. For example we can replace huge lists/arrays (in returned results) that are not important in terms of logging with ```"XXXMASKEDXXX"```
 
[back to Table of Contents](#table-of-contents)

 
### Logging Styles

Available values: ```PRETTY_PRINT_WITH_NULLS```, ```PRETTY_PRINT_NO_NULLS```, ```COMPACT_WITH_NULLS```, ```MINIMAL```, ```AS_IS```.

By default it's ```PRETTY_PRINT_WITH_NULLS```

Here PRETTY_PRINT means _"pretty printed JSON"_.

```COMPACT_WITH_NULLS``` and ```MINIMAL``` also mean _JSON_ but not _"pretty printed"_

See the difference

```COMPACT_WITH_NULLS``` and ```MINIMAL```
```
{"user":{"zipcode":"12345","firstName":"John","lastName":"Smith","password":"passasdfasdf","email":"XXXMASKEDXXX"}}
```

 ```PRETTY_PRINT_WITH_NULLS``` and ```PRETTY_PRINT_NO_NULLS```
```
{  
   "user":{  
      "zipcode":"12345",
      "firstName":"John",
      "lastName":"Smith",
      "password":"passasdfasdf",
      "email":"XXXMASKEDXXX"
   }
}
```

Use ```PRETTY_PRINT_WITH_NULLS``` and ```COMPACT_WITH_NULLS``` if you want to log (serialize ```null```s)

Use ```PRETTY_PRINT_NO_NULLS``` and ```MINIMAL``` if you want to exclude nulls from logging.

```AS_IS``` is used if you want to serialize the parameters and returned result with the ```toString``` method. In this case ```maskFields``` will be ignored

[back to Table of Contents](#table-of-contents)

## Exceptions

If the method annotated with ```@LogIt``` (or belongs to the class annotated with ```LogIt```) throws an exception, it will be logged like this

```
14:17:54.359 [main] ERROR  io.lenar.easy.log.ExceptionLogger - java.lang.ArithmeticException: / by zero 
 <- Universe.getStarsBeforeBigBang(): 
java.lang.ArithmeticException: / by zero
	at io.lenar.examples.model.Universe.getStarsBeforeBigBang_aroundBody4(Universe.java:50) [classes/:na]
	at io.lenar.examples.model.Universe$AjcClosure5.run(Universe.java:1) ~[classes/:na]
	at org.aspectj.runtime.reflect.JoinPointImpl.proceed(JoinPointImpl.java:221) ~[aspectjrt-1.8.7.jar:na]
	at io.lenar.easy.log.UneasyLogger.logMethod(UneasyLogger.java:35) ~[easy-log-1.1.5-SNAPSHOT.jar:na]
	at io.lenar.easy.log.EasyLoggerNoSpring.logItMethodLevel(EasyLoggerNoSpring.java:22) ~[easy-log-1.1.5-SNAPSHOT.jar:na]
	at io.lenar.examples.log.MyLogger.methodLog(MyLogger.java:22) ~[classes/:na]
	at io.lenar.examples.model.Universe.getStarsBeforeBigBang(Universe.java:49) [classes/:na]
	at io.lenar.examples.LoggerTest.exceptionTest(LoggerTest.java:32) [test-classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_152]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_152]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_152]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_152]
...
```

### WebApplicationException

EasyLog logs ```WebApplicationException```s different way.

So if a service call throws the ```WebApplicationException``` exception then we will be also able to log error messages.

For example

```text
08:34:40.439 [main] INFO  UneasyLogger - 
AUTH SERVICE CLIENT
-> public LoginResponse AuthServiceClient.login(LoginRequest request)
request: {
  "password": "passasdfasdf",
  "loginID": "bademail@webmail.rei.com"
}


08:34:40.682 [main] ERROR io.lenar.easy.log.ExceptionLogger - javax.ws.rs.ForbiddenException: HTTP 403 Forbidden
AUTH SERVICE CLIENT <- AuthServiceClient.login(..): 
{"errorMessage":"Invalid LoginID","errorDetail":"invalid loginID or password","errorCode":403}
``` 

### Retry on Exception

If you need to retry a method on some specific exception or exceptions then you can use these parameters to setup the retry functionality.

```Class<? extends Throwable>[] retryExceptions() default {}``` -  a list of exceptions to retry on.

```int retryAttempts() default 1``` - retry ```retryAttempts``` times.

```long retryDelay() default 0``` - time delay between attempts in _ms_.

These parameters can be set for each method individually.

### Retry Example

```java
@LogIt(retryExceptions = {ForbiddenException.class, BadRequestException.class}, 
	retryDelay = 1000, 
	retryAttempts = 3)
```

In the logs you will see

```text
16:26:43.969 [main] ERROR io.lenar.easy.log.UneasyLogger - javax.ws.rs.BadRequestException: HTTP 400 Bad Request 
 <- UserService.findUser(..)
Retry 1/3 in 1000 ms
16:26:45.017 [main] ERROR io.lenar.easy.log.UneasyLogger - javax.ws.rs.BadRequestException: HTTP 400 Bad Request 
 <- UserService.findUser(..)
Retry 2/3 in 1000 ms
16:26:46.063 [main] ERROR io.lenar.easy.log.UneasyLogger - javax.ws.rs.BadRequestException: HTTP 400 Bad Request 
 <- UserService.findUser(..)
Retry 3/3 in 1000 ms
16:26:47.112 [main] ERROR io.lenar.easy.log.ExceptionLogger - javax.ws.rs.BadRequestException: HTTP 400 Bad Request
 <- UserService.findUser(..): 
{"code":400,"message":"First name is required","path":null,"parameterName":"firstName"}
```

_Note: You can specify a parent exception to cover all child exceptions. 
For example ```WebApplicationException``` covers ```BadRequestException```, ```ForbiddenException```,```NotFoundException``` etc_

[back to Table of Contents](#table-of-contents)

## Interfaces

If you want to log interface's method in all interface implementations then you can enable that feature by adding LoggerExtension like that

```java
@Aspect
@Component
public class LoggerExtension extends EasyLoggerExtension {

    @Pointcut("execution(* com.yourcompany.yourapp...*.*(..))")
    public void anyMethodInPackage() {}

    @Around("anyMethodInPackage()")
    public Object method(ProceedingJoinPoint jp) throws Throwable {
        return logIfMethodHasAnnotatedInterface(jp);
    }

}
```

```com.yourcompany.yourapp``` is your application package. It might be the top level package or some specific package in your application.

Now you can annotate any method of any interface or any interface inside this package and its sub-packages.
It will apply for all implementations of the interface.

### RESTEasy Client Interfaces

You can even annotate RESTful client interfaces that are used for RESTEasy Client Proxies (interfaces with ```JAX-RS``` annotations)._

```java
@Path("/users")
@LogIt(label = "USER SERVICE CLIENT")
public interface UserService {

   @GET
   @Produces("application/json")
   @Path("/{id}")
   User getUser(@PathParam("id") int id);

   @POST
   @Consumes("application/json")
   @Produces("application/json")
   User createUser(User user);

   @PUT
   @Consumes("application/json")
   @Produces("application/json")
   @Path("/{id}")
   User updateUser(@PathParam("id") int id, User user);
}
```

```LogIt``` can be used either for the whole interface or for its methods

[back to Table of Contents](#table-of-contents)

## Examples

 ```java
@LogIt(label="DEBUGGING ISSUE 1234", level=DEBUG)
public class ClassWithMethods {
    ...
}
```

 ```java
@LogIt(label="USER SERVICE CALL")
public class ClassWithMethods {
    ...
}
```

 ```java
@LogIt(label="USER CARDS", maskFields={"cardNumber", "pin"})
public UserCardsInfo getUserCards(..) {
    ...
}
```

 ```java
@LogIt(label="USER SERVICE CALL", maskFields={"password"})
public User login(String userName, String password) {
    ...
}
```

[back to Table of Contents](#table-of-contents)

Working example projects
- [EasyLog for non-Spring projects - example](https://github.com/LenarBad/EasyLog-no-Spring-Example)
- [EasyLog for Spring projects - example](https://github.com/LenarBad/EasyLog-Spring-Example)

## Warning

EasyLog walks through all passed and returned objects fields but It should not noticeably affect the performance

If you’re passing/returning very big objects or lists/arrays in highly loaded applications and concerned about performance I’d recommend to use AS_IS style and use your own toString() methods for serialization. Another option is to exclude those fields from logging with ```maskFields```

Examples

```java
    @LogIt(style=AS_IS)
    public Universe bigBang(int numberOfStars) {
        ...
    }
```

 ```java
    @LogIt(maskFields={"reallyHugeList"})
    public User login(String userName, String password) {
        ...
    }
```

## Issues and suggestions

Report your issues or suggestions [here](https://github.com/LenarBad/EasyLog/issues)

## Contributions

This is an opensource project - feel free to send your pull requests

Hot issues to work on:

 - [#15 Add JavaDocs](https://github.com/LenarBad/EasyLog/issues/15)
 - [#16 Add Unit Tests](https://github.com/LenarBad/EasyLog/issues/16)


[back to Table of Contents](#table-of-contents)
