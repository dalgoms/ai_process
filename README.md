# AI Process Playbook

**Growth Marketing OS — Automate** 파트를 담당하는 자동화 허브입니다.
모바일에서 작업을 등록하면 AI가 코드를 작성하고, 리드를 수집하고, 콘텐츠를 생성합니다.

```
Analyze (WebScout) → Create (Ad Creative Tool) → ▶ Automate (ai_process) → Measure (UTM+GA)
```

---

## Service Diagnostics

서비스 구조를 직접 크롤링·분석하고, 성장 기회와 솔루션 방향을 정리합니다.
[WebScout](https://github.com/dalgoms/webscout-next)으로 사이트를 크롤링하고, 구조 분석 → 기회 도출 → 솔루션 설계까지 정리한 케이스들입니다.

| Case | 대상 | 분석 테마 | 상태 |
|------|------|----------|------|
| [나만의닥터 — AI 건강 비서](ai-geongangi-mvp/) | 헬스케어 · 비대면 진료 | SEO 유입 → 일상 참여 전환, CRM 설계, 경쟁 분석 | Done |
| *(다음 케이스)* | | | |

> 새 케이스 시작 시 [`case-template/`](case-template/)를 복사하여 사용합니다.
> 프로세스 가이드: [역방향 리크루팅 플레이북](docs/reverse-recruiting-playbook.md)

---

## Playbooks

산업별 AI 자동화 적용 전략.

| Playbook | 대상 | 내용 |
|----------|------|------|
| [Healthcare](playbooks/healthcare/README.md) | 비대면 진료, 디지털 헬스케어 | 성장 루프, 자동화 구조, 데이터 레이어 |
| SaaS (향후) | B2B SaaS | 온보딩 자동화, 이탈 방지 |
| Content Platform (향후) | 미디어, 교육 | 콘텐츠 자동 생산, 배포 |

## Business Cases

구체적인 적용 사례.

| 사례 | 설명 |
|------|------|
| [Healthcare Platform](docs/business-case-healthcare.md) | 헬스케어 B2B/B2C/내부 자동화 전체 설계 |

## Templates

새 고객사 온보딩 시 재사용 가능한 템플릿.

| 템플릿 | 용도 |
|--------|------|
| [Notion Leads DB Schema](templates/notion-leads-db-schema.md) | CRM DB 스키마 (업종별 커스텀 가이드 포함) |
| [Make.com Scenario Template](templates/make-scenario-template.md) | 자동화 시나리오 설계 양식 + 패턴 5가지 |

## Diagrams

Mermaid 기반 아키텍처 다이어그램.

| 다이어그램 | 내용 |
|-----------|------|
| [Healthcare Automation Architecture](diagrams/healthcare-automation-architecture.md) | 헬스케어 자동화 전체 구조도 |
| [Healthcare Growth Loop](diagrams/healthcare-growth-loop.md) | AARRR 기반 성장 루프 |
| [AI Process Architecture](diagrams/ai-process-architecture.md) | 범용 AI 프로세스 아키텍처 |

## Resume

| 문서 | 내용 |
|------|------|
| [Seyoung Lee (EN)](resume/resume-seyoung-lee.md) | AI Automation Architect |
| [이세영 (KO)](resume/resume-seyoung-lee-ko.md) | Growth Marketing & AI Automation |

---

## AI Automation Pipeline

### Pipeline

```
Notion 등록 → ai_process가 프로젝트 식별 → 해당 레포에 Issue 생성 → GPT Codex 코드 수정 → PR 생성 → CI 검증 → Telegram 알림 → 수동 승인 → 배포
```

### 멀티 프로젝트 지원

하나의 Notion Work Inbox에서 여러 프로젝트를 관리합니다. **Project** 필드를 기반으로 자동 라우팅됩니다.

| Project | Repository | 용도 |
|---------|-----------|------|
| `webscout-next` | [dalgoms/webscout-next](https://github.com/dalgoms/webscout-next) | Next.js 메인 서비스 |
| `ai_process` | [dalgoms/ai_process](https://github.com/dalgoms/ai_process) | 자동화 문서 관리 |

새 프로젝트를 추가하려면 `config/projects.json`에 항목을 추가하세요.

### Architecture

```
┌──────────┐
│  Notion   │ (Work Inbox - 모든 프로젝트 통합)
│  Inbox    │
└─────┬────┘
      │ 5분 cron
      ▼
┌──────────────────────────────────┐
│  ai_process / notion-sync.yml    │ (중앙 오케스트레이터)
│  config/projects.json 참조       │
└─────┬──────────────┬─────────────┘
      │              │
      ▼              ▼
┌──────────┐   ┌──────────┐
│ webscout │   │ ai_proc  │   ← 프로젝트별 레포
│ -next    │   │ ess      │
├──────────┤   ├──────────┤
│ codex    │   │ codex    │   ← 각 레포의 워크플로우
│ ci       │   │ notify   │
│ notify   │   └──────────┘
└────┬─────┘
     │ PR merge
     ▼
┌──────────┐   ┌──────────┐
│  Vercel  │   │ Telegram │
│  Deploy  │   │ 알림     │
└──────────┘   └──────────┘
```

---

## 비즈니스 자동화

코드 파이프라인 외에 마케팅/세일즈 자동화도 운영합니다.

| Notion DB | 용도 | Make.com 연동 |
|-----------|------|--------------|
| [Leads](https://www.notion.so/49f20ec826ed41ff9f5cc137af122ed3) | B2B+B2C 리드 CRM | 리드 캡처, 팔로업 리마인더 |
| [Content Pipeline](https://www.notion.so/69a39bc304664c658ec1b1fb7117c100) | 마케팅 콘텐츠 관리 | AI 초안 생성 |
| [Proposals](https://www.notion.so/3e86d8a31dfd4fc5829a6ed4482b3708) | B2B 제안서 관리 | (향후) |

웹훅 엔드포인트: `/api/webhook/contact`, `/api/webhook/inquiry`

---

## 사용법

### Notion (모바일/PC)

Work Inbox에 새 행 추가 → Project 필드 선택 → 5분 내 자동 실행

### Cursor에서 대화로

```
Notion에 등록해줘: [제목], [변경사항], [파일 경로]
```

### GitHub Issue 직접 생성

해당 프로젝트 레포에서 Issue를 만들 때 `ai-task` 라벨만 붙이면 Codex가 자동 실행

### 프롬프트 팁

```
# 좋은 예
app/analyze/page.tsx에서 "분석 시작" 버튼의 텍스트를 "Start Analysis"로 변경해주세요

# 나쁜 예
버튼 바꿔주세요
```

---

## Connected Services

| 서비스 | 역할 |
|--------|------|
| [Notion Work Inbox](https://www.notion.so/7f471e8dcba44e878b96cfae8d0de083) | 작업 등록 (모든 프로젝트 통합) |
| [webscout-next](https://github.com/dalgoms/webscout-next) | Next.js 서비스 |
| [ai_process](https://github.com/dalgoms/ai_process) | 중앙 오케스트레이터 + 플레이북 |
| [Vercel](https://webscout-next-8veo.vercel.app) | 배포 |
| Telegram (@seyounginboxbot) | 알림 |
| Make.com | 비즈니스 자동화 (리드, 콘텐츠, 팔로업) |

---

## 상세 문서

### 파이프라인 기술 문서

| 문서 | 내용 |
|------|------|
| [Architecture](docs/architecture.md) | 시스템 구성도, 데이터 흐름, 기술 스택, DB 스키마 |
| [Setup Guide](docs/setup-guide.md) | 처음부터 전체 파이프라인 구축하는 방법 |
| [Workflows](docs/workflows.md) | 각 GitHub Actions 워크플로우 상세 동작 |
| [Troubleshooting](docs/troubleshooting.md) | 자주 발생하는 문제 해결 + FAQ |
| [Changelog](docs/changelog.md) | 구축 이력, 해결한 버그, 참조 링크 |

### 비즈니스 자동화 문서

| 문서 | 내용 |
|------|------|
| [Make.com Scenarios](docs/make-scenarios.md) | Make.com 시나리오 설정 가이드 (리드, 팔로업, 콘텐츠) |
| [Notification Channels](docs/notification-channels.md) | 알림 채널 가이드 (Teams, Slack, 카카오, SMS, Gmail) |
| [Business Case: Healthcare](docs/business-case-healthcare.md) | 헬스케어 플랫폼 적용 사례 |

### 프로젝트 추가 방법

1. GitHub 레포 생성
2. `config/projects.json`에 항목 추가
3. PAT의 Repository access에 새 레포 추가
4. 새 레포에 `codex.yml`, `ci.yml`, `deploy-notify.yml` 복사
5. 새 레포에 Secrets 등록
6. `.codex-context` 파일 생성 (프로젝트 설명)
7. Notion Work Inbox의 Project 셀렉트에 새 이름 추가
