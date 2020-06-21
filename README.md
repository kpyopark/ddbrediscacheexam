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


# Example Application IDE에 설치하기, 

해당 HoL에서는 PetClinic Example을 

# DynamoDB Session / Redis Session 의 차이점 확인해 보기

## DynamoDB를 이용하는 Session Manager 설정하기
1. 먼저 DynamoDB Session을 이용하는
