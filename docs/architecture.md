# Architecture - AI Automation Pipeline

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        INPUT LAYER                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │
│  │  Notion   │  │  Cursor  │  │  GitHub  │  │  Telegram (향후) │   │
│  │  Mobile   │  │  Desktop │  │  Issue   │  │  Bot Command     │   │
│  └─────┬────┘  └─────┬────┘  └─────┬────┘  └────────┬─────────┘   │
│        │              │             │                 │             │
└────────┼──────────────┼─────────────┼─────────────────┼─────────────┘
         │              │             │                 │
         ▼              ▼             ▼                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     ORCHESTRATOR LAYER                               │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  ai_process/notion-sync.yml (중앙 오케스트레이터)             │   │
│  │  - Notion DB 폴링 (Processed = false)                        │   │
│  │  - config/projects.json으로 Project → 레포 라우팅             │   │
│  │  - 해당 레포에 GitHub Issue 생성 (ai-task 라벨)               │   │
│  │  - 해당 레포의 Codex workflow_dispatch 호출                   │   │
│  │  - Notion에 Processed 체크 + PR URL 기록                     │   │
│  │  - Telegram 알림 발송                                         │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │                                      │
└──────────────────────────────┼──────────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      CODE AGENT LAYER                               │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  codex.yml (GitHub Actions)                                   │   │
│  │  - openai/codex-action@v1 실행                                │   │
│  │  - sandbox: workspace-write                                   │   │
│  │  - Issue 내용을 프롬프트로 전달                                │   │
│  │  - 코드 수정 → 브랜치 생성 → PR 생성                          │   │
│  │  - Issue에 결과 코멘트 (SUCCESS / FAILED / NO CHANGES)        │   │
│  │  - Telegram 알림 (성공/실패 모두)                              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │                                      │
└──────────────────────────────┼──────────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     VALIDATION LAYER                                │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  ci.yml (GitHub Actions)                                      │   │
│  │  - PR 대상: master 브랜치                                     │   │
│  │  - npm ci → tsc --noEmit → npm run build                     │   │
│  │  - 성공/실패 PR 코멘트                                        │   │
│  │  - 실패 시 Telegram 알림                                      │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  Branch Protection (GitHub Settings)                          │   │
│  │  - Required check: "build"                                    │   │
│  │  - build 실패 시 머지 차단                                    │   │
│  │  - enforce_admins: false (owner 직접 push 허용)               │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │                                      │
└──────────────────────────────┼──────────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     APPROVAL LAYER (수동)                            │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  사람이 PR을 확인하고 Merge 클릭                               │   │
│  │  - GitHub Web / Mobile App에서 가능                            │   │
│  │  - auto-merge는 현재 비활성 (L4 전환 시 활성화)                │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │                                      │
└──────────────────────────────┼──────────────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     DEPLOY + NOTIFICATION LAYER                     │
│                                                                     │
│  ┌────────────────────┐  ┌─────────────────────────────────────┐   │
│  │  Vercel             │  │  deploy-notify.yml                  │   │
│  │  - master push 감지 │  │  - push 이벤트: 배포 알림           │   │
│  │  - 자동 빌드+배포   │  │  - workflow_run: Codex/CI 결과 알림 │   │
│  │  - Preview Deploy   │  │  - Telegram 메시지 발송             │   │
│  └────────────────────┘  └─────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Business Automation Layer

```
┌─────────────────────────────────────────────────────────────────────┐
│                   BUSINESS AUTOMATION LAYER                         │
│                                                                     │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────────┐   │
│  │ /contact 폼  │     │ /api/webhook │     │   Make.com       │   │
│  │ (webscout)   │────▶│ /contact     │────▶│   시나리오       │   │
│  │              │     │ /inquiry     │     │                  │   │
│  └──────────────┘     └──────────────┘     └───────┬──────────┘   │
│                                                     │              │
│                    ┌────────────────────────────────┼──────┐       │
│                    ▼                ▼               ▼      │       │
│              ┌──────────┐   ┌──────────┐   ┌──────────┐   │       │
│              │ Leads DB │   │ Content  │   │Proposals │   │       │
│              │ (CRM)    │   │ Pipeline │   │   DB     │   │       │
│              └──────────┘   └──────────┘   └──────────┘   │       │
│                    │                                       │       │
│                    ▼                                       ▼       │
│              ┌──────────┐                          ┌──────────┐   │
│              │ Telegram │                          │  OpenAI  │   │
│              │ 알림     │                          │ 초안생성 │   │
│              └──────────┘                          └──────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Make.com 시나리오

| 시나리오 | 트리거 | 동작 | 자동화 레벨 |
|---------|--------|------|------------|
| 리드 캡처 | Webhook (문의폼) | Notion Leads 저장 + Telegram 알림 | 완전 자동 |
| 팔로업 리마인더 | 매일 08:00 | Follow-up Date = 오늘인 리드 알림 | 완전 자동 |
| 콘텐츠 파이프라인 | Content DB Status=idea | AI 초안 생성 → 검토 요청 | AI+사람 승인 |

### Notion 비즈니스 DB

| DB | 용도 | Data Source ID |
|----|------|----------------|
| Leads | B2B+B2C 리드 CRM | b398c377-2698-4624-9b00-2a1926f9c790 |
| Content Pipeline | 마케팅 콘텐츠 관리 | 4be97f57-aec9-47bd-b616-105f4bcd3c96 |
| Proposals | B2B 제안서 (Leads relation) | 7371c260-62c2-46bd-92bc-8e957abbb80d |

### Vercel 웹훅 엔드포인트

| 엔드포인트 | 환경변수 | 용도 |
|-----------|---------|------|
| `/api/webhook/contact` | `MAKE_CONTACT_WEBHOOK_URL` | 문의폼 → Make.com |
| `/api/webhook/inquiry` | `MAKE_INQUIRY_WEBHOOK_URL` | 고객문의 → Make.com |

## Data Flow

```
1. INPUT
   Notion DB (Work Inbox)
   ┌──────────────────────────────────────────────────┐
   │ Title │ Task Type │ Goal │ Target Files │ Priority│
   │ ...   │ dev-task  │ ...  │ app/page.tsx │ medium  │
   └──────────────────────────────────────────────────┘
                         │
                         ▼ (5분 폴링)
