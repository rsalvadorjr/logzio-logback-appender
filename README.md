[![Build Status](https://travis-ci.org/logzio/logzio-logback-appender.svg?branch=master)](https://travis-ci.org/logzio/logzio-logback-appender)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.logz.logback/logzio-logback-appender/badge.svg)](http://mvnrepository.com/artifact/io.logz.logback/logzio-logback-appender)

# Logzio logback appender
This appender sends logs to your [Logz.io](http://logz.io) account, using non-blocking threading, bulks, and HTTPS encryption. Please note that this appendr requires logback version 1.1.7 and up, and java 8 and up.

### Technical Information
This appender uses [BigQueue](https://github.com/bulldog2011/bigqueue) implementation of persistent queue, so all logs are backed up to a local file system before being sent. Once you send a log, it will be enqueued in the buffer and 100% non-blocking. There is a background task that will handle the log shipment for you. This jar is an "Uber-Jar" that shades both BigQueue, Gson and Guava to avoid "dependency hell".

### Installation from maven
```xml
<dependency>
    <groupId>io.logz.logback</groupId>
    <artifactId>logzio-logback-appender</artifactId>
    <version>1.0.10</version>
</dependency>
```

### Logback Example Configuration
```xml
<!-- Use debug=true here if you want to see output from the appender itself -->
<configuration>
    <!-- Use shutdownHook so that we can close gracefully and finish the log drain -->
    <shutdownHook class="ch.qos.logback.core.hook.DelayingShutdownHook"/>
    <appender name="LogzioLogbackAppender" class="io.logz.logback.LogzioLogbackAppender">
        <token>yourlogziopersonaltokenfromsettings</token>
        <logzioType>myAwesomeType</logzioType>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
    </appender>
</configuration>
```

### Parameters
| Parameter          | Default                              | Explained  |
| ------------------ | ------------------------------------ | ----- |
| **token**              | *None*                                 | Your Logz.io token, which can be found under "settings" in your account |
| **logzioType**               | *java*                                 | The [log type](http://support.logz.io/support/solutions/articles/6000103063-what-is-type-) for that appender |
| **drainTimeoutSec**       | *5*                                    | How often the appender should drain the buffer (in seconds) |
| **fileSystemFullPercentThreshold** | *98*                                   | The percent of used file system space at which the appender will stop buffering. When we will reach that percentage, the file system in which the buffer rests will drop all new logs until the percentage of used space drops below that threshold. Set to -1 to never stop processing new logs |
| **bufferDir**          | *System.getProperty("java.io.tmpdir")* | Where the appender should store the buffer |
| **socketTimeout**       | *10 * 1000*                                    | The socket timeout during log shipment |
| **connectTimeout**       | *10 * 1000*                                    | The connection timeout during log shipment |
| **addHostname**       | *false*                                    | Optional. If true, then a field named 'hostname' will be added holding the host name of the machine. If from some reason there's no defined hostname, this field won't be added |
| **additionalFields**       | *None*                                    | Optional. Allows to add additional fields to the JSON message sent. The format is "fieldName1=fieldValue1;fieldName2=fieldValue2". You can optionally inject an environment variable value using the following format: "fieldName1=fieldValue1;fieldName2=$ENV_VAR_NAME". In that case, the environment variable should be the only value. In case the environment variable can't be resolved, the field will be omitted. |
| **debug**       | *false*                                    | Print some debug messages to stdout to help to diagnose issues |


### Code Example
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogzioLogbackExample {

    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(LogzioLogbackExample.class);
        
        logger.info("Testing logz.io!");
        logger.warn("Winter is coming");
    }
}
```

### MDC
Each key value you will add to MDC will be added to each log line as long as the thread alive. No further configuration needed.
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

public class LogzioLogbackExample {

    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(LogzioLogbackExample.class);

        MDC.put("Key", "Value");
        logger.info("This log will hold the MDC data as well");
    }
}
```

Will send a log to Logz.io that looks like this:
```
{
    "message": "This log will hold the MDC data as well",
    "Key": "Value",
    ... (all other fields you used to get)
}
```

### Release notes
 - 1.0.9
   - Fixed an issue preventing the appender to restart if asked. Also, prevented hot-loading of logback config.xml
 - 1.0.8
   - Fixed filesystem percentage wrong calculation (#12)
 - 1.0.7
   - Create different buffers for different types
   - Moved LogzioSender class to be a factory, by log type - meaning no different configurations for the same type (which does not make sense anyway)
 - 1.0.6
   - Fix: Appender can get into dead-lock thus causing all threads logging to bloc
   - Refactored all Unit tests 
   - Switched to maven wrapper for build consistency
 - 1.0.5
   - Add MDC support
   - Replace exception handling to use Logback own instead of implementing alone
   - Periodically calls BigQueue GC function so we can release files on local disk
 - 1.0.4
   - If you logged a throwable as well, we will put it inside a field named "exception"
 - 1.0.3
   - Addional fields support
   - Add hostname to logs support
 - 1.0.0 - 1.0.2
   - Initial releases
   

### Contribution
 - Fork
 - Code
 - ```mvn test```
 - Issue a PR :)
