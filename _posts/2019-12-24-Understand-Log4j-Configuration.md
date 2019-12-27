---
title: "Understand Log4j Configuration"
---

Log4j is a commonly used tool in Java. There are many advantages for using a professional logging tool compared to ```System.out.println```. This [post](https://stackoverflow.com/questions/8601831/do-not-use-system-out-println-in-server-side-code) talks some disadvatages of the system out method and advatages about using loggings. Here are some pros of logging and cons of system out I've read and heard:

| Cons of ```System.out.println```            | Pros of Logging                                                     |
|---------------------------------------------|---------------------------------------------------------------------|
| It is an IO-operation, performance issue    | Implemented with queue, it writes when the system has the bandwidth |
| Taking up the thread, until message printed | Process won't block on printing with the queue implementation       |
| No level control                            | Logging level can be adjusted                                       |
| Single or limited destination               | Multiple destinations can be configured                             |
| Content is in arbitrary form                | Layout of logging can be configured                                 |

Logging tool is from everyway better than using ```System.out.println``` except for you need to import dependency and configure it. I am working a bit on Log4j2 recently, and I did some research on  Log4j2 configuration. Within this post, I am not going very deep into every details of Log4j2, but just giving an idea of how it is configured to understand how the logging works.

Here is the link of the Log4j2 doc I have been reading, it is very useful and I think it helps a lot on understanding this tool.

<div class="embed"><iframe src="https://logging.apache.org/log4j/2.x/manual/architecture.html" frameborder="0" allowfullscreen></iframe></div>

Before we getting into any jargons of Log4j, let's take a glance at the configuration of Log4j. According to the Log4j2 doc, reading in a configuration file is a preferred way to configure Log4j (you can also change/update configuration with code). Here I am showing an XML configuration file, but you can also use json, yaml and property files. Different file types should not differ on what/how things are configured.

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%r [%t] %-5p %c - %m%n"/>
    </Console>
    <Console name="CloudwatchAppender" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{DATE} [%5p] (%t) %C:%L: %m%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="WARN">
      <AppenderRef ref="Console"/>
    </Root>
    <Logger name="com.kaiyi.activity" level="INFO" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>
    <Logger name="com.kaiyi.event" level="TRACE" additivity="true">
      <AppenderRef ref="CloudwatchAppender"/>
    </Logger>
    <Logger name="com.kaiyi.listener" level="TRACE" additivity="true">
    </Logger>
  </Loggers>
</Configuration>
```

There are 2 different parts in this configuration file, Appenders and Loggers. Within Appenders, there seems to be a PatternLayout defines the pattern of log message. For the Loggers, you seems to be able to define some features for each logger you have.

Now, with the configuration file present over there, even though we don't understand what exactly it means, you should be able to start using Log4j in your project. I think even those not familiar with Log4j configuration should be quite familiar with the usage of Log4j.

```java
package com.kaiyi.activities;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class CollectMushroomActivity {
  private static final Logger LOG = LogManager.getLogger(CollectMushroomActivity.class);

  public CollectMushroomActivity() {
    LOG.info("Constructing new " + CollectMushroomActivity.class + " instance.");
  }
}
```

With the dependencies all imported, you would see the following message from the console when instantiate this class:

```
616 [main] INFO  com.kaiyi.activity.CollectMushroomActivity - Constructing new class com.kaiyi.activity.CollectMushroomActivity instance.
```

So far, you might have a lot of questions, like what are "Appenders" and "additivity"? How does a logger use the settings we have in the configuration file? Let's go into more details about this logging tool.

### Initial Configuration

Let's get back to the xml file we present at the beggining. The configuration has 2 different parts: Appenders and Loggers. They both define a dimension of the logging system.

#### Appenders

```
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%r [%t] %-5p %c - %m%n"/>
    </Console>
    <Console name="CloudwatchAppender" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{DATE} [%5p] (%t) %C:%L: %m%n"/>
    </Console>
  </Appenders>
```

Appenders are the destinations of your log message, they can be "console, files, remote socket servers, Apache Flume, JMS, remote UNIX Syslog daemons, and various database APIs" according to the [apache document](https://logging.apache.org/log4j/2.x/manual/architecture.html#Appender). The above xml file declares a console appender and a CloudWatch appender. (This is not a correct way to define a CloudWatch appender, here is an [example](https://github.com/speedwing/log4j-cloudwatch-appender) for using CloudWatch appender if you are interested.) This is one of the advantages we are talking about with logging, where it allows you to print the log into many different places.

Within each Appender, you can define the layout of your message. The Console appender defined here is using a PatternLayout

```
"%r [%t] %-5p %c - %m%n"
```

%r   - milliseconds after it has started the program
%t   - the thread making log request
%-5p - Log level
%c   - name of Logger
%m   - message

The actual log you will see with the layout pattern is

```
616 [main] INFO  com.kaiyi.activity.CollectMushroomActivity - Constructing new class com.kaiyi.activity.CollectMushroomActivity instance.
```

PatternLayout is one of the layouts you can use, there are a lot more details within this [doc](https://logging.apache.org/log4j/2.x/manual/architecture.html#Appender) of how you can use them.

If you don't define any pattern for an appener, it will only print the raw log message without any additional information.

Defining the Appenders doesn't mean that all the message will be sent to these appenders by default. You will need to assign your logger with each appener in the next part of your configuration.

#### Loggers

When you configure a logger in your configuration file, you are configuring loggers within a scope instead of a single logger.

You can see that within this part of configuration, I only defined a logger with name "com.kaiyi.activity", but later when I invoked a logger in class "com.kaiyi.activity.CollectMushroomActivity", it still used the configuration of "com.kaiyi.activity".

In Log4j, the loggers follow the hierarchical naming rule. When a name is given to a logger with a path, it follows some convention like java class path. In this given example, "com.kaiyi.activity" is the parent "com.kaiyi.activity.CollectMushroomActivity". Without specific configuration on child logger, the child will inherit all the settings from its parent.

##### Root logger

Root logger is the ancestor of all the other loggers. If a logger and its ancestor logger doesn't have specific settings, it will use the settings of the root logger.

```
    <Root level="WARN">
      <AppenderRef ref="Console"/>
    </Root>
```

If you don't specify any settings for Root logger, Log4j will assign a default log level but not appender to the root logger. I don't see the default level from any documentation yet but I tested it and the default level is ERROR with my version of Log4j(2.12).

##### Level

Level acts as a filter in log4j. If you decided you just want to expose only the most critical logs, you can set the level of your logger so any log message with a lower level won't be printed in the logs. If you are only using the built-in levels, the comparison is TRACE < DEBUG < INFO < WARN	< ERROR	< FATAL	< OFF. If you set level to one of them, anything has lower serverity will be omitted.

```
    <Logger name="com.kaiyi.activity" level="INFO" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>
```

If you don't specify a level for a logger, it will inherit the level from its direct parent. If none of the ancestors have level defined, it will inherit the root logger's level.

If you need more levels than the built-in levels provide, you can refer to this [doc](https://logging.apache.org/log4j/2.x/manual/customloglevels.html). Or if you want ability to filter more specifically, you can refer to [Log4j Filter settings](https://logging.apache.org/log4j/2.x/manual/configuration.html#Filters).


##### Additivity

Additivity defines the inheritence of the appenders. If a logger has additivity set to true, it will inherit the appenders from its parent. One thing to notice is, if you have the same appender added to both a parent and child logger with additivity being true, the system will take both the appenders in child and parent and print the log twice into same appender. This table shows how additiviy works in different situations

| Logger                  | Assigned Appenders | additivity     | Target Appenders |
|-------------------------|--------------------|----------------|------------------|
| Root                    | A1                 | Not applicable | A1               |
| com                     |                    | true           | A1               |
| com.kaiyi               | A2                 | true           | A1, A2           |
| com.kaiyi.activity      | A3                 | false          | A3               |
| com.kaiyi.activity.Test | A3, A4             | true           | A3, A3, A4       |

### Invoking

Since we've figured out what the configuration is talking about, we can now easily understand the code that logs the message.

```java
package com.kaiyi.activity;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class CollectMushroomActivity {
  private static final Logger LOG = LogManager.getLogger(CollectMushroomActivity.class);

  public CollectMushroomActivity() {
    LOG.info("Constructing new " + CollectMushroomActivity.class + " instance.");
  }
}
```

In this class, we defined a class variable Logger, and invoke LogManager.getLogger with the full qualified class name. Clearly, it will initialize a logger with name ```com.kaiyi.activity.CollectMushroomActivity```. We know the parent of this logger must be ```com.kaiyi.activity```. Although we didn't config anything for logger ```com.kaiyi.activity.CollectMushroomActivity```, it will inherit all the settings of ```com.kaiyi.activity```.

```
    <Logger name="com.kaiyi.activity" level="INFO" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>
```

It will allow log level as much as INFO and print the logs out to "Console". Now when you call ```Log.info("...");```, your logs will appear in your console.
