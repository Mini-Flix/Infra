# ☁️ Miniflix – 하이브리드 클라우드 기반 미디어 스트리밍 플랫폼

> 퍼블릭(AWS)과 프라이빗(OpenStack) 클라우드를 연계한 실무형 **하이브리드 클라우드 아키텍처 설계 및 구축 프로젝트**
![web](homepage.png)
<br>
<br>

## 📌 프로젝트 개요

- **프로젝트 명**: Miniflix – 하이브리드 클라우드 기반 Netflix 스타일 미디어 플랫폼
- **개발 기간**: 2025년 5월
- **팀 구성**: 총 4인 (인프라 + 서비스 통합 중심)
- **목표**: AWS + OpenStack 환경을 연결하여, 비용 최적화와 유연성을 동시에 확보하는 **하이브리드 클라우드 인프라 구축**

<br>
<br>

## 🧠 프로젝트 배경

- 1차 프로젝트에서는 온프레미스 환경에서 직접 인프라를 구성하며 실무 감각을 익혔습니다.
- 이후, 동일한 서비스 아키텍처를 **AWS 기반 클라우드로 재구성**하면서,
  - 온프레미스의 한계와 퍼블릭 클라우드의 장점을 비교
  - 비용 절감과 유연한 인프라 확장을 위해 **프라이빗 + 퍼블릭 환경을 연결한 하이브리드 구조**로 전환

<br>
<br>

## ☁️ 전체 아키텍처 개요
![img](miniflix2-동영상 파일 업로드 흐름도.png)
- **프론트엔드**: Vue.js 앱을 `S3 + CloudFront`에 정적 웹사이트로 배포
- **백엔드**: Spring Boot API 서버 → EC2에 배포 → Target Group 구성 → ALB → CloudFront에서 `/api/*`로 라우팅
- **스트리밍**: OpenStack VM에서 FFmpeg로 변환된 `.m3u8` 영상 → S3 업로드 → CloudFront 통해 스트리밍 제공
- **DB**: PostgreSQL 고가용성 구성 (HAProxy + Patroni)
- **로그 수집**: Kafka → Logstash → Elasticsearch → Kibana로 실시간 분석 파이프라인 구축

<br>
<br>

## 🛠 사용 기술

| 분야 | 기술 |
|------|------|
| 인프라 | AWS (S3, CloudFront, EC2, ALB, VPC), OpenStack |
| 웹 배포 | Vue.js, npm run deploy, GitHub Actions |
| 백엔드 | Spring Boot |
| DB | PostgreSQL, Patroni, HAProxy |
| 스트리밍 | FFmpeg, HLS (.m3u8, .ts), S3 |
| 로그 수집 | Kafka, Logstash, Elasticsearch, Kibana |
| 모니터링 | Kibana |
| 인증 | JWT 기반 인증, OAuth2.0 연동 예정 |

<br>
<br>

## 🔧 담당 업무 상세

### 박수빈

### ✅ 프론트엔드 자동 배포
- `npm run deploy` 명령으로 빌드 결과물을 S3에 업로드
- CloudFront 캐시 무효화 스크립트로 즉시 반영되도록 처리

### ✅ 백엔드 연동 구성
- EC2 + Spring Boot + ALB → Target Group 구성
- ALB를 CloudFront 오리진으로 등록해 `/api/*` 요청 처리

### ✅ 스트리밍 업로드 자동화
- OpenStack 트랜스코딩 서버에서 FFmpeg로 `.mp4 → .m3u8` 자동 변환
- 결과 파일을 S3의 스트리밍 버킷에 업로드하는 **자동화 스크립트** 작성
- CloudFront를 통해 `.m3u8` 영상 스트리밍 제공


<br>
<br>

## 💡 프로젝트를 통해 얻은 것

- **온프레미스와 클라우드 환경 모두에서 구축 경험**을 통해 아키텍처 설계 및 비교 분석 능력 향상
- **퍼블릭 클라우드의 장점(확장성, 유연성)**과 **단점(비용)**을 모두 체감
- **프라이빗 자원을 활용한 트랜스코딩 처리**를 통해 비용 절감 모델 수립
- **하이브리드 클라우드 환경의 가능성과 현실성**에 대해 실무적인 인사이트 확보
- 로그 수집/분석 파이프라인 구성까지 포함하여 **운영/관찰 가능한 인프라 설계 실현**

<br>
<br>

## 🎥 데모 영상 및 자료

- [📽️ 시연 영상 (YouTube)](https://www.youtube.com/watch?v=_oIZswled7s)
- [🖥️ 발표 자료 (Canva)](https://sulgasaeng.my.canva.site/miniflix)
- [📂 GitHub Repository](https://github.com/subin4420/project_infra/tree/main)

<br>
<br>

## ✅ 회고 및 개선점

- 온프레미스 환경에서 부족했던 확장성과 유연성을 클라우드를 통해 보완
- 비용 최적화와 데이터 통제의 현실적인 방법으로 **하이브리드 클라우드 전략** 체감
- 추후 Kubernetes 기반으로 서비스 재배포 예정 → 운영 자동화 및 자원 효율화 기대
