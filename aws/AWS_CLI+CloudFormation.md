# AWS CLI + CloudFormation

![](https://images.velog.io/images/stbpiza/post/dca0a4f2-6d90-4c17-915b-d1e8c37c5708/aws-cli-logo.svg)

# AWS CLI(명령줄 인터페이스)
터미널에서 aws 서비스를 관리할 수 있게 해주는 도구

콘솔에서 클릭으로 서비스를 관리할 수 있지만,
터미널 명령어를 사용하면 일관성있게 관리하거나 자동화가 가능해짐

# 설치
파이썬을 사용하므로 파이썬이 설치되어 있어야 함

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

맥의 경우 brew로 설치 가능
```bash
$ brew install awscli
```

# 환경세팅

IAM 계정을 만들고 키 발급해와야 사용가능

aws configure입력하고 액세스키, 시크릿키 입력,
지역 입력(ap-northeast-2 = 서울), 출력형식입력


```bash
aws configure

AWS Access Key ID [None]: {액세스키}
AWS Secret Access Key [None]: {시크릿키}
Default region name [None]: ap-northeast-2
Default output format [None]: yaml
```

# CloudFormation 사용하기
--stack-name {스택이름(중복되지않게 설정)}
--template-body {템플릿파일경로(file:// 로 시작해야함)}
--parameters ParameterKey={키이름},ParameterValue={키값}

```bash
aws cloudformation create-stack --stack-name {myteststack} 
--template-body {file://sampletemplate.json} 
--parameters ParameterKey={KeyPairName},ParameterValue={TestKey}
ParameterKey={SubnetIDs},ParameterValue={SubnetID1\\,SubnetID2}
```

CloudFormation 템플릿 작성한 대로 스택 생성됨

생성된 스택 상태 조회하기
```bash
aws cloudformation describe-stacks --stack-name {myteststack}
```
