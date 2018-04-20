---
layout: post
title: "AWS Elastic Beanstalk .ebextensions"
author: "kykkyn2"
---

### Elastic Beanstalk
요즘은 서버 인프라 구축을 대부분 AWS 환경으로 넘어가는 추세 인거 같다.
아무래도 배포 및 관리가 용이 하기 때문에 Linux 에서 하나하나 구축 하던 시절에서 이제는 버튼 클릭 몇번으로 서버 인프라 구성이 완성 된다. 그 만큼 시간을 아낄 수 있기 때문에 개발에 조금 더 집중 할 수 가 있다.
뭐 장단점이 있겠지만 난 좋다.
그중에 Elastic Beanstalk 의 단점 중 하나는 시스템 구성 변경에 제한적 이라는 점이다.

예를 들어 파일 업로드 기능을 해야 하는데 파일 크기가 1GB 라고 한다면 local 환경 (각 환경에 맞게 셋팅을 했다는 조건) 에서는 테스트가 잘 되다가 Elastic Beanstalk에 올리는 순간 부터 에러가 발생한다.

바로 413 Request Entity Too Large 다. 필자 Beanstalk Proxy Server는 NGINX로 구성 되어 있다.
(Beanstalk 은 Proxy Server 로 Apache 와 Nginx 을 지원 한다.)

AWS EC2 였다면 /etc/nginx/nginx.conf 에서 **client_max_body_size 1024M;** 추가만 해주면 해결할 수 있었지만 Beanstalk 에서는 일시적으로 변경은 가능 하지만 서버가 재 시작 되거나 하면 설정이 다 초기화 된다.

위 와 같은 단점을 보완 하기 위해 AWS 에서는 **.ebextensions** 을 제공 합니다.

### .ebextensions

[.ebextensions 문서 참조](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/ebextensions.html)

아래 정보는 Spring Boot + Maven + Nginx 로 구성 되어 있습니다.

예를 들어 Nginx Proxy Server 에서 파일 크기를 1GB로 받게 변경 하기 위해 ebextensions 설정은 아래와 같이 하면 됩니다.

[프록시 문서 참조](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/java-tomcat-proxy.html)

    src/main/resources 아래에 .ebextensions/nginx/conf.d/elasticbeanstalk 디렉토리를 생성
    (.으로 시작하는 폴더가 생성이 안되는 경우도 있습니다. 그럴때는 . 을 빼고 생성 하셔도 됩니다.)
    저도 .빼고 생성 했습니다.
    위 디렉토리에 example.conf 생성
    

*nginx.conf 파일을 재 정의(override) 해서 사용 하고 싶으면 .ebextensions/nginx 에 nginx.conf 파일을 작성해서 재 정의 하면 됩니다.*

example.conf
```bash
client_max_body_size 1024M;
```

위 와 같이 정의 한 후 war 파일로 묶어 배포 하기 위해서 maven-plugin 을 사용하겠습니다.
```xml
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <configuration>
        <webResources>
            <resource>
            	<!-- . 으로 만드신 분은 .ebextensions 파일 경로 까지만 적용-->
                <directory>src/main/resources/ebextensions</directory>
                <targetPath>.ebextensions</targetPath>
                <filtering>true</filtering>
            </resource>
        </webResources>
    </configuration>
</plugin>
```

** targetPath 에 .ebextensions 로 설정한 이유는 resources 파일에서 ebextensions 을 . 으로 설정 하지 않았기 때문 입니다.
.ebextensions 으로 디렉토리를 생성 하셨다면 위에 targetPath 는 생략 하셔도 됩니다.
**

maven 빌드 후 war 파일 생성시 아래와 같은 구조로 생성이 되어야 합니다.

<pre>
.
├── .ebextensions
├── META-INF
├── org
└── WEB-INF
</pre>

AWS Beanstalk 배포 후 /etc/nginx/conf.d/elasticbeanstalk 을 확인해 보면 아래와 같이 example.conf 가 포함 되어 있습니다.

```shell
-rw-r--r-- 1 root root 407 Apr 19 07:00 00_application.conf
-rw-r--r-- 1 root root   1 Apr 19 07:00 01_static.conf
-rw-r--r-- 1 root root 277 Apr 19 07:00 example.conf
-rw-r--r-- 1 root root 216 Apr 19 07:00 healthd.conf
```
이제 maven 빌드 후 war 파일을 재 배포 해도 위와 같은 설정은 계속 해서 유지 되는 것을 볼 수 있습니다.

ebextensions 을 리서치 하다보니 Commands (ec2 인스턴스에서 구동되는 실행 명령) 사용 방법도 나오는데 한번 찾아보고 적용 해보고 싶다.
