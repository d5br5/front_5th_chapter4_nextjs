# 프론트엔드 배포 파이프라인 구성

## 주요 링크

- S3 bucket website endpoint: http://hhplus-nextjs-dohkim.s3-website-ap-southeast-2.amazonaws.com
- CloudFront 배포 도메인: https://dj6c06iswjpqc.cloudfront.net

## 주요 개념

### CI/CD

#### 용어 정리

- CI
  - Continuous Integration, 지속적 통합
  - 여러 개발자들이 작성한 코드를 주요 브랜치(main, master 등)에 병합하는데, 그때마다 자동으로 코드를 테스트하고 빌드하는 프로세스
  - 코드의 충돌을 방지하고, 테스트를 자동화해 코드의 품질을 일관되게 유지하는 것이 목적
- CD
  - Continuous Delivery
    - CI 이후, 배포 가능한 상태의 산출물을 자동으로 준비하는 프로세스
    - 실제 배포는 사람이 수동으로 '승인' 버튼을 눌러야 진행됨
  - Continuous Deployment
    - CI 테스트 통과 이후, 인간의 개입 없이 운영 환경까지 자동으로 배포되는 프로세스

#### 자동화 도구 및 플랫폼

- 클라우드 기반 통합 플랫폼

| 플랫폼                  | 설명                                   | 특징                                             |
| ----------------------- | -------------------------------------- | ------------------------------------------------ |
| **GitHub Actions**      | GitHub에 내장된 CI/CD 도구             | 코드 push, PR 등을 기준으로 워크플로우 자동 실행 |
| **GitLab CI/CD**        | GitLab에 내장된 풀스택 DevOps 플랫폼   | Git 저장소, 이슈, CI/CD를 하나의 도구에서        |
| **Bitbucket Pipelines** | Bitbucket용 내장 CI/CD 도구            | YAML로 파이프라인 구성, Jira와 연동 용이         |
| **CircleCI**            | 빠르고 유연한 CI/CD 서비스             | GitHub/GitLab 연동, 빠른 캐시 전략               |
| **Travis CI**           | GitHub와 함께 자주 쓰이던 CI/CD 서비스 | 오픈소스에 무료, 설정 간단 (지금은 사용률 하락)  |

- 자체 호스팅 가능한 CI/CD 서버

| 도구          | 설명                                   | 특징                                                    |
| ------------- | -------------------------------------- | ------------------------------------------------------- |
| **Jenkins**   | 가장 널리 알려진 오픈소스 CI 도구      | 플러그인 기반으로 매우 유연하지만 설정이 복잡할 수 있음 |
| **TeamCity**  | JetBrains에서 만든 CI 도구             | 강력한 UI와 설정 기능 제공, 상용 라이선스               |
| **Drone CI**  | 경량의 컨테이너 기반 CI 시스템         | Docker/Kubernetes 친화적                                |
| **Buildkite** | 자체 서버에서 실행하는 하이브리드 방식 | 자체 호스팅 runner와 클라우드 UI 분리 구조              |

#### Github Actions의 특징 및 장점

- GitHub 저장소에 기본 내장되어 별도 설정 없이 바로 사용 가능
- `push`, `pull_request`, `issue`, `schedule(cron)` 등 다양한 GitHub 이벤트에 반응
- `.github/workflows/*.yml` 파일에 YAML 문법으로 작성하여 워크플로우 구성
- Job & Step 구조로 병렬 또는 순차적 실행을 세밀하게 제어할 수 있음
- 전 세계 개발자들이 만든 액션(step) 재사용할 수 있는 Marketplace 제공 (`checkout`, `setup-node`, `upload-artifact` 등)
- GitHub 서버 외에 자체 머신에서도 워크플로우 실행 가능
- 퍼블릭 저장소는 무료, 프라이빗 저장소는 일정량까지 무료 제공 (초과 시 과금)

| 항목          | GitHub Actions                        | Jenkins                        | GitLab CI                             |
| ------------- | ------------------------------------- | ------------------------------ | ------------------------------------- |
| 설치 필요     | ❌ (내장)                             | ✅ (서버 필요)                 | ❌ (내장)                             |
| 러너 관리     | GitHub-hosted / self-hosted 선택 가능 | 직접 관리                      | GitLab-hosted / self-hosted 선택 가능 |
| 진입 장벽     | 낮음 (간단한 설정)                    | 높음 (복잡한 설정 및 플러그인) | 중간 (GitLab에 익숙해야 함)           |
| Git 연동      | GitHub와 완전 통합                    | 수동 설정 필요                 | GitLab과 완전 통합                    |
| 커뮤니티 액션 | 매우 풍부                             | 플러그인 중심                  | GitLab 전용 CI 스크립트               |

### Storage

#### 스토리지??

