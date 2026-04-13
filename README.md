# Actuator and Logging Guide

This project is a focused demonstration of **Spring Boot Actuator configuration** and **application logging setup**. The main learning value is not the business logic itself, but how the application is made more observable and production-friendly through monitoring endpoints and structured logs.

## What This Project Demonstrates

The project shows two main things:

1. How to expose and configure **Actuator endpoints**
2. How to configure **logging levels, log patterns, and rolling file logs**

The most important files for these topics are:

- `pom.xml`
- `src/main/resources/application.properties`
- `src/main/resources/logback-spring.xml`
- `src/main/java/com/actuator/pranay/prod_ready_features/prod_ready_features/clients/impl/EmployeeClientImpl.java`

## Actuator

### Where Actuator is enabled

Actuator is added through this dependency in `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

This enables Spring Boot’s production-ready monitoring endpoints.

### Actuator configuration in the project

In `application.properties`, these lines control Actuator behavior:

```properties
management.endpoints.web.exposure.include=*
management.info.env.enabled=true
info.app.author=pranay Kumar Sharma
info.app.version=1.0.0
info.app.documentation=https://actuator.com/docs
management.endpoint.env.show-values=always
management.info.build.enabled=true
management.info.git.enabled=true
management.info.java.enabled=true
management.info.os.enabled=true
```

### What each property does

#### `management.endpoints.web.exposure.include=*`

This exposes all web Actuator endpoints.

That means endpoints such as these become available:

- `/actuator`
- `/actuator/health`
- `/actuator/info`
- `/actuator/env`
- `/actuator/loggers`
- `/actuator/metrics`
- `/actuator/mappings`
- `/actuator/beans`
- `/actuator/configprops`

This is useful for learning, but in a real production system it is usually safer to expose only selected endpoints.

#### `management.info.env.enabled=true`

This allows environment-related info to appear in the `/actuator/info` response.

#### `info.app.author`, `info.app.version`, `info.app.documentation`

These custom properties enrich the `/actuator/info` endpoint with application metadata.

Example purpose:

- identify the app owner
- show version
- link documentation

#### `management.endpoint.env.show-values=always`

This tells the `/actuator/env` endpoint to show actual property values.

This is useful for debugging configuration, but it is sensitive and should be used very carefully because it can expose secrets or internal config values.

#### `management.info.build.enabled=true`

Adds build-related information to the info endpoint when available.

#### `management.info.git.enabled=true`

Adds git-related details when git metadata is available.

#### `management.info.java.enabled=true`

Adds Java runtime details.

#### `management.info.os.enabled=true`

Adds operating system details.

### What you learn from the Actuator side

This project teaches that Actuator is not just a dependency. It becomes useful only when you:

- expose endpoints intentionally
- decide what metadata should be visible
- decide how much environment detail should be shown
- use endpoints like health, info, env, metrics, and loggers for observability

### Most relevant Actuator endpoints in this project

#### `/actuator`

Shows the list of exposed Actuator endpoints.

#### `/actuator/health`

Used to verify whether the application is alive and whether key components like the datasource are healthy.

#### `/actuator/info`

Shows custom metadata configured through the `info.*` properties plus build, git, java, and os info if available.

#### `/actuator/env`

Shows application environment and property sources. In this project, values are configured to be visible.

#### `/actuator/loggers`

Shows current logger levels and is especially relevant for this project because logging is one of the main focuses.

## Logging

### Where logging is configured

Logging behavior is primarily controlled in:

- `application.properties`
- `logback-spring.xml`

### Logging settings in `application.properties`

```properties
logging.level.com.actuator.pranay.prod_ready_features.prod_ready_features.clients=trace
logging.level.com.actuator.pranay.prod_ready_features.prod_ready_features.config=INFO
logging.pattern.console=%d{dd-mm-yyyy hh:MM:ss} [%level] %c{1.} --- %m%n
logging.file.name=application.log
logging.config=classpath:logback-spring.xml
```

### What these logging settings mean

#### `logging.level...clients=trace`

This sets the logging level of the `clients` package to `TRACE`.

That means the most detailed logs from that package will be printed, including:

- trace logs
- debug logs
- info logs
- error logs

This is why `EmployeeClientImpl` is a key learning file in this project.

#### `logging.level...config=INFO`

This sets the `config` package to `INFO`.

So very detailed logs like trace/debug are not shown for configuration classes, but normal informational logs are allowed.

This teaches package-level logging control.

#### `logging.pattern.console=...`

This customizes how logs look in the console.

The configured pattern includes:

- date/time
- log level
- shortened class name
- actual message

This is useful because readable log format makes debugging easier.

#### `logging.file.name=application.log`

This enables log writing to a file named `application.log`.

#### `logging.config=classpath:logback-spring.xml`

This tells Spring Boot to use the custom Logback configuration file instead of only default logging behavior.

That is a very important line because it hands control over to `logback-spring.xml`.

## Logback Configuration

The custom Logback file is:

- `src/main/resources/logback-spring.xml`

### What it defines

#### Custom pattern

```xml
<property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} %-5level %c{1} - %msg%n" />
```

This creates a clean file-log format with:

- timestamp
- log level
- class name
- message

#### Log directory

```xml
<property name="LOG_PATH" value="logs" />
```

All rolling log files are stored inside a `logs` directory.

#### Rolling file appender

```xml
<appender name="ROLLING_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
```

This means logs are written to files and rotated instead of growing forever in one single file.

#### Time-based rolling policy

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <fileNamePattern>${LOG_PATH}/application.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
    <maxHistory>30</maxHistory>
```

