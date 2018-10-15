---
layout: post
title: "Spring Boot Logback Slack Appender"
author: "kykkyn2"
---

### Logback Slack Appender

[Slack Appender](https://github.com/maricn/logback-slack-appender)

개발을 하다가 보면 로그를 찍어서 확인 할 일이 많이 있다.
local 에서는 log 보면서 개발을 할 수 도 있고, real (prd) 에서는 error 를 빠르게 확인 할 수 있어야 한다.

log 를 특정 공간에 적재 해 두어야 할때는 s3에 올리기도 하고, notification 용으로는 Email, SMS, Messenger 등 을 활용 할 수 가 있다.

slack 뿐만 아니라 discord 및 hipchat 등 webhook 기능을 제공하는 Vender 제품은 다 사용 할 수 있다.
이 글에서는 Messenger 를 사용 할 계획이다. 가장 많이 사용 하고 있는 slack 을 사용 하겠다.


## Slack Setting

1. [API Slack](https://api.slack.com/apps) 가셔서 Create New App 을 합니다.

2. Slack App 생성
![blog-1](https://user-images.githubusercontent.com/5660626/46941114-5213a300-d0a5-11e8-8d76-07136c3fe240.png)

3. Incoming Webhooks 클릭
![blog-2](https://user-images.githubusercontent.com/5660626/46941570-6310e400-d0a6-11e8-8733-da359241f402.png)

4. Activate Incoming Webhooks On 으로 켜주세요. 
![blog-3](https://user-images.githubusercontent.com/5660626/46941302-c9e1cd80-d0a5-11e8-9f6f-0a8177e5bb7d.png)

5. Webhook URL	복사해 두세요.

## Spring boot Setting (maven)

1. slack appender 추가

```xml
<dependency>
    <groupId>com.github.maricn</groupId>
    <artifactId>logback-slack-appender</artifactId>
    <version>1.4.0</version>
</dependency>
```

2. logback.xml 생성

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>
  <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
  <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}/}spring.miningLogs}"/>
  <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
  <include resource="org/springframework/boot/logging/logback/file-appender.xml"/>

  <appender name="SLACK" class="com.github.maricn.logback.SlackAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
      <pattern>%-4relative [%thread] %-5level %class - %msg%n</pattern>
    </layout>
    <webhookUri>https://hooks.slack.com/services/token</webhookUri>
    <username>kykkyn2-logger</username>
    <iconEmoji>:stuck_out_tongue_winking_eye:</iconEmoji>
    <colorCoding>true</colorCoding>
  </appender>

  <!-- Currently recommended way of using Slack appender -->
  <appender name="ASYNC_SLACK" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="SLACK"/>
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      <level>WARN</level>
    </filter>
  </appender>

  <root level="INFO">
    <appender-ref ref="CONSOLE"/>
    <appender-ref ref="FILE"/>
    <appender-ref ref="ASYNC_SLACK"/>
  </root>
</configuration>
```

3. webhookUri 에 {token} 이부분은 아까 복사 해둔 것으로 변경

4. log level 을 WARN 이상(?) 으로 설정을 했다.

5. warn 으로 테스트 진행
![blog-4](https://user-images.githubusercontent.com/5660626/46942471-5f7e5c80-d0a8-11e8-99ac-505f77de870b.png)

6. slack 확인
![blog-5](https://user-images.githubusercontent.com/5660626/46942476-61482000-d0a8-11e8-9a7d-d84814ee9ac7.png)


**테스트 코드는 [링크](https://github.com/kykkyn2/spring-boot-slack-appender)**

### Ending

slack 으로 알림을 받기 시작하고 나서 에러 대응이 빨라졌다.
대부분 메신저는 히스토리가 일정 시간이 지나면 지워진다. (유료로 사용하면 된다.)
메신저는 notification 용 으로 사용 하고 log 저장은 별도로 사용하자!