- 데이터를 저장하고 관리하는 시스템
- 운영체제나 서버, 클라우드 서비스 등에서 사용자의 파일, 이미지, 로그, 백업 데이터 등을 저장하는데 사용

| 유형                  | 설명                              | 예시                      |
| --------------------- | --------------------------------- | ------------------------- |
| **블록 스토리지**     | 하드디스크처럼 작동, DB 등에 사용 | AWS EBS, iSCSI            |
| **파일 스토리지**     | 디렉토리 구조, NAS처럼 공유 사용  | NFS, EFS                  |
| **오브젝트 스토리지** | 파일 + 메타데이터를 객체로 저장   | AWS S3, GCP Cloud Storage |

#### S3

> AWS 에서 제공하는 Object Storage Service

- 기본 구성 요소
  - Bucket: S3 안에서 파일을 저장하는 공간 (디렉토리 느낌)
  - Object: 업로드한 실제 파일 (이미지, PDF, HTML 등)
  - Key: 오브젝트의 고유한 경로 (예: images/banner.png)
  - Metadata: 파일 크기, 타입, 사용자 정의 정보 등
- 주요 특징
  - 이미지, HTML, JS, CSS 파일 등을 안전하게 저장
  - IAM 권한, 버킷 정책, 암호화 지원
  - 백업 및 로그 저장소 장기 보관 및 무제한 확장 가능
  - CloudFront와 연동 시, 빠른 전 세계 파일 전송 가능
  - 비용 효율적 사용한 만큼 과금. 고정 용량 필요 없음
  - 퍼블릭 URL 접근 여부 세부 설정 가능. 제한된 시간만 다운로드 가능한 링크 생성 가능
  - 내구성 99.999999999%, 고장/손실 위험 매우 낮음

### CDN: Content Delivery Network

- CDN
  - 정적/동적 콘텐츠를 전 세계 각 지점에 분산저장해두고, 요청한 사용자와 지리적으로 가까운 곳에서 제공함으로써 응답 속도를 효과적으로 줄여주는 기술, 방식
- 특징

  - 사용자와 가까운 서버에서 전달
  - 서버 과부하 요청을 CDN 서버에서 처리해 원 서버 부담 감소
  - 대역폭 비용 증가 캐시된 콘텐츠 제공으로 원 서버 요청 최소화
  - DDoS 공격 등 보안 문제 글로벌 네트워크로 트래픽 분산 및 보호

- 동작 방식

  1. 사용자가 파일 요청
  2. CDN 엣지 서버에서 캐시된 콘텐츠 있으면 → 바로 응답
  3. 없으면 원 서버(origin)에서 가져와 → 엣지 서버에 캐시 → 사용자에 전달

- CloudFront?
  - AWS에서 제공하는 CDN 서비스
  - 특징
    - 정적/동적 콘텐츠 캐싱: HTML, JS, 이미지, API 응답 등 캐싱 가능
    - 보안 기능: SSL, WAF, OAI(Origin Access Identity), Signed URL 지원
    - 자동 압축 및 캐싱 제어: Brotli, Gzip 압축 지원, TTL 설정 가능
    - DDoS 보호

### 캐시 무효화 Cache Invalidation

- 정의
  - 캐시에 저장된 오래된 데이터를 무효화하고, 최신 데이터를 다시 받아오도록 만드는 과정
- 왜 필요한가?
  - 데이터나 코드가 업데이트 되었는데, 이전에 캐시된 내용이 계속 보여지는 것은 문제가 될 수 있음
- 방법
  - 파일명 hash versioning: 파일에 해시값 또는 버전 번호를 붙여서 새 파일처럼 취급
  - CDN 자체 갱신 API 사용
  - 캐시 제어 헤더 설정
    - Cache-Control: no-cache 항상 서버에 확인 후 캐시 사용
    - Cache-Control: no-store 아예 캐시에 저장하지 않음
    - Cache-Control: max-age=3600 1시간 동안 캐시 유지
    - ETag, Last-Modified 변경 여부에 따라 조건부 요청 처리

### Repository secret과 환경변수

- 환경변수

  - 특징
    - 코드 외부에서 주입하는 설정값
    - 애플리케이션이 동작하는 환경에 따라 유동적으로 값을 변경할 수 있게 함
  - 사용 목적
    - 민감한 정보 숨기기 (API 키, DB 비밀번호 등)
    - 환경별 설정 구분 (개발/운영)
    - 소스코드 재배포 없이 설정값 변경

- GitHub Repository Secret
  - GitHub Actions에서 사용하는 비공개 환경 변수
  - API 키, Access Token, 배포 비밀번호 같은 민감한 정보를 안전하게 저장하고 사용할 수 있도록 제공

## CDN과 성능최적화

CDN 도입 전과 도입 후의 성능 개선 보고서 작성
