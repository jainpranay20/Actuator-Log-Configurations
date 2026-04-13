# Learning Notes: Actuator and Logging

This file is written as a study note for learning the two main concepts shown in this project:

- Spring Boot Actuator
- Application Logging

The goal is to understand what these features do, why they matter, and what this project teaches through them.

## 1. What I Learned About Actuator

### Actuator is for observability

Actuator gives ready-made endpoints that help inspect the application while it is running. Instead of writing custom monitoring APIs, Actuator provides standard endpoints that expose useful runtime information.

From this project, the core idea is:

**Actuator helps us see what is happening inside the application from outside the application.**

### Main Actuator idea in this project

This project mainly teaches **configuration-based Actuator usage**.

It does not create custom Actuator endpoints. Instead, it shows how much can already be done just by:

- adding the Actuator dependency
- exposing endpoints
- enriching the info endpoint
- enabling environment visibility

### Important Actuator properties and what they teach

#### `management.endpoints.web.exposure.include=*`

Learning:

- all endpoints are exposed for easy testing and learning
- useful for local understanding
- risky for production if not restricted

Lesson:

Actuator endpoint exposure should always be intentional.

#### `management.info.env.enabled=true`

Learning:

- the `/actuator/info` endpoint can include environment-related details

Lesson:

The info endpoint is customizable and can become much more useful than a simple static message.

#### `info.app.author`, `info.app.version`, `info.app.documentation`

Learning:

- custom metadata can be attached to the app

Lesson:

The `/actuator/info` endpoint is useful for operations teams, release tracking, and quick app identification.

#### `management.endpoint.env.show-values=always`

Learning:

- the env endpoint can show actual values instead of hiding them

Lesson:

This is great for debugging but dangerous for secrets. Convenience and security must be balanced.

#### `management.info.build.enabled=true`
#### `management.info.git.enabled=true`
#### `management.info.java.enabled=true`
#### `management.info.os.enabled=true`

Learning:

- Actuator can include system and build metadata without custom code

Lesson:

Observability can come from configuration, not only from business logic.

## 2. Which Actuator Endpoints Matter Most Here

### `/actuator`

Learning:

- use it as the starting point
- it tells you which endpoints are exposed

### `/actuator/health`

Learning:

- use it to check whether the application is running correctly
- useful for uptime monitoring and readiness checks

### `/actuator/info`

Learning:

- use it to see custom application metadata and environment/build/system details

### `/actuator/env`

Learning:

- helps debug configuration and property sources

Important thought:

- this endpoint is powerful, but it can leak sensitive information

### `/actuator/loggers`

Learning:

- logger configuration is observable at runtime
- this directly connects Actuator with the logging part of the project

### `/actuator/metrics`

Learning:

- Actuator can expose runtime metrics without building custom code for them

## 3. What I Learned About Logging

### Logging is not just printing text

Before learning proper logging, it is easy to treat logs like random console messages. This project teaches that logging should be:

- structured
- level-based
- configurable
- persistent

### Logging should answer questions like

- what happened
- when it happened
- where it happened
- how serious it is
- what failed

### Important logging properties in this project

#### `logging.level...clients=trace`

Learning:

- a specific package can be made extremely detailed
- this is useful when one module needs heavy debugging

Lesson:

Logging levels can be controlled at package/class granularity.

#### `logging.level...config=INFO`

Learning:

- different packages can have different logging depth

Lesson:

Not every package should log with the same verbosity.

#### `logging.pattern.console=...`

Learning:

- log appearance can be customized

Lesson:

Readable logs save debugging time.

#### `logging.file.name=application.log`

Learning:

- logs can be persisted to files instead of only console output

Lesson:

File logging matters for real applications because console output alone is not enough.

#### `logging.config=classpath:logback-spring.xml`

Learning:

- custom Logback setup can fully control logging behavior

Lesson:

Spring Boot defaults are helpful, but production logging usually needs deeper customization.

## 4. What I Learned From `logback-spring.xml`

This file is one of the most important learning points in the project.

### Custom log pattern

Learning:

- logs can be formatted in a consistent and readable way
- useful parts include timestamp, level, class, and message

### Rolling file appender

Learning:

- logs should not stay forever in a single file
- rotating files prevents unlimited log growth

### Time-based rolling

Learning:

- log files can be organized by date

### Size-based rolling

Learning:

- even inside one day, a large file can be split once it crosses a size limit

### Log retention

Learning:

- old logs can be retained only for a limited period

Lesson:

Good logging is not only about producing logs. It is also about managing storage and retention.

## 5. What I Learned From `EmployeeClientImpl`

This class is the clearest practical example of logging in the project.

It shows how logs can be used at multiple levels:

- `trace` at method start or for detailed values
- `info` before important actions
- `debug` after successful steps
- `error` when exceptions happen

### Why this is useful

If a remote call fails, logs can tell:

- which method started
- what was being attempted
- whether the request succeeded or failed
- what the response body contained
- what exception occurred

That makes diagnosis much faster.

### Best lesson from this class

Different levels should have different meaning:

- `TRACE` = deepest technical detail
- `DEBUG` = developer troubleshooting detail
- `INFO` = important application flow
- `WARN` = suspicious but not broken
- `ERROR` = failure

## 6. Biggest Combined Learning

The strongest idea from this project is:

**Actuator tells us about the running system, while logs tell us about the running behavior.**

Together they answer different questions:

- Actuator: Is the app healthy? What config is loaded? What endpoints and metrics exist?
- Logs: What exactly happened during execution? What failed? Which class produced the issue?

This combination is what makes an application easier to operate in real life.

## 7. What To Say In Interview

If someone asks what you learned from this project, a strong answer is:

> I learned how to make an application observable using Spring Boot Actuator and structured logging. Actuator helped expose health, info, env, metrics, and logger-related endpoints, while Logback and package-level logging helped me control how much detail was logged, where it was stored, and how it rotated over time.

## 8. Final Summary

From this project, the key learnings are:

- how to add and expose Actuator endpoints
- how to enrich `/actuator/info`
- how to inspect runtime configuration through Actuator
- how to tune log levels package by package
- how to use Logback for rolling file logs
- how to design logs for debugging and support
- how observability depends on both monitoring endpoints and useful logs

If I had to compress the whole learning into one sentence, it would be:

**Actuator shows the state of the application, and logging shows the story of the application.**
