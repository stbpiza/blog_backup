# AWS CloudFormation 

![](https://images.velog.io/images/stbpiza/post/8759d5b0-cd6a-4de8-8951-165e4f16c69e/amazon_cloudformation_logo_icon_168665.png)


# CloudFormation이란?
AWS 리소스를 생성하기 위한 각종 설정을 템플릿 파일로 만들어서 사용하는 도구이다.
템플릿 파일로 인프라를 모델링할 수 있다.
인프라를 코드로 관리하는걸 IaC(Infrastructure as Code)라 부른다.

코드로 관리하기 때문에 유지보수면에서 유리하다.
서로 의존관계인 리소스들을 하나로 묶어서 통합 관리가 가능하다.
생성된 리소스 요금을 제외하면 추가적인 사용요금은 없다.

# 구성요소
## 1. Template
설정과 설명을 작성한 템플릿 파일
JSON 또는 YAML 형식으로 작성
CloudFormation Designer에서 GUI로 템플릿 생성 가능
## 2. Stack
CoudFormation으로 생성한 AWS 리소스 집합
Stack 단위로 생성 수정 삭제 가능
Stack을 삭제하면 관련 리소스가 모두 삭제
리소스 간에 의존관계가 있는 경우 순서에 맞게 생성됨

# Template 작성 항목
```json
{
  "AWSTemplateFormatVersion" : " version data ",
  "Description" : " JSON string ",
  "Metadata" : {
       템플릿에 대한 추가 정보
  },
  "Parameters" : {
       (스택을 생성하거나 업데이트할 때) 실행 시간에 템플릿에 전달하는 값
  }, 
  "Mappings" : {
       템플릿 실행 시 선택하게 되는 값(특정 리전, 인스턴스)
  },
  "Conditions" : { 
       특정 리소스가 생성되는지 여부를 제어하는 조건
  }, 
  "Transform" : {
       서버리스 애플리케이션에서 사용
  },
  "Resources" : {
       생성될 스택 리소스 및 해당 속성 (필수)
  }, 
  "Outputs" : {
       생성된 리소스 결과값 조회(자원 ID, IP 등)
  }
}
```

## 1. Parameters
- 리소스 생성시 넘겨줄 파라미터 생성
- 여러 인풋값을 만들어 두고 상황에 맞게 값을 바꾸어 넣으려는 목적
- ex) 테스트용 실행인지 실제 배포인지 구분하는 값 넣기, 이미지 종류나 인스턴스 타입 고르기
- 파라미터 이름으로 다른 항목에서 가져다 쓸 수 있음

```yaml
Parameters:

  # 파라미터 이름(ec2 암호화 키페어)
  KeyName:
    # 파라미터 설명
    Description: Name of an existing EC2 KeyPair to enable SSH access to the web server
    # 파라미터 타입
    Type: AWS::EC2::KeyPair::KeyName
    # 오류메시지
    ConstraintDescription: must be the name of an existing EC2 KeyPair.

  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    # 디폴트 값
    Default: t2.micro
    # 선택 가능한 값들 지정
    AllowedValues:
    - t2.micro
    - t2.small
    ConstraintDescription: must be a valid EC2 instance type.

  SSHLocation:
    Description: Lockdown SSH access to the bastion host (default can be accessed
      from anywhere)
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    # 정규표현식도 사용 가능
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: must be a valid CIDR range of the form x.x.x.x/x.
```

## 2. Mappings
- mappings에 저장해둔 정보는 resources에서 꺼내서 사용
- parameters에서 입력한 값으로 mappings에서 데이터 꺼내가는 형식
- 특정 이미지, 인스턴스 타입을 상황별로 저장해두고, 필요할 때 마다 원하는 정보를 골라서 사용 가능
- ex) 테스트 상황, 실제 배포 상황에 맞는 설정 골라가기
- 매핑 이름 → 파라미터값과 비교할 이름 → 꺼낼 값 이름 3단계로 작성
```yaml
Parameters:
  MyImageId:
    Type: String
    Default: ap-northeast-1
    AllowedValues:
      - ap-northeast-1
      - ap-northeast-2
      - ap-northeast-3


Mappings:

  # 매핑 이름
  AWSRegionArch2AMI:
		# 파라미터값과 비교할 값
    ap-northeast-1:
			# 꺼낼 값 이름
      HVM64: ami-0b2c2a754d5b4da22
      HVMG2: ami-09d0e0e099ecabba2
    ap-northeast-2:
      HVM64: ami-0493ab99920f410fc
      HVMG2: NOT_SUPPORTED
    ap-northeast-3:
      HVM64: ami-01344f6f63a4decc1
      HVMG2: NOT_SUPPORTED


Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
			# !FindInMap으로 매핑 탐색 [매핑이름, !Ref '파라미터이름', 꺼낼값이름]
      ImageId: !FindInMap [AWSRegionArch2AMI, !Ref 'MyImageId', HVM64]
```

## 3. Conditions
- 상황별로 특정 리소스를 생성할지 말지 설정 가능
```yaml
Parameters:
  # 파라미터 설정
  Env:
    Description: Enviroment type
    Default: test
    Type: String
    AllowedValues:
      - test
      - prod
    ConstraintDescription: test or prod

Conditions:

  # 컨디션 설정(Env 파라미터가 test 라면 true)
  CreateTest: !Equals [ !Ref Env, test ]

Resources:
	
  EC2Instance:
    Type: AWS::EC2::Instance
    # 컨디션이 true면 생성
    Condition: CreateTest
    Properties:
        ImageId: !Ref ImageId
        InstanceType: t2.micro
```

## 4. Resources

- 실제 만들어질 리소스에 대한 정보
- parameters나 mappings 값을 넣어서 제작 가능
- `Ref` - 다른 항목에서 값 참조
- `Fn::FindInMap` - mappings에서 값 꺼내오기
```yaml
Resources:

  # EC2 생성 선언
  EC2Instance:
    # 타입
    Type: AWS::EC2::Instance
    # 설정값
    Properties:
      # 인스턴스 타입(클라우드 사양)
      InstanceType:
        # 파라미터에 넣은 값 참조
        Ref: InstanceType
      SecurityGroups:
      - Ref: InstanceSecurityGroup
      KeyName:
        Ref: KeyName
      ImageId:
        Ref: ImageId

  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp:
          Ref: SSHLocation
```

## 5. Outputs
- 리소스가 생성된 후 조회할 수 있는 값
- 다른 항목들의 값을 참조하여 세팅 가능
```yaml
Outputs:

  InstanceId:
    Description: InstanceId of the newly created EC2 instance
    Value:
      Ref: EC2Instance

  AZ:
    Description: Availability Zone of the newly created EC2 instance
    Value:
      Fn::GetAtt:
      - EC2Instance
      - AvailabilityZone

  PublicDNS:
    Description: Public DNSName of the newly created EC2 instance
    Value:
      Fn::GetAtt:
      - EC2Instance
      - PublicDnsName

  PublicIP:
    Description: Public IP address of the newly created EC2 instance
    Value:
      Fn::GetAtt:
      - EC2Instance
      - PublicIp
```