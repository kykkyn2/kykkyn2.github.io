---
layout: post
title: "AWS Elastic Beanstalk Load Balancing And Auto Scaling"
author: "kykkyn2"
---

### AWS Beanstalk ?

aws는 수 없이 많은 서비스 들을 제공 하고 있다. 그 중 aws beanstalk 은 Application 을 빠르게 배포 하고 관리 할 수 있는 서비스다.
beanstalk 는 배포를 하면 자체적으로 Load Balancing, Auto Scaling, Monitoring 등을 손 쉽게 사용 할 수 있는 장점이 있다.

뭐 기타 다른 장점이 있지만 자세한 사항은 [aws 공식 문서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/Welcome.html) 를 확인 해 보자.

### Beanstalk Setting (SpringBoot war 배포 기준)
aws 계정이 있다는 가정 아래 진행 하겠습니다.

[1]. Application 생성하기 전 test 로 생성 했습니다.

[2]. 환경 티어 선택
 ![blog-1](https://user-images.githubusercontent.com/5660626/47548727-daf7cd80-d934-11e8-8ab2-a84590c5ecfd.png)

[3]. 환경 정보 및 기본 구성 설정
 ![blog-2](https://user-images.githubusercontent.com/5660626/47549108-ffa07500-d935-11e8-8f35-0f5734ca1a13.png)
  * aws 에서는 docker, go, php, node, java, tomcat 등등 다양한 플랫폼을 제공합니다.  
  저는 test 용으로 tomcat 을 사용 했습니다.  
  애플리케이션 코드는 샘플을 사용하셔도 되고 war 파일 만들어서 업로드 하셔도 됩니다.  
  
   * 배포 완료된 상태
   ![blog-3](https://user-images.githubusercontent.com/5660626/47626330-e550dc00-db6d-11e8-8d2d-f3dfe94208ca.png)
   
   * 보통 여기까지만 적용 하셔도 테스트 하시는데에는 큰 문제가 없습니다. 하지만 우리는 로드벨런스 및 오토 스케일링 기능을 사용해 보려고 합니다.
   
[4]. 만든 환경에 구성을 클릭 하시면 맨 오른쪽에 용량 -> 수정 버튼 클릭해 줍니다. 
 * 환경 유형 현재는 단일 인스턴스 입니다. 로드 밸런싱 수행 으로 변경해 줍니다.
 
 * 인스턴스는 최소 몇대부터 최대 몇대 까지 오토스케일링을 할 것인지 설정 하는 부분입니다.  
 ![blog-4](https://user-images.githubusercontent.com/5660626/47626793-3feb3780-db70-11e8-9e16-cd9a16e24474.png)  
   
   <pre>로드 밸런싱을 사용 하려면 최소 2개 이상의 인스턴스가 각기 다른 가용 영역을 가져야 합니다.</pre>
 
 * 조정 트리거 셋팅  
 ![blog-5](https://user-images.githubusercontent.com/5660626/47626910-cef84f80-db70-11e8-95a3-066aaa6660a7.png)  
   
   지표 는 오토 스케일랑 할때 무슨 조건으로 할 지 나타내는 것이다. 각자 서비스 환경에 맞게 셋팅 하면 됩니다.
   필자는 보통 CPUUtilization 를 많이 사용 합니다. CPU 사용량이 올라갈때 Scale-Up 내려가면 Scale-Out 처리는 한다,
   각자 타입에 맞게 설정 한 후 생성을 누르면 일정 시간 지난 후 ec2 인스턴스로 가서 로드 밸런서 탭을 클릭하고 설정을 확인해 보자.
   
 * 인스턴스 상태를 확인 해보자  
 ![blog-6](https://user-images.githubusercontent.com/5660626/47627276-6ad68b00-db72-11e8-8b37-d75e9e895d5e.png)  
   
   2대의 서비스가 돌고 있는거를 확인 할 수 있다.
   로드 밸런싱 및 오토 스케일링 이 되고 있는거를 확인해 보려면 지표에 맞게 테스트를 하면 됩니다.
   
[5]. 위 내용 처럼 beanstalk 는 간단하게 load balancing 및 auto scaling 을 제공합니다. 안 쓸 이유가 없죠? 전 그래서 거의 모든 서비스를 beanstalk 로 사용 하고 있답니다. 

### 마무리
필자는 이 글 작성 기준 beanstalk 에서 로드 밸런서 생성시 자동으로 타입을 classic 타입으로 생성 된다. 언제쯤 Application 타입으로 생성 될런지 커스텀을 물론 하면 되지만
자동으로 되면 (일부 셋팅은 필요 하겠지만) 얼마나 좋겠냐? (이미 되고 있다면 알려주세요 ㅠ) 으흐흐 aws 화이팅