This means:

- log files are grouped by date
- an index is added when multiple files are created on the same day
- logs are retained for 30 periods

#### Size-based rotation inside time windows

```xml
<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
    <maxFileSize>10MB</maxFileSize>
</timeBasedFileSizeAndTriggeringPolicy>
```

Conceptually, this means:

- log files rotate by date
- if a file becomes too large on the same date, a new indexed file is created
- max size is `10MB`

#### Root logger

```xml
<root level="debug">
    <appender-ref ref="ROLLING_FILE" />
</root>
```

This means:

- overall root logging level is `DEBUG`
- logs are sent to the rolling file appender

So by default:

- debug, info, warn, and error logs are written
- package-specific overrides can raise or lower detail

## Where Logging Is Used in Code

The most important code example is:

- `EmployeeClientImpl.java`

This class demonstrates practical logging at multiple levels:

- `trace` when entering methods and printing detailed context
- `info` before external calls
- `debug` after successful operations
- `error` when exceptions or bad responses happen

### What is good about this example

It shows how different log levels are used for different purposes:

- `TRACE`: very detailed execution tracking
- `INFO`: meaningful business/process steps
- `DEBUG`: developer-level success details
- `ERROR`: failures and exception stack traces

That is exactly how structured logging should be thought about.

## Key Learnings From This Project

### Actuator learnings

- how to expose Actuator endpoints
- how to add app metadata to `/actuator/info`
- how to inspect runtime configuration via `/actuator/env`
- how `/actuator/loggers` helps inspect logger configuration
- how Actuator improves observability without writing custom monitoring code

### Logging learnings

- how to set package-level logging granularity
- how to create custom console log patterns
- how to use a dedicated `logback-spring.xml`
- how to configure rolling log files
- how to retain logs and limit file size
- how to use trace/debug/info/error properly in real code

## Practical Takeaway

The biggest lesson from this project is that **observability is a combination of runtime visibility and useful logs**.

Actuator gives visibility into:

- health
- info
- configuration
- metrics
- logger setup

Logging gives visibility into:

- what the code is doing
- what failed
- how far execution progressed
- what external calls returned

Together, Actuator and logging make the application easier to debug, monitor, and support.