2. ORCHESTRATOR
   GitHub Issue #N (ai-task 라벨)
   ┌──────────────────────────────────────────────────┐
   │ ## Task                                          │
   │ ## Context (Project, Type, Priority, Notion URL) │
   │ ## Goal (Notion의 Goal 필드)                     │
   │ ### Target Files                                 │
   │ notion:{page-id}                                 │
   └──────────────────────────────────────────────────┘
                         │
                         ▼ (workflow_dispatch)
3. CODE AGENT
   GPT Codex → git diff
   ┌──────────────────────────────────────────────────┐
   │ Branch: codex/issue-N                            │
   │ Commit: feat: {issue title}                      │
   │ Changed: app/page.tsx (1 file)                   │
   └──────────────────────────────────────────────────┘
                         │
                         ▼
4. VALIDATION
   PR #M → CI Build Check
   ┌──────────────────────────────────────────────────┐
   │ npm ci → tsc --noEmit → npm run build            │
   │ Result: PASSED / FAILED                          │
   └──────────────────────────────────────────────────┘
                         │
                         ▼ (사람 승인)
5. DEPLOY
   Vercel → Production
   ┌──────────────────────────────────────────────────┐
   │ https://webscout-next.vercel.app            │
   └──────────────────────────────────────────────────┘
```

## 성숙도 단계

| 레벨 | 이름 | 설명 | 상태 |
|------|------|------|------|
| L1 | 수동 | 모든 단계를 사람이 실행 | 완료 |
| L2 | 반자동 | 일부 단계만 자동, 핵심 연결은 수동 | 완료 |
| **L3** | **준자동** | **트리거부터 결과 통보까지 연결, 승인만 수동** | **현재** |
| L4 | 완전자동 | 입력부터 배포까지 사람 개입 없음 | 향후 |

## 기술 스택

| 구분 | 기술 |
|------|------|
| Code Agent | OpenAI Codex (openai/codex-action@v1) |
| CI/CD | GitHub Actions |
| Deployment | Vercel |
| Task Management | Notion API |
| Notification | Telegram Bot API |
| Version Control | Git + GitHub |
| Framework | Next.js |
| Remote Access | AnyDesk |
| Business Automation | Make.com |
| CRM | Notion Leads DB |
| Webhook | Next.js API Routes (Vercel) |

## Secrets

### ai_process (중앙 오케스트레이터)

| Secret | 용도 |
|--------|------|
| `GH_PAT` | Cross-repo Issue 생성 + workflow 트리거 (Fine-grained PAT) |
| `OPENAI_API_KEY` | GPT Codex API 호출 |
| `TELEGRAM_BOT_TOKEN` | Telegram Bot 메시지 발송 |
| `TELEGRAM_CHAT_ID` | Telegram 수신 대상 |
| `NOTION_TOKEN` | Notion API 접근 (Integration) |

### 각 프로젝트 레포

| Secret | 용도 |
|--------|------|
| `OPENAI_API_KEY` | GPT Codex API 호출 |
| `TELEGRAM_BOT_TOKEN` | Telegram Bot 메시지 발송 |
| `TELEGRAM_CHAT_ID` | Telegram 수신 대상 |
| `GITHUB_TOKEN` | 자동 제공 - Issue/PR 생성 |

## 프로젝트 라우팅 (config/projects.json)

Notion의 Project 필드 값이 key가 되어 해당 레포로 자동 라우팅됩니다.

```json
{
  "webscout-next": { "repo": "dalgoms/webscout-next", "branch": "master", "framework": "nextjs" },
  "ai_process": { "repo": "dalgoms/ai_process", "branch": "master", "framework": "docs" }
}
```

새 프로젝트 추가 시 이 파일에 항목만 추가하면 됩니다.

## Notion DB Schema (Work Inbox)

| 필드 | 타입 | 용도 |
|------|------|------|
| Title | title | 작업 제목 |
| Task Type | select | planning, design-fix, dev-task, bug-fix, refactor, deploy |
| Goal | rich_text | 구체적인 변경 목표 |
| Target Files | rich_text | 수정 대상 파일 경로 |
| Priority | select | urgent, high, medium, low |
| Project | select | webscout-next, timbel-homepage, clipdesk 등 |
| Approval Required | checkbox | 수동 승인 필요 여부 |
| PR URL | url | 자동 기록 - Issue/PR 링크 |
| Error Log | rich_text | 에러 기록 |
| Processed | checkbox | 자동 체크 - sync 완료 여부 |
| Requested At | created_time | 자동 기록 |
