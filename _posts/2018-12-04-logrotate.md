---
layout: post
title: "logrotate 를 이용해 로그 관리 하기"
author: "kykkyn2"
---

### logrotate

최신 사내에서 catalina 파일을 볼일이 있었다. 하지만 무려 8GB 이거 그냥 vi로 열면.. ㅠㅠ
그래서 예전에 사용한 적이 있는 logrotate 를 적용 시키기로 했다. logrotate 는 서버 로그를 주기에 따라서 (일,주,월) 자동? 으로 관리해 준다. 역시 검색만 잘하면 편하다.
최신 리눅스 시스템에는 자동 설치 되어 있지만 버전에 따라 다르다.
확인 하는 방법은 bash 접속 후 아래와 같이 명령어를 날려보자.

``` logrotate -? ```

없다면 설치를 하면 된다.

``` sudo yum install logrotate ```

설정 파일은
> /etc/logrotate.conf
> /etc/logrotate.d/*

[logrotate 참고 사이트](https://linux.die.net/man/8/logrotate)

### 사용법
> vi /etc/logrotate.conf 파일을 열어보자.
> 이해를 쉽게 할수 있을것이다.
> include /etc/logrotate.d ?? 즉 저 아래 파일을 넣어두면 그냥 읽는구나 생각하시면 됩니다.
> 자. 그럼 /etc/logrotate.d/ 이동 후 vi tomcat 파일 생성 (이름은 자유)
> 
```
/목적지 경로/가즈아/logs/catalina.out {
    copytruncate
    missingok
    ifempty
    nocompress
    dateext
    create 664 ec2-user ec2-user
}
```
> 필자는 위 옵션으로 설정했다. hour, daily, weekly, monthly 옵션이 있지만 어차피 daily cron에 등록하기때문에 필요 없다고 생각 했다. 테스트 해본 결과 역시 잘 돌아간다.
> 엄청 많은 옵션이 있다. 꼭 문서를 확인해 보자 [관련 문서 바로가기](https://linux.die.net/man/8/logrotate)
> 간단한 옵션에 대한 설명
> > copytruncate : 대상 파일(catalina file)을 내용을 같이 복사하겠다는 뜻이다.
> > missingok : 로그 파일이 없어도 에러 발생 ㄴㄴ
> > ifempty : 로그파일이 비어있는 경우에도 로테이트 한다.
> > nocompress : 압축하지 않겠다.
> > dateext : catalina.out-20181204 이런식으로 date 붙힌다.
> > create 664 ec2-user ec2-user : ec2-user 로 664 권한 파일 생성


> ``` ls -al /etc/logrotate.d/ ``` 확인해 보자. tomcat 이라는 파일을 생성되었다.
> 
> ![log-1](https://user-images.githubusercontent.com/5660626/49414963-84de3b80-f7b8-11e8-9578-7276c42e49d4.png)

> ``` /etc/logrotate.d/* ``` 밑에 파일을 두면 자동으로 셋팅된다. 이유는 뭐 위에서 설명했다 싶이 logrotate.conf 에서 /etc/logrotate.d/* 파일들을 include 하고 있기 때문이다.
> ``` ls -al /etc/cron.daily ``` 확인을 해보자.
> 
> ![log-2](https://user-images.githubusercontent.com/5660626/49415344-b99ec280-f7b9-11e8-978e-6be83f25a5f0.png)

> 위 이미지에 있는거 처럼 logroate bash가 있다 그렇기 때문에 daily 로 cron이 실행 된다.
> hour,weekly,monthly 로 변경 하고 싶으면 해당 파일을 옮기거나 파일을 bash 파일로 만들어
> 각 해당 폴더에 옮겨주거나 그 해당폴더에서 생성 하면 된다.
> 
> ![log-3](https://user-images.githubusercontent.com/5660626/49415502-519cac00-f7ba-11e8-9a68-5dd2e6f2fbf4.png)

> 각자 cron 환경에 맞게 돌리면 됩니다.
> 저는 daily 로 작업을 했습니다. 정상적으로 반영된다면
> 아래와 같은 화면을 볼수 있습니다.
> ![log-4](https://user-images.githubusercontent.com/5660626/49415646-b0622580-f7ba-11e8-8184-ae817fe31f68.png)

### 마무리

log 파일을 날짜별로 나누고 나니 catalina file 사이즈가 엄청 작아졌다. catalina file 은 대부분 쉬쉬하는 경향이 있다. 그러다가 서버가 다운 되고 나서야 수습한다. 대부분 그렇게 할 것이다. 왜냐? 우리는 항상 바쁘니깐..

여기서 마무리 하지 말고 저 날짜별로 나눈 파일을 어떻게 할 것인가? 고민 해 볼 필요가 있다.
필자는 SFTP 로 내가 접속해서 일일히 백업을 할 것인가? 아니면 자동화를 할텐가? 뭐 방법이야 여러가지가 있다.
나는 자동화를 하는것을 좋아하므로 저 파일들을 한달에 한번 묶어서 aws cli를 이용해 s3에 배포를 한다.
bash 파일 하나 만들어서 cron job으로 등록 했다. 이제 신경 안써도 된다? ㅎㅎ

다들 서버 다운 되지 않도록 기도 합시다 :)