---
title: "[aws] EC2 인스턴스에 SSM(Session Manager) 설정하기"
date: 2025-07-09 13:22:00 +0900
categories: ["aws", "ec2"]
tags: ["aws", "ec2", "session-manager", "ssm", "iam", "보안"]
pin: false
math: false
mermaid: false
---

# EC2 인스턴스에 SSM(Session Manager)으로 접속하기

AWS Systems Manager의 Session Manager는 SSH 없이 인스턴스에 접속할 수 있는 방법이다.
키 페어 없이도, 보안그룹 포트 오픈 없이도, IAM 권한만으로 웹 콘솔이나 CLI에서 접속이 가능하다.

이 글에서는 EC2 인스턴스에서 Session Manager를 사용하기 위한 IAM 설정, SSM Agent 설치, 권한 부여, 접속 방법까지 정리한다.

---

## 사전 요구 조건 체크리스트

| 항목                            | 필요 여부 | 설명                                             |
| ------------------------------- | --------- | ------------------------------------------------ |
| EC2 인스턴스 IAM 역할           | 필요      | `AmazonSSMManagedInstanceCore` 연결 필요         |
| 인스턴스 내 SSM Agent 설치 여부 | 필요      | 기본적으로 Amazon Linux / Ubuntu는 포함되어 있음 |
| 인바운드 포트                   | 불필요    | SSH(22) 열 필요 없음                             |
| IAM 사용자 권한                 | 필요      | `AmazonSSMFullAccess` 또는 최소 권한 필요        |

---

# 1. EC2 인스턴스에 IAM 역할 부여

### 역할에 부여할 정책: `AmazonSSMManagedInstanceCore`

1. IAM → 역할 → 새 역할 생성
2. 신뢰할 수 있는 엔터티 유형: EC2
3. 정책: `AmazonSSMManagedInstanceCore` 선택
4. 역할 이름 예시: `EC2_SSM_ROLE`
5. EC2 인스턴스 → 작업 > 보안 > IAM 역할 수정
6. 위 역할(`EC2_SSM_ROLE`) 연결

---

# 2. EC2 인스턴스에서 SSM Agent 확인 또는 설치

### Ubuntu 기준, 설치 스크립트

```bash
# snap 버전 확인
snap services amazon-ssm-agent

# 설치되어 있지 않다면 설치
sudo snap install amazon-ssm-agent --classic
sudo snap start amazon-ssm-agent
```

### 수동 deb 패키지 설치 예시

```bash
curl "https://s3.ap-northeast-2.amazonaws.com/amazon-ssm-ap-northeast-2/latest/debian_amd64/amazon-ssm-agent.deb" -o "ssm-agent.deb"
sudo dpkg -i ssm-agent.deb
```

> snap으로 이미 설치된 경우 `.deb` 설치 시 충돌할 수 있으므로 snap 버전 유지 권장
> {: .prompt-info }

---

# 3. IAM 사용자에게 Session Manager 권한 부여

웹 콘솔 또는 CLI 사용자에게도 SSM에 접근할 권한이 필요하다.

### IAM 정책: `AmazonSSMFullAccess` (관리형)

또는 아래와 같은 최소 권한 커스텀 정책을 생성할 수 있다.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:StartSession",
        "ssm:DescribeSessions",
        "ssm:TerminateSession",
        "ssm:DescribeInstanceInformation",
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

> 이 정책을 사용자 또는 사용자 그룹에 연결
> {: .prompt-info }

---

# 4. Session Manager로 접속하기

## AWS 웹 콘솔에서 접속

1. Systems Manager → 세션 관리자
2. 세션 시작 클릭
3. 연결할 EC2 인스턴스 선택

## AWS CLI에서 접속

```bash
aws ssm start-session --target i-0123456789abcdef0
```

> CLI를 사용하려면 AWS CLI가 설치되어 있어야 하며, 인증 정보가 설정되어 있어야 한다.
> {: .prompt-info }

---

# 5. IAM 사용자 생성 및 콘솔 로그인 설정

웹 콘솔로 접속할 수 있는 사용자를 생성하고 비밀번호 로그인 설정을 진행한다.

### 사용자 생성 순서

1. IAM → 사용자 → 사용자 추가
2. 액세스 유형: 프로그래밍 액세스 + AWS 관리 콘솔 액세스
3. 비밀번호 직접 입력 또는 자동 생성
4. 사용자 그룹 연결 또는 `AmazonSSMFullAccess` 직접 연결

### 로그인 링크 확인

```
https://<account-id>.signin.aws.amazon.com/console
예: https://224944859378.signin.aws.amazon.com/console
```

---

# 참고

| 항목                    | 설명                                       |
| ----------------------- | ------------------------------------------ |
| SSM Agent 로그 경로     | `/var/log/amazon/ssm/amazon-ssm-agent.log` |
| 시스템 서비스 상태 확인 | `snap services amazon-ssm-agent`           |
| 강제 재시작 명령        | `sudo snap restart amazon-ssm-agent`       |

---

> 포트를 개방하지 않아도 안전하게 EC2에 접속할 수 있는 Session Manager는 보안 및 운영 효율성 측면에서 매우 유용하다.
> {: .prompt-info }
