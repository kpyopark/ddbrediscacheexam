# Redis Session 구성하기 전에...

현재 Cloud9 Instance에 설치되어 있는 JPetStore는 단일 인스턴스에 Local Database, Local Session Manager를 이용하여 구성되어 있습니다. 즉 하나의 Local Instance가 죽을 경우, 어떤 가용성 보장도 제공하지 않습니다. 
즉, 가용성을 보장 받기 위해서는 자원을 분리하고, 인스터를 N개로 확장하고, User Endpoint를 고가용성이 보장되는 ELB로 구성하여야 합니다. 
이를 위해서 간단하게, Elastic Beanstalk를 이용하여 위에서 생성한 JPetStore를 N개의 Instance에 배포해 보겠습니다. 

## Elastic Beanstalk Sample Application 배포해 보기

Elastic BeanStalk은 초기 환경을 생성하면서, 필요한 Managed Role / Managed Policy를 생성합니다. 따라서 Console상에서 Sample Application을 배포해볼 필요가 있습니다. 

1. Elastic Beanstalk 서비스로 이동합니다. 

2. "Create Application" 버튼을 선택합니다. 

![image](https://user-images.githubusercontent.com/9047122/85353221-accff400-b542-11ea-8beb-3f6c3260d947.png)

3. 아래와 같이 내용을 채워 넣습니다. 
 - "Application Name" : SampleApp
 - "Platform" : Java
 - "Platform branch" : Corretto 11 running on 64bits Amazon Linux 2
 - "Platform version" : 3.0.2
 - "Application Mode" : Sample Application (Check)

![image](https://user-images.githubusercontent.com/9047122/85353369-06382300-b543-11ea-8edb-ab660a7a0a01.png)

4. "Create application" 버튼을 클릭합니다. 
 ** 일단 오류가 발생해도, 왼쪽 메뉴의 Environments를 선택하였을 경우, 아래와 같은 목록이 표시되면 괜찮습니다. 
 
![image](https://user-images.githubusercontent.com/9047122/85353625-a5f5b100-b543-11ea-828c-a6daf9b108fb.png)

5. 여기까지 수행되면, 다음에 진행되는 CLI 형태의 접근이 원활하게 수행됩니다. 


## Elastic Beanstalk CLI 설정하기

1. Cloud 9 instance에 eb cli를 설치합니다. Cloud 9 Terminal에서 아래 내용을 실행시키십시요. 

```
cd ~
sudo yum install "Development Tools" zlib-devel openssl-devel ncurses-devel libffi-devel sqlite-devel.x86_64 readline-devel.x86_64 bzip2-devel.x86_64 -y
git clone https://github.com/aws/aws-elastic-beanstalk-cli-setup.git
# the below installation takes more 5 minutes.
./aws-elastic-beanstalk-cli-setup/scripts/bundled_installer
echo 'export PATH="/home/ec2-user/.ebcli-virtual-env/executables:$PATH"' >> ~/.bash_profile && source ~/.bash_profile
echo 'export PATH=/home/ec2-user/.pyenv/versions/3.7.2/bin:$PATH' >> /home/ec2-user/.bash_profile && source /home/ec2-user/.bash_profile

```

2. ElasticBeanstalk은 배포가능한 Application 패키지를 이용하여, 앞단에 ELB를 Backend에는 Elastic Beanstalk에서 관리하는 EC2 Instance에 배포를 자동적으로 진행해 줍니다. ELB 는 기본적으로 80 Port를 사용하도록 구성되어 있으며, Backend Application은 5000 Port를 이용하여, 통신하도록 되어 있습니다. 이를 위하여 JPetstore의 application.properties 파일을 수정합니다. 

```
cd ~/environment/mybatis-spring-boot-jpetstore
echo "server.port=5000" >> src/main/resources/application.properties
```

3. Elastic Beanstalk을 초기화 합니다. (eb init) !!!주의!!! 반드시 ap-northeast-2 (seoul) 을 선택하셔야 합니다 !!!

```
ec2-user:~/environment/mybatis-spring-boot-jpetstore (master) $cd ~/environment/mybatis-spring-boot-jpetstore
ec2-user:~/environment/mybatis-spring-boot-jpetstore (master) $ eb init
```
아래 내용이 나왔을 때 반드시, 10 (ap-northeast-2)를 선택하십시요.
```
Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-south-1 : Asia Pacific (Mumbai)
7) ap-southeast-1 : Asia Pacific (Singapore)
8) ap-southeast-2 : Asia Pacific (Sydney)
9) ap-northeast-1 : Asia Pacific (Tokyo)
10) ap-northeast-2 : Asia Pacific (Seoul)
11) sa-east-1 : South America (Sao Paulo)
12) cn-north-1 : China (Beijing)
13) cn-northwest-1 : China (Ningxia)
14) us-east-2 : US East (Ohio)
15) ca-central-1 : Canada (Central)
16) eu-west-2 : EU (London)
17) eu-west-3 : EU (Paris)
18) eu-north-1 : EU (Stockholm)
19) eu-south-1 : EU (Milano)
20) ap-east-1 : Asia Pacific (Hong Kong)
21) me-south-1 : Middle East (Bahrain)
22) af-south-1 : Africa (Cape Town)
(default is 3): 10
```
 Enter를 쳐서 Default값으로 고정합니다. 
```
Enter Application Name
(default is "mybatis-spring-boot-jpetstore"): 
```
 Platform은 Java로 선택하십시요 (5 번)
```
Select a platform.
1) .NET on Windows Server
2) Docker
3) GlassFish
4) Go
5) Java
6) Node.js
7) PHP
8) Packer
9) Python
10) Ruby
11) Tomcat
(make a selection): 5
```
 Platform branch에는 1번을 선택합니다. 
```
Select a platform branch.
1) Corretto 11 running on 64bit Amazon Linux 2
2) Corretto 8 running on 64bit Amazon Linux 2
3) Java 8 running on 64bit Amazon Linux
4) Java 7 running on 64bit Amazon Linux
(default is 1): 1
```
 CodeCommit은 사용하지 않기 때문에, 그냥 Enter를 칩니다.
```
Do you wish to continue with CodeCommit? (y/N) (default is n): 
```
 SSH 연결은 필요할 수도 있으므로, Default 값을 이용합니다. Enter를 입력합니다. 
```
Do you want to set up SSH for your instances?
(Y/n): 
```
 Key Pair를 생성을 선택하고, jpetstore-keypair 를 입력합니다. (임의로 입력해도 상관 없습니다.) 
```
1) [ Create new KeyPair ]
Type a keypair name.
(Default is aws-eb): jpetstore-keypair
```
 Keypair에서 사용하는 Key Phrase는 빈공란으로 입력합니다. Enter를 두번 입력합니다. 
```
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:

```

3. 추가된 application.properties 파일이 포함된 jar package를 생성합니다. 

```
cd ~/environment/mybatis-spring-boot-jpetstore
./mvnw clean 
./mvnw package -Dmaven.test.skip=true
```

4. 생성된 jar package를 elasticbeanstalk에 등록할 수 있도록, yaml파일에 명시합니다. 

```
cd ~/environment/mybatis-spring-boot-jpetstore
echo "deploy:" >> .elasticbeanstalk/config.yml
echo "  artifact: target/mybatis-spring-boot-jpetstore-2.0.0-SNAPSHOT.jar" >> .elasticbeanstalk/config.yml
```

5. eb create 명령어를 이용하여 Deploy를 수행하십시요. 

 이름은 기본 값을 이용합니다. enter를 입력합니다. 
 
```
$ eb create
Enter Environment Name
(default is mybatis-spring-boot-jpetstore-dev): 
```

 CNAME Prefix값은 변경을 하여야 합니다. 원하는 cname prefix 값을 입력하십시요. 예를 들어 gildong-test 라는 Prefix를 입력하면, 향후에 gildong-test.ap-northeast-2.elasticbeanstalk.com URI가 실제 Site URL이 됩니다. 
 
```
Enter DNS CNAME prefix
(default is mybatis-spring-boot-jpetstore-dev): gildong-test
```

 Application Load Balancer를 선택합니다. 기본값이므로, enter를 입력합니다. 
 
```
Select a load balancer type
1) classic
2) application
3) network
(default is 2):
```

 Spot Fleet은 현재 HoL에서는 사용해도 무방하나, 일단은 기본값인 N으로 설정합니다. 
 
```
Would you like to enable Spot Fleet requests for this environment?
(y/N):
```

 배포를 수행합니다. 배포를 하면 아래와 같이, CNAME 값이 표시됩니다. 해당 CNAME값을 복사해 둡니다.
 
```
Environment details for: mybatis-spring-boot-jpetstore-dev
  Application name: mybatis-spring-boot-jpetstore
  Region: ap-northeast-2
  Deployed Version: app-6f9a-200623_022327
  Environment ID: e-9gxheywvfp
  Platform: arn:aws:elasticbeanstalk:ap-northeast-2::platform/Corretto 11 running on 64bit Amazon Linux 2/3.0.2
  Tier: WebServer-Standard-1.0
  CNAME: gildong-test.ap-northeast-2.elasticbeanstalk.com
  Updated: 2020-06-23 02:23:30.362000+00:00
Printing Status:
```

6. 배포가 완료되고, 정상적으로 기동되었다면, browser에 tab을 열고 위에서 복사한 URL을 입력하여, 화면을 표시합니다. 

7. 정상적으로 Site에 들어가고 기능 테스트를 수행하십시요. 정상적으로 수행되면 성공입니다. 
   수고하셨습니다. 현재까지 아래 아키텍처 까지 구성하셨습니다. 

![image](https://user-images.githubusercontent.com/9047122/85356921-28ce3a00-b54b-11ea-800b-0b5df7bb75e5.png)

## Instance 의 숫자를 늘릴경우, 발생하는 문제점 확인하기

현재 상태는 ALB 뒤에 하나의 Instance만을 운영하여, 문제가 보이지 않습니다. 만약 ALB 뒤에 Instance가 2개 이상 있다면, 어떤 문제점이 발생할까요?
문제점을 하나씩 확인해 보겠습니다. 

1. Elastic Beanstalk에서는 간단하게 Instance를 늘리거나 줄어들게 할 수 있습니다. 
   이를 위하여, eb scale 명령을 사용하면 됩니다. 아래와 같이 2개로 인스턴스 갯수를 늘려보십시요. 

```
$ eb scale 2
```

2. 먼저 브라우저로 JPetStore에 접근하시기 바랍니다. 이후 Login을 수행하십시요. 이상한 부분을 눈치채셨습니까? 여러 기능들을 수행해 보시기 바랍니다. 

   여러 기능들을 수행하다 보면, 다음과 같은 문제점들이 보입니다. 
   
   - 로그인을 하여도 로그인이 되지 않는 경우
   
   - 로그인 정보가 누락되는 경우
   
   - 카트 정보가 나타났다 사라지는 경우
   
   위 내용은 모두, Database 가 별도의 인스턴스에서 동작하고, Http Session이 독립적으로 동작하기 때문에 발생합니다. 

3. 현재까지 구성사항은 아래와 같습니다. 

![image](https://user-images.githubusercontent.com/9047122/85357839-3684bf00-b54d-11ea-86fb-5f3d85234314.png)


수고하셨습ㄴ다. 다음 챕터로 이동하십시요. [Lab2-Redis.md](Lab2-Redis.md)
