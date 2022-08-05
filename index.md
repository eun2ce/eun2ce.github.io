# 조은희 (eun2ce)

**저는 `______` 하는 엔지니어입니다.**

1. 제품의 변경을 우선적으로 고민하는
2. 성장을 위한 개발 문화를 고민하는
3. 채용을 비롯하여 조직과 사람에 관심이 많은

**저는 `______` 하는 조직을 선호합니다.**

1. 출퇴근 시간을 본인이 결정할 수 있는
2. 의사결정이 투명하게 이뤄지고 이를 위한 정보가 수평적으로 공유되는
3. 구성원들이 조직의 비전을 함께 구체화하고 공감할 수 있는

|             |                                 |
| :---------: | ------------------------------- |
| **GitHub**  | <https://github.com/eun2ce>     |
|  **Blog**   | <https://eun2ce.tistory.com>    |
| **Contact** | <joeun2ce@gmail.com>            |

# Experiences

## Ahnlab

|              |                                                              |
| -----------: | ------------------------------------------------------------ |
|   **period** | 19.09 ~ current                                              |
| **position** | devOps / 서비스 개발                                            |
| **projects** | 제품의 AI 탐지기능 개발 및 고도화 (Ahnlab v3 365 clinic, 모바일 v3, ) |

#### Frontend Platform Engineering

- 원클릭 배포 및 롤백 환경을 위한 어드민 웹 애플리케이션 구성
- A/B Testing을 위한 TypoeScript SDK
- 사용자 지표 추적을 위한 TypeScript/React 로깅 SDK
- 이메일 템플릿 작업을 위한 어드민 작업 (AWS SES, Nest.js, Next.js)

#### 프론트엔드 제품의 배포 환경 구성 (20.10 ~ 22.02)

- AWS 리소스들을 Terraform으로 관리
- AWS의 S3, CloudFront, Lambda@edge를 사용한 정적 배포 구성
- GitHub Action을 기반으로 CI/CD 구성

#### 모델 학습을 위한 raw data 수집 MSA 애플리케이션 개발 (2020.03 - 2021.04)

##### contribute
- scala finagle 기반의 chrome extension으로부터 웹 데이터를 수집하여 kafka에 저장하는 서버 개발 
- react, GraphQL을 이용해 악성 탐지 결과를 모니터링 할 수 있도록 하는 시스템 기능, UI 개발 
- 스토리지로부터 데이터를 주기적으로 가져와 kafka에 dump 하는 k8s daemon 개발 
- 3-4) python 기반의 syslog daemon을 통해 사내 프록시 로그로부터 url을 수집하는 애플리케이션 개발 및 airflow 등록

##### tech
- scala finagle, twitterServer 를 기반으로 thrift 프로토콜을 사용하여 서버 개발 
- react, python, ubuntu 서버
- apache airflow를 활용한 애플리케이션 스케줄링 기법

##### achievements of results
- 수집된 데이터를 기반으로 ahnlab v3 365 clinic 제품의 ai 탐지기능 고도화

#### 데이터 수집, 분석에 필요한 MSA 애플리케이션 개발 (2020.11 - 2021.09)

##### contribute
- k8s 환경에서의 시스템 테스트 자동화 파이프라인 구축 및 유지보수
- 사내저장소와 연동된 CI툴（bamboo）기반의CI（빌드）프로세스를 구축
- CD툴（rancher）기반의CD（배포）프로세스를 통해 개발자에게 일관된 배포 환경 제공

##### tech
- 오픈소스 vmware python api를 이용하여 k8s 서비스 테스트 pip package 개발

##### achievements of results
- ahnlab 인공지능팀 내 k8s를 기반으로 한 서비스의 IT 테스트 가능

#### 쿠버네티스를 활용한 서비스 환경 셋업 자동화 (2020.09 - 2020.10)

##### contribute
- ansible 코드를 바탕으로 서버 구축 자동화
- 서버 주요 리소스 grafana, prometheus 기반의 모니터링 시스템 제공

##### tech
- ansible
- 서버 리소스 모니터링을 위한 prometheus, elasticsearch 쿼리문 작성

##### achievements of results
- 구동중인 서버 리소스 모니터링 및 alert를 통한 장애 대응 가능


#### MSA 서비스 docker container 모니터링 환경 구축 (2020.01 - 2020.03)
 
##### contribute
- Filebeat + ES(ElasticSearch) + Kibana 로그 수집 시스템을 구축하여 모니터링 환경 셋업

##### tech
- helm 차트 기반으로 k8s 에 elastic-stack 시스템 구축

##### achievements of results
- 실제 production 환경에서 구동중인 마이크로서비스 모니터링 가능

#### AWS EKS기반 정부과제 프로젝트 인프라 구축 및 외부 서비스용 rest api 서버 개발 (2019.10 - 2019.12)

##### contribute
- 쿠버네티스를 활용한 hyperledger eco시스템 구축
- 안정적인 외부 서비스를 위한 방화벽 로드 밸런싱, 쿠버네티스 버전 업그레이드 작업 진행
- aws 환경에 구축되어있는 폐쇄망 인프라와 외부 업체에 서비스 되어야 할 rest api 서버 개발

##### tech
- k8s, aws eks, aws lb 등
- go 언어를 이용한 rest api 서버 개발

##### achievements of results
- 구축된 eks 에 사내 eco system이 올라가 외부에 서비스가 가능해짐

# Communities

#### Open Source

- EOSIO eosio.contracts contributions ([PRs](https://github.com/EOSIO/eosio.contracts/pull/282))

#### Activity

- [interview](https://signal.sedaily.com/NewsView/1VGLKLN4T2/) (19.03)

# Education

### coursera

(21.07 - 21.11)  practical data science 수료