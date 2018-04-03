logback.xml
````xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds">
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <jmxConfigurator/>
    
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <Target>System.out</Target>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>%d - [%t] %-5p %c:%L %X - %m%n</pattern>
        </encoder>
    </appender>
    
    <appender name="DAILY_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <Append>true</Append>
        <File>${logs.dir}/celltower.log</File>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>%d - [%t] %-5p %c:%L %X - %m%n%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logs.dir}/celltower.log.%d{yyyy-MM-dd}</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
    </appender>
    
    <appender name="ROLLING_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${logs.dir}/celltower-error.log</File>
        <Append>true</Append>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>%d - [%t] %-5p %c:%L %X - %m%n%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logs.dir}/celltower-error.log.%d{yyyy-MM-dd}</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
    </appender>

    <logger name="org.springframework" level="INFO"/>
    <logger name="com.alibaba.dubbo" level="INFO" />

    <root level="INFO">
        <!--<appender-ref ref="CONSOLE"/>-->
        <appender-ref ref="DAILY_FILE"/>
        <appender-ref ref="ROLLING_FILE"/>
    </root>

    <!-- 批量请求授权耗时日志 -->
    <appender name="batchRequestOpenTimesAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <Append>true</Append>
        <File>${logs.dir}/batchRequestOpenTimes.log</File>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>%d - [%t] %-5p %c:%L %X - %m%n%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logs.dir}/batchRequestOpenTimes.log.%d{yyyy-MM-dd}</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
    </appender>

    <logger name="batchRequestOpenTimes" level="INFO" additivity="false">
        <appender-ref ref="batchRequestOpenTimesAppender" />
    </logger>

</configuration>
````
application.properties
````html
logging.config=classpath:logback.xml
````
````java
public class Test {
    private static Logger logger = LoggerFactory.getLogger(BatchRequestOpenProcessor.class);
    private static Logger batchRequestOpenTimesLogger = LoggerFactory.getLogger("batchRequestOpenTimes");

    public void log() {
        logger.info("root log.......");
        batchRequestOpenTimesLogger.info("batchRequestOpenTimesLogger..........");
    }
}
````
