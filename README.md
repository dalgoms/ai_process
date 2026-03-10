# AI Process

원격 AI 자동화 파이프라인 - 모바일에서 작업을 등록하면 AI가 코드를 작성하고 배포합니다.

## Pipeline

```
Notion 등록 → ai_process가 프로젝트 식별 → 해당 레포에 Issue 생성 → GPT Codex 코드 수정 → PR 생성 → CI 검증 → Telegram 알림 → 수동 승인 → 배포
```

## 멀티 프로젝트 지원

하나의 Notion Work Inbox에서 여러 프로젝트를 관리합니다. **Project** 필드를 기반으로 자동 라우팅됩니다.

| Project | Repository | 용도 |
|---------|-----------|------|
| `webscout-next` | [dalgoms/webscout-next](https://github.com/dalgoms/webscout-next) | Next.js 메인 서비스 |
| `ai_process` | [dalgoms/ai_process](https://github.com/dalgoms/ai_process) | 자동화 문서 관리 |

새 프로젝트를 추가하려면 `config/projects.json`에 항목을 추가하세요.

## 사용법

### 1. Notion Work Inbox에서 새 작업 등록

| 필드 | 설명 | 예시 |
|------|------|------|
| **Title** | 작업 제목 | `로그인 버튼 색상 변경` |
| **Task Type** | 작업 유형 | `dev-task`, `design-fix`, `bug-fix`, `refactor` |
| **Goal** | 구체적인 목표 | `app/page.tsx에서 로그인 버튼 배경색을 #2563eb로 변경` |
| **Target Files** | 수정 대상 파일 | `app/page.tsx` |
| **Priority** | 우선순위 | `urgent`, `high`, `medium`, `low` |
| **Project** | 프로젝트 | `webscout-next`, `ai_process` |

### 2. 자동으로 일어나는 일

1. **5분 내** - ai_process의 notion-sync가 Notion을 확인
2. **즉시** - Project 필드에 맞는 레포에 GitHub Issue 생성
3. **~2분** - 해당 레포의 GPT Codex가 코드를 수정
4. **즉시** - PR 생성 + CI 빌드 검증 시작
5. **Telegram** - 각 단계마다 알림 수신

### 3. 최종 승인

GitHub에서 PR을 확인하고 **Merge** 클릭 → 자동 배포

## 프롬프트 팁

Goal을 작성할 때 구체적일수록 정확합니다:

```
# 좋은 예
app/analyze/page.tsx에서 "분석 시작" 버튼의 텍스트를 "Start Analysis"로 변경해주세요

# 나쁜 예
버튼 바꿔주세요
```

핵심 규칙:
- **파일 경로**를 명시하세요
- **before/after**를 설명하세요
- **한 번에 하나의 작업**만 요청하세요

## 실행 방법 (어디서든)

### 방법 1: Notion (모바일/PC)

Work Inbox에 새 행 추가 → Project 필드 선택 → 5분 내 자동 실행

### 방법 2: Cursor에서 대화로

아래처럼 말하면 됩니다:

```
Notion에 등록해줘: [제목], [변경사항], [파일 경로]
```

예시:
- "Notion에 등록해줘: 메인 페이지 배경색 변경, app/page.tsx에서 bg-white를 bg-gray-50으로"
- "Notion에 등록해줘: 분석 결과 다운로드 버튼 추가, app/analyze/page.tsx"
- "Notion에 등록해줘: footer 링크 수정, app/layout.tsx에서 GitHub 링크를 https://github.com/dalgoms로 변경"

### 방법 3: GitHub Issue 직접 생성

해당 프로젝트 레포에서 Issue를 만들 때 `ai-task` 라벨만 붙이면 Codex가 자동 실행

### 방법 4: 재시도

Issue에 `@codex` 코멘트를 달면 다시 실행

## Architecture

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

## Workflows

### 중앙 (ai_process 레포)

| 파일 | 트리거 | 동작 |
|------|--------|------|
| `notion-sync.yml` | 5분 cron | Notion 폴링 → Project별 레포 라우팅 → Issue 생성 → Codex 트리거 |

### 프로젝트별 (각 레포)

| 파일 | 트리거 | 동작 |
|------|--------|------|
| `codex.yml` | Issue/comment/dispatch | GPT Codex 실행 → PR 생성 |
| `ci.yml` | PR | 빌드 검증 (프로젝트에 따라 다름) |
| `deploy-notify.yml` | push/workflow_run | Telegram 알림 |

## 프로젝트 추가 방법

1. GitHub 레포 생성
2. `config/projects.json`에 항목 추가
3. PAT의 Repository access에 새 레포 추가
4. 새 레포에 `codex.yml`, `ci.yml`, `deploy-notify.yml` 복사
5. 새 레포에 Secrets 등록
6. `.codex-context` 파일 생성 (프로젝트 설명)
7. Notion Work Inbox의 Project 셀렉트에 새 이름 추가

## 비즈니스 자동화

코드 파이프라인 외에 마케팅/세일즈 자동화도 운영합니다.

| Notion DB | 용도 | Make.com 연동 |
|-----------|------|--------------|
| [Leads](https://www.notion.so/49f20ec826ed41ff9f5cc137af122ed3) | B2B+B2C 리드 CRM | 리드 캡처, 팔로업 리마인더 |
| [Content Pipeline](https://www.notion.so/69a39bc304664c658ec1b1fb7117c100) | 마케팅 콘텐츠 관리 | AI 초안 생성 |
| [Proposals](https://www.notion.so/3e86d8a31dfd4fc5829a6ed4482b3708) | B2B 제안서 관리 | (향후) |

웹훅 엔드포인트: `/api/webhook/contact`, `/api/webhook/inquiry`

## Connected Services

| 서비스 | 역할 |
|--------|------|
| [Notion Work Inbox](https://www.notion.so/7f471e8dcba44e878b96cfae8d0de083) | 작업 등록 (모든 프로젝트 통합) |
| [webscout-next](https://github.com/dalgoms/webscout-next) | Next.js 서비스 |
| [ai_process](https://github.com/dalgoms/ai_process) | 중앙 오케스트레이터 + 문서 |
| [Vercel](https://webscout-next-8veo.vercel.app) | 배포 |
| Telegram (@seyounginboxbot) | 알림 |
| Make.com | 비즈니스 자동화 (리드, 콘텐츠, 팔로업) |

## 상세 문서

| 문서 | 내용 |
|------|------|
| [Architecture](docs/architecture.md) | 시스템 구성도, 데이터 흐름, 기술 스택, DB 스키마 |
| [Setup Guide](docs/setup-guide.md) | 처음부터 전체 파이프라인 구축하는 방법 |
| [Workflows](docs/workflows.md) | 각 GitHub Actions 워크플로우 상세 동작 |
| [Make.com Scenarios](docs/make-scenarios.md) | Make.com 시나리오 설정 가이드 (리드, 팔로업, 콘텐츠) |
| [Business Case: Healthcare](docs/business-case-healthcare.md) | 헬스케어 플랫폼 적용 사례 (B2B/B2C/내부 자동화) |
| [Notification Channels](docs/notification-channels.md) | 알림 채널 가이드 (Teams, Slack, 카카오, SMS, Gmail) |
| [Troubleshooting](docs/troubleshooting.md) | 자주 발생하는 문제 해결 + FAQ |
| [Changelog](docs/changelog.md) | 구축 이력, 해결한 버그, 참조 링크 |
