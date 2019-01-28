---
layout: post
title: "AWS CloudWatch To Slack"
author: "kykkyn2"
---


# 1. AWS CloudWatch

* Aws 에서 제공하는 [Application Monitoring](https://aws.amazon.com/ko/cloudwatch/) 시각화 서비스  
기본적인 모니터링은 무료? 정확히는 무료는 아니다. 어차피 서비스 (ec2,beanstalk 등)를 사용 해서 비용을 내야 하기 때문이다. 어쨌든 기본료에 간단한 시각화 (극히 제한된) 모니터링을 제공한다.  
유료로 요금을 지불 하면 훨씬 많이 데이타를 볼수 있다.

## 1-1. CloudWatch To Slack
* CloudWatch 기능 중 가장 강력한 기능이라고 생각하는 경보 기능 !!  
예를 들어 RDS 사용시 용량이 꽉 차갈때 쯤 경보를 발생 시킬수 있다. aws 에서는 알림 제공 기능으로 SMS 및 이메일만 제공하고 있다. 하지만 오늘 필자가 사용할 것은 [slack](https://slack.com/features) 이다. slack 은 필자가 회사 및 스터디 커뮤니티 channel 로 사용하고 있다. 카카오톡 보다 많이 사용 하는 듯 하다.  
CloudWatch 에서 제공하는 알림 기능을 Slack 으로 받아보자.

## 1-2. Aws lambda 를 사용해서 slack 연결하는 방법
* slack notification 을 받기 위해서는 여러 방법이 있지만 제일 편한 방법으로 aws lambda 를 사용하는 방법이 있다.
1. [Slack](https://api.slack.com/incoming-webhooks) 으로 이동 후 Webhook URL 을 생성 받자.  

    ![blog-1](https://user-images.githubusercontent.com/5660626/51731800-70833480-20bf-11e9-8c2d-2c7ea2c2a80b.png)

2.  [Aws Cli](https://aws.amazon.com/ko/cli/) 를 통해 [kms](https://docs.aws.amazon.com/ko_kr/kms/latest/developerguide/overview.html) 에 키를 생성 하고 키를 저장 해 둡니다.  
 
    ```bash
    aws kms create-key --region ap-northeast-2
    ```
    ![blog-2](https://user-images.githubusercontent.com/5660626/51750163-8ced9400-20f4-11e9-8084-2e81b8499bce.png)

    ```bash
    aws kms create-alias --alias-name "alias/SlackKey" 
    --target-key-id "위에서 저장한 KeyId를 넣어줍니다." --region ap-northeast-2
    ```

    ```bash
    aws kms encrypt --key-id alias/SlackKey --plaintext 
    "hooks.slack.com/services/1번 에서 받은 slack 주소를 넣어줍니다." --region ap-northeast-2
    ```

    ![blog-3](https://user-images.githubusercontent.com/5660626/51750854-a263bd80-20f6-11e9-9124-26444774f4dc.png)

3. Aws lambda 접속 후 함수를 생성 하고 블루프린트 및 Aws Repository 에서 slack 을 검색해 취향에 맞는 언어로 (nodejs, python 등등) 설정 하면 된다. 필자 회사에서는 python 이니깐 같은 python 으로 하겠습니다.  
 
    ![blog-4](https://user-images.githubusercontent.com/5660626/51802264-abd25e80-228b-11e9-9db3-1a674a3bed8c.png)

    ![blog-5](https://user-images.githubusercontent.com/5660626/51802342-75e1aa00-228c-11e9-8c61-a59dd2581f5b.png)

4. 각 서비스에 맞게 모니터링 -> 경보 생성을 할 수 있습니다.  
 
    ![blog-6](https://user-images.githubusercontent.com/5660626/51801758-41b6bb00-2285-11e9-8011-3843e0c4f51d.png)

    아래와 같이 알림 받을 대상에서 이전에 만든 to slack 으로 설정 하시고 테스트로 CPU 사용량 10% 이상일 경우 알람을 받을 수 있게 설정 하겠습니다.

    ![blog-7](https://user-images.githubusercontent.com/5660626/51812208-eec81c80-22f3-11e9-8772-a82ea4d9037b.png)

5. CPU 부하 주기위해 유용한 stress test 를 해보겠습니다.  
 
    ```bash
    sudo yum install -y stress
    ```
    ```bash
    stress --cpu 1 --timeout 300
    ```
6. slack 에 알림 오는거 확인  
 
    ![blog-8](https://user-images.githubusercontent.com/5660626/51812435-d7d5fa00-22f4-11e9-8402-3cf5ff2fdd83.png)

## 마치면서
lambda 설정을 통해 SNS, KMS 등 한다고 하니 복잡 할꺼 같았지만 차근차근 접근해 보니 어렵지 않았다. 오히려 하나하나 이해 하는데 더 큰 도움이 되었다. 이렇게 서비스 하나씩 알람을 설정하면 최소한 서버가 멈추거나 죽기전에 대응을 할 수 있을꺼 같다. 개인적으로나 팀으로도 유용하게 쓰일수 있을꺼 같습니다.

## 도움링크
> https://aws.amazon.com/ko/blogs/korea/slack-devops-with-aws-lambda-and-eb/