# DynamoDB / Redis session cache & Redis Result Cache example

DynamoDB를 이용한 Session Cache와, Redis 를 이용한 Session Cache의 차이점을 확인할 수 있는 HoL 가이드입니다. 

![image](https://user-images.githubusercontent.com/9047122/85213506-41e3b900-b39a-11ea-9936-7a8defb3934f.png)

# 테스트 환경 설정

 모든 HoL은, 기본적으로 Cloud9 IDE와 Linux 환경을 기본으로 수행합니다. Cloud 9 은 Browser에서 직접 Coding을 수행할 수 있는 Browser기반 IDE(Integrated Development Environment)이며, 추가적으로 Linux EC2 Instance기반하에 작업을 수행하기 때문에, 별도의 Linux Machine이 필요하지 않습니다. 
 
1. 먼저 Cloud 9 서비스를 구성하기 위하여, Console에서 위와 같이 Cloud9 서비스를 입력하고 엔터를 치면, 해당 서비스 화면으로 이동합니다. 
![image](https://user-images.githubusercontent.com/9047122/85213530-a7d04080-b39a-11ea-929e-e97b33dc2626.png)


2. 화면 상단에 보이는 "Create Environment" 버튼을 클릭하십시요. 
![image](https://user-images.githubusercontent.com/9047122/85213580-48266500-b39b-11ea-983c-144b2a67d1bd.png)

3. 아래와 같이, 이름/설명에 대한 부분을 입력하시고 (임의로 입력하셔도 상관 없습니다. ) "Next Step"을 클릭하세요.

![image](https://user-images.githubusercontent.com/9047122/85213598-8c196a00-b39b-11ea-8c40-f3547adf6d55.png)

4. "환경 설정" 부분에서 특별하게 수정할 사항이 없습니다. 제일 하단에 있는 "Next Step"을 클릭하세요. 

5. 화면 하단에 있는 "Create Environment"를 클릭하시면, 아래와 같은 개발 환경 생성 화면이 보이고, 5분 후에, Cloud 9 IDE가 Browser에 표시됩니다. 

![image](https://user-images.githubusercontent.com/9047122/85213643-fb8f5980-b39b-11ea-99d8-d7730f84b34d.png)

6. 테스트에 필요한 일부 Toolkit을 설치합니다. Cloud 9 Terminal에서 아래 Command를 수행합니다. 

```
$ sudo yum install jq -y
```


# Example Application 을 Cloud 9 Instance에 설치하기, 

해당 HoL에서는 Spring JPetStore Example을 이용하여, DynamoDB / Redis session을 테스트해 보도록 하겠습니다.

1. 먼저 Cloud 9에 있는 Terminal을 이용하여 아래 내용을 입력하여, Java, JPetStore를 설치합니다. (COPY & PASTE). 

```
# Install java 11 - 5 minues
sudo rpm -ivh https://corretto.aws/downloads/latest/amazon-corretto-11-x64-linux-jdk.rpm
# Install sample application (Spring JPetStore) - 20 minutes
git clone https://github.com/kpyopark/mybatis-spring-boot-jpetstore
cd mybatis-spring-boot-jpetstore
./mvnw clean 
./mvnw package -Dmaven.test.skip=true
```

2. JpetStore를 Cloud 9이 설치되어 있는 Instance에서 실행합니다. 

```
java -jar target/mybatis-spring-boot-jpetstore-2.0.0-SNAPSHOT.jar
```

3. 상단에 있는 Preview - "Preview running application" 내용을 (아래 그림 참조) 클릭하면, JPetStore 화면이 표시됩니다. 

![image](https://user-images.githubusercontent.com/9047122/85239960-30b9ab80-b471-11ea-9861-c65733b79b24.png)

** 화면을 키우기 위하여, Preview 화면 오른쪽 상단의 버튼을 클릭하면, 전체 화면으로 전환됩니다. 


4. 먼저 Sample 프로그램의 기능을 확인해 보기 위하여, Enter the Store를 클릭합니다. 

![image](https://user-images.githubusercontent.com/9047122/85240016-82facc80-b471-11ea-94d6-e6fedde009bf.png)

5. 상단에 있는 "Sign in"을 클릭하고, Login 화면에 표시되는, "Register Now" 링크를 클릭하세요. 

![image](https://user-images.githubusercontent.com/9047122/85240059-af164d80-b471-11ea-9693-506e3b53f6c5.png)

6. 모든 항목을 입력하고, 하단에 있는 "Enable MyList", "Enable MyBanner"를 체크합니다. 
  (Phone 항목은 000-0000-0000 형태로 입력해 주셔야 합니다. )

![image](https://user-images.githubusercontent.com/9047122/85240087-d10fd000-b471-11ea-92c2-619a0dc735d3.png)

![image](https://user-images.githubusercontent.com/9047122/85240134-f7357000-b471-11ea-8024-86f087c4571a.png)

7. 다시 로그인 화면으로 이동하여 위에서 생성한 사용자 정보를 이용하여 로그인을 수행합니다.

8. 이후 몇가지 애완동물을 선택하여 카드에 담아 보시기 바랍니다. 

![image](https://user-images.githubusercontent.com/9047122/85240276-935f7700-b472-11ea-978f-7a20b23bae6e.png)

9. 카트에서 Proceed to checkout을 선택하여 구매 화면으로 이동합니다. 

10. 구매화면에서 Fake Card Number 를 이용하여, Card Number, Expiry Date 를 입력합니다. 

![image](https://user-images.githubusercontent.com/9047122/85240386-f4874a80-b472-11ea-8885-fd8111fb514e.png)

11. "Continue"를 선택하여, 구매확정 화면으로 이동하고, "Submit"을 클릭하여, 구매합니다. 

![image](https://user-images.githubusercontent.com/9047122/85240438-27314300-b473-11ea-9d82-3471d2494d9a.png)

12. 수고하셨습니다. 해당 내용을 통하여, 기본적인 JPetStore와 해당 기능에 대하여 알아보았습니다. 구성한 아키텍처는 단일 EC2 Instance에 Spring Boot와 H2 Standalone Database를 이용하여 구성한 경우며, 아래와 같은 구성으로 생각하면 됩니다. 

![image](https://user-images.githubusercontent.com/9047122/85339790-d1ff3b00-b51f-11ea-9161-28a49d5a6d4b.png)



# DynamoDB Session / Redis Session 의 차이점 확인해 보기

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


## Redis를 이용하는 Session Manager 설정하기

위와 같이 Session Cluster (Http Session)가 필요한 경우에 자주 사용되는 서비스가 ElastiCache Redis입니다. Redis의 짧은 latency와 throughput이 Session manager에 매우 적합하기 때문입니다. 

Spring에서는 Redis session manager가 기본적으로 제공되기 때문에, Redis Cluster를 구성하고, 이후, Configuration 수정을 통하여, 적용해 보도록 하겠습니다. 

1. Redis Cluster 생성을 수행하여야 합니다. Console 상에서 수행을 하여도 되며, 아래와 같이 CLI를 수행하여 구성할 수 있습니다. 
 Cloud 9 Terminal에서 아래 내용을 수행하십시요. 
 
```
DEFAULT_VPC=`aws ec2 describe-vpcs | jq ' .Vpcs[] | select(.IsDefault == true) | .VpcId' | sed '1,$s/"//g'`
SUBNETS=`aws ec2 describe-subnets --filters Name=vpc-id,Values=${DEFAULT_VPC} | jq ' .Subnets[] | .SubnetId' | sed '1,$s/"//g' | xargs`
aws elasticache create-cache-subnet-group \
--cache-subnet-group-name redis-subnetgroup \
--cache-subnet-group-description "Redis Test Cluster Subnets" \
--subnet-ids ${SUBNETS}

EB_ENVNAME=`aws elasticbeanstalk describe-environments | jq ' .Environments[] | .EnvironmentName' | sed '1,$s/"//g'`
EB_INSTANCE=`aws elasticbeanstalk describe-environment-resources --environment-name ${EB_ENVNAME} | jq ' .EnvironmentResources.Instances[0] | .Id' | sed '1,$s/"//g'`
EB_SG=`aws ec2 describe-instances --instance-id ${EB_INSTANCE} | jq ' .Reservations[0].Instances[0].SecurityGroups[0].GroupId ' | sed '1,$s/"//g'`

aws elasticache create-cache-cluster \
--cache-cluster-id redis-session-manager-cluster \
--az-mode single-az \
--num-cache-nodes 1 \
--cache-node-type cache.t3.small \
--engine redis \
--engine-version 5.0.6 \
--cache-subnet-group-name redis-subnetgroup \
--security-group-ids ${EB_SG} \
--port 6379
```

2. Redis Cluster를 생성할 때, ElasticBeanstalk에서 생성한 Security Group을 이용하여 Security Group을 설정하였습니다. EB Instances들이 Redis에 접근할 수 있도록, Ingress Rule을 추가합니다. (Cloud 9 Terminal에서 수행하십시요. ) 향후, Cloud 9 Terminal에서 Redis CLI를 이용하여 자료 확인을 하기 위해서 추가적인 Ingress Rule을 추가합니다. 

```
aws ec2 authorize-security-group-ingress \
--group-id ${EB_SG} \
--protocol tcp \
--port 6379 \
--source-group ${EB_SG}

DEFAULT_VPC_CIDR=`aws ec2 describe-vpcs | jq ' .Vpcs[] | select(.IsDefault == true) | .CidrBlock' | sed '1,$s/"//g'`

aws ec2 authorize-security-group-ingress \
--group-id ${EB_SG} \
--protocol tcp \
--port 6379 \
--cidr ${DEFAULT_VPC_CIDR}


```

3. 이것으로 인프라에서 Redis Cluster를 생성하고, Security Group을 설정하여, Elastic Beanstalk에 있는 Instance가 Redis에 접근할 수 있게 구성을 완료하였습니다. 이후에는 Spring Framework에서 Redis session manager를 설정해야 합니다. 

4. pom 파일에서 Spring redis session package를 추가합니다. Cloud 9 에서 project root에 있는 pom.xml 파일을 엽니다. 

![image](https://user-images.githubusercontent.com/9047122/85365998-6b027600-b561-11ea-816d-02f1b24c9e4b.png)

5. Dependencies 안에, 아래 내용을 추가합니다. 

```
  <dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session-data-redis</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
  
```

![image](https://user-images.githubusercontent.com/9047122/85366344-20352e00-b562-11ea-834a-1e73ebbe5230.png)

6. Redis Cluster 정보를 추가합니다. 

```
REDIS_ENDPOINT=`aws elasticache describe-cache-clusters --cache-cluster-id redis-session-manager-cluster --show-cache-node-info | jq ' .CacheClusters[0].CacheNodes[0].Endpoint.Address' | sed '1,$s/"//g'`

echo "spring.redis.host=${REDIS_ENDPOINT}" >> src/main/resources/application.properties
# echo spring.redis.password= # Login password of the redis server.
echo "spring.redis.port=6379" >> src/main/resources/application.properties

```

7. src/main/resources/application.properties 파일에 위에서 설정한 host 정보가 정상적으로 들어가 있는지 확인합니다. 

8. Redis CLI를 설치하고, redis cluster 에 붙어서, 현재 KEY 목록을 가지고 옵니다. (Cloud 9 Terminal에서 수행)

```
$ npm install -g redis-cli
$ rdcli -h ${REDIS_ENDPOINT}
redis-session-manager-cluster.xxxxx.0001.apn2.cache.amazonaws.com:6379> KEYS *

```

** 만약 연결이 되지 않으면, Security Group Setting이 제대로 되지 않은 것입니다. Redis Cluster console화면에서 Security Group를 확인해 보십시요. 연결이 되어 있지 않다면, Elastic BeanStalk Instance가 사용하는 Default SG값을 Redis Cluster에 적용하면 됩니다. 

현재는 어떤 세션 정보도 가지고 있지 않기 때문에, 아무 값이 나오지 않는 것을 볼 수 있습니다. exit명령어로 나오십시요. 

```
redis-session-manager-cluster.xxxxx.0001.apn2.cache.amazonaws.com:6379> exit
```

10 ElastiCache Redis는 Redis on EC2와는 다른 방식을 취합니다. 예를 들어 CONFIG와 같이 보안적으로 사용을 지양하는 Command는 ElastiCache Redis에서 사용할 수 없습니다. 문제는 Spring Redis Session Manager 는 초기 Connection 구성시에 CONFIG 명령어를 사용하도록 되어 있습니다. 이를 Disable하기 위해서는 Class 방식의 환경 설정이 필요합니다. 이를 위하여, 아래 내용을 Cloud 9 Terminal에서 수행해 주십시요. 

```
cd ~/environment/mybatis-spring-boot-jpetstore
echo "package com.kazuki43zoo.jpetstore.config;" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "import org.springframework.context.annotation.Configuration;" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "import org.springframework.session.data.redis.config.ConfigureRedisAction;" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "import org.springframework.context.annotation.Bean;" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "@Configuration" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "public class RedisConfig {" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "	@Bean" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "	ConfigureRedisAction configureRedisAction() {" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "		return ConfigureRedisAction.NO_OP;" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "	}" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java
echo "}" >> src/main/java/com/kazuki43zoo/jpetstore/config/RedisConfig.java

```

11. 반영된 패키지를 만들기 위하여, maven으로 재구성하고, eb를 통하여 update 합니다. 

```
./mvnw clean 
./mvnw package -Dmaven.test.skip=true
eb deploy

```

12. 패키지가 배포되면, 앞에서 복사한 Elastic Beanstalk CNAME URI를 이용하여 다시 Pestore에 접근 해봅니다. 

13. 로그인을 수행합니다. 

14. Cloud 9 Terminal 에서 다음과 같은 명령어를 이용하여, Redis에 접속합니다. 이후 KEYS * 명령어를 이용하여 Session 정보가 들어가 있는 것을 확인합니다. 

```
$ rdcli -h ${REDIS_ENDPOINT}
redis-session-manager-cluster.xxxxx.0001.apn2.cache.amazonaws.com:6379> KEYS *
redis-session-manager-cluster.xxxxx.0001.apn2.cache.amazonaws.com:6379> KEYS *
1) spring:session:sessions:32e02a17-d30e-494a-8ddd-8646a5f7ec30
2) spring:session:index:org.springframework.session.FindByIndexNameSessionRepository.PRINCIPAL_NAME_INDEX_NAME:honggildong
3) spring:session:sessions:9e6a3a5e-c638-4ee3-a603-0b19c1f05ac5

```

15. 여기까지 수행하였으면, 아래 아키텍처를 완료한 상태가 됩니다. 

![image](https://user-images.githubusercontent.com/9047122/85513558-18df5480-b636-11ea-8878-83e56e4a3787.png)


## Aurora MySQL 이용하여, 공통 Database 생성하고, 연결하기

현재는 개별 JPetStore Application에 있는 Local Storage (Hyper SQL)을 이용하여 자료를 저장하고 있습니다. 이럴 경우, 데이터가 서로 다르기 때문에
문제가 발생할 수 밖에 없습니다. 이를 해결하기 위하여, Aurora MySQL을 별도의 외부 Resource로 등록하고, JPetStore Application을 연결해 보도록 하겠습니다. 

1. RDS Aurora MySQL 에서 사용할 user / password 에서 사용할 정보를 Cloud 9 에 아래와 같이 저장합니다. !!! 주의 - <what as you want> 부분을 반드시 수정하십시요.

```
DB_USER=<what as you want>
DB_PASS=<what as you want>
```

1. RDS MySQL 생성을 하기 위하여, 아래 Command를 Cloud 9 Terminal에서 입력하시기 바랍니다. 
해당 내용은, JPetStore Instance에서 RDS에 접근할 수 있게 Security Group을 설정하고, Aurora Cluster 생성 이후, Aurora instance를 생성합니다. 

```
EB_ENVNAME=`aws elasticbeanstalk describe-environments | jq ' .Environments[] | .EnvironmentName' | sed '1,$s/"//g'`
EB_INSTANCE=`aws elasticbeanstalk describe-environment-resources --environment-name ${EB_ENVNAME} | jq ' .EnvironmentResources.Instances[0] | .Id' | sed '1,$s/"//g'`
EB_SG=`aws ec2 describe-instances --instance-id ${EB_INSTANCE} | jq ' .Reservations[0].Instances[0].SecurityGroups[0].GroupId ' | sed '1,$s/"//g'`
DEFAULT_VPC_CIDR=`aws ec2 describe-vpcs | jq ' .Vpcs[] | select(.IsDefault == true) | .CidrBlock' | sed '1,$s/"//g'`
DB_PORT=3306
DB_ENGINE=aurora-mysql
SUBNET_AZ=`aws ec2 describe-subnets --filters Name=vpc-id,Values=${DEFAULT_VPC} | jq ' .Subnets[] | .AvailabilityZone' | sed '1,$s/"//g' | xargs`

aws ec2 authorize-security-group-ingress \
--group-id ${EB_SG} \
--protocol tcp \
--port ${DB_PORT} \
--source-group ${EB_SG}

aws ec2 authorize-security-group-ingress \
--group-id ${EB_SG} \
--protocol tcp \
--port ${DB_PORT} \
--cidr ${DEFAULT_VPC_CIDR}

aws rds create-db-subnet-group \
          --db-subnet-group-name jpetstore-db-subnet \
          --db-subnet-group-description 'jpetstore database subnet. example' \
          --subnet-ids ${SUBNETS}

aws rds create-db-cluster \
          --availability-zones ${SUBNET_AZ} \
          --database-name dev \
          --db-cluster-identifier jpetstoredb \
          --vpc-security-group-ids ${EB_SG} \
          --db-subnet-group-name jpetstore-db-subnet \
          --engine ${DB_ENGINE} \
          --master-username ${DB_USER} \
          --master-user-password ${DB_PASS} \
          --no-enable-iam-database-authentication

aws rds create-db-instance \
--db-instance-identifier jpetinstance \
--db-instance-class db.t3.medium \
--engine ${DB_ENGINE} \
--db-subnet-group-name jpetstore-db-subnet \
--no-auto-minor-version-upgrade \
--db-cluster-identifier jpetstoredb

```

2. 일단 생성될 때까지 대기를 하여야 합니다. 이를 위해서 wait 명령어를 이용해 봅니다. 

```
aws rds wait db-instance-available --filters Name=db-cluster-id,Values=jpetstoredb
```

3. 생성이 완료되었기 때문에, Database Endpoint를 확인하고 해당 값을, JPetStore에 설정합니다. 

```
DB_ENDPOINT=`aws rds describe-db-clusters | jq ' .DBClusters[] | select(.DBClusterIdentifier == "jpetstoredb").Endpoint' | sed '1,$s/"//g'`
cd ~/environment/mybatis-spring-boot-jpetstore
cat src/main/resources/application.properties | sed "s/jdbc:hsqldb:file:~\/db\/jpetstore/jdbc:mysql:\/\/${DB_ENDPOINT}:${DB_PORT}\/dev/g" > src/main/resources/application.properties.new
rm src/main/resources/application.properties
mv src/main/resources/application.properties.new src/main/resources/application.properties
echo "spring.datasource.username=${DB_USER}" >> src/main/resources/application.properties
echo "spring.datasource.password=${DB_PASS}" >> src/main/resources/application.properties
echo "spring.datasource.driver-class-name=org.mariadb.jdbc.Driver" >> src/main/resources/application.properties

```

4. JDBC를 패키징해야 하기 때문에, pom.xml 파일에 maria jdbc library를 추가합니다. project root 디렉토리에 있는 pom.xml 파일을 엽니다. 이후 아래 내용을 추가합니다. 

```
<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
    <version>2.5.4</version>
</dependency>

```

4. 새로 프로그램을 Packaging 합니다. 

```
./mvnw clean 
./mvnw package -Dmaven.test.skip=true

```
5. 프로그램을 Elastic BeanStalk을 이용하여, 서버에 배포합니다. 

```
eb deploy
```

6. 다시 JPetStore Application 에 CNAME을 이용하여 접근합니다. 이후 로그인 및 기능 테스트를 수행하십시요. 

7. 여기까지 수행하셨다면 다음과 같은 아키텍처가 완성된 상태입니다. 

![image](https://user-images.githubusercontent.com/9047122/85593137-ba3fc800-b681-11ea-804b-e39b3d85eefc.png)

