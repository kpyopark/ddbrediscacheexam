# Aurora MySQL 이용하여, 공통 Database 생성하고, 연결하기

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
