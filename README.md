# ec2-running-reporter
이 서비스는 한국 시간으로 매일 오전 7시에(UTC 기준 22:00) 설치 시 입력한 이메일 주소로 해당 AWS 계정에서 실행(running) 중인 모든 리전(Region)의 EC2 인스턴스들을 리스트로 만들어서 메일로 보내줍니다.

## 설치 방법
- ec2-running-reporter.yaml을 AWS CloudFormation 서비스에서 설치
- 설치 시 입력한 이메일로 발송된 SNS 구독 확인 메일에서 "Confirm subscription" 클릭

## 설치 세부내역
- AWS 콘솔에서 CloudFormation 서비스 > Create stack (standard) 선택 후, Choose an existing template 선택, Upload a template file 선택하고  "ec2-running-reporter.yaml"을 선택하고 Next 를 클릭해서 CloudFormation(이하 CF)으로 자동화된 서비스 구성을 진행합니다
- CF 생성 시 stack 이름엔 "ec2-running" (다른 이름도 가능) 그리고 리포트를 수신할 이메일 주소를 입력합니다. 이 CF template은 내부적으로,
  - 리포트를 이메일로 받을 수 있는 SNS Topic을 생성하고
  - 리포트 내용을 만드는 Lambda 함수(Python)를 생성하고
  - Lambda 함수가 사용할 IAM 역할(권한)을 생성하고
  - Lambda 함수를 매일 오전 7시에 실행시키는 CloudWatch Rule 을 생성합니다
- CF 수행이 완료되면 다음의 확인 작업을 수행해주세요
  - 입력한 이메일 주소로 SNS topic 가입확인 메일이 옵니다. 메일 중간의 "Confirm subscription"를 클릭해서 가입확인을 해줍니다

## 필요한 권한
- CloudFormation 스택 생성 권한
- Lambda, SNS, CloudWatch, IAM 리소스 생성 권한
- 모든 리전의 EC2 인스턴스 조회 권한 (Lambda 함수가 자동으로 획득)

## 참고사항
- 리포트를 수신할 이메일 주소를 변경하거나 다른 이메일 주소도 추가해야 할 땐, AWS 콘솔에서 SNS 서비스 > 왼쪽 메뉴에서 Topics 선택 후 > ec2-running.. topic을 선택 후 subscriptions 에서 기존 주소를 삭제하거나 다른 이메일 주소를 추가할 수 있습니다. (추가 시엔 Protocol 을 Email 선택 후 수신할 이메일 주소를 입력하면 됩니다)
- 리포트 받는 시간의 변경이 필요한 경우엔, AWS 콘솔에서 CloudWatch 서비스 > 왼쪽 메뉴에서 Event 아래의 Rules 선택 후 > ec2-running.. Rule 을 선택 후 Edit 에서 다른 시간으로 수정하시면 됩니다.
- 서비스 비용(Lambda 함수 비용)은 한달에 약 $1 이하로 발생합니다 (EC2 서버가 많아 Lambda 함수 수행 시간이 길어지면 좀 더 발생할 수 있습니다)

## 문제 해결
- 이메일이 오지 않는 경우: SNS 구독 확인 여부와 스팸 폴더를 확인해주세요
- Lambda 함수 오류 확인: CloudWatch Logs에서 해당 Lambda 함수의 로그를 확인할 수 있습니다

## 발송되는 이메일 예시
```
제목 : EC2 인스턴스 상태 보고 - 계정 590100000000

AWS 계정 N/A (590100000000)에서 실행 중인 EC2 인스턴스 목록 (2024-07-19 07:01:32 KST)

리전: ap-northeast-2
인스턴스 이름: test-server
인스턴스 ID: i-06fd..
인스턴스 타입: t3.xlarge
퍼블릭 IP: 1.2.3.4

리전: us-east-1
인스턴스 이름: test-emr
인스턴스 ID: i-0307...
인스턴스 타입: m5.large
퍼블릭 IP: 5.6.7.8

주의사항: 현재 실행(Running) 중인 상태의 EC2 인스턴스 리스트입니다. 중지(Stop)와 같은 다른 상태의 인스턴스들은 포함되어 있지 않습니다.
```

## 서비스 삭제 방법
- AWS 콘솔에서 CloudFormation 서비스 > 해당 스택 선택 후 Delete 클릭
- 모든 관련 리소스(Lambda, SNS, CloudWatch Rule, IAM Role)가 자동으로 삭제됩니다