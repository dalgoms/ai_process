# AI Process

원격 AI 자동화 파이프라인 - 모바일에서 작업을 등록하면 AI가 코드를 작성하고 배포합니다.

## Pipeline

```
Notion 등록 → GitHub Issue 자동 생성 → GPT Codex 코드 수정 → PR 생성 → CI 빌드 검증 → Telegram 알림 → 수동 승인 → Vercel 배포
```

## 사용법

### 1. Notion Work Inbox에서 새 작업 등록

| 필드 | 설명 | 예시 |
|------|------|------|
| **Title** | 작업 제목 | `로그인 버튼 색상 변경` |
| **Task Type** | 작업 유형 | `dev-task`, `design-fix`, `bug-fix`, `refactor` |
| **Goal** | 구체적인 목표 | `app/page.tsx에서 로그인 버튼 배경색을 #2563eb로 변경` |
| **Target Files** | 수정 대상 파일 | `app/page.tsx` |
| **Priority** | 우선순위 | `urgent`, `high`, `medium`, `low` |
| **Project** | 프로젝트 | `webscout-next` |

### 2. 자동으로 일어나는 일

1. **5분 내** - notion-sync가 Notion을 확인하고 GitHub Issue 생성
2. **~2분** - GPT Codex가 코드를 읽고 수정
3. **즉시** - PR 생성 + CI 빌드 검증 시작
4. **Telegram** - 각 단계마다 알림 수신

### 3. 최종 승인

GitHub에서 PR을 확인하고 **Merge** 클릭 → Vercel 자동 배포

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

Work Inbox에 새 행 추가 → 5분 내 자동 실행

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

Issue를 만들 때 `ai-task` 라벨만 붙이면 Codex가 자동 실행

### 방법 4: 재시도

Issue에 `@codex` 코멘트를 달면 다시 실행

## Connected Services

| 서비스 | 역할 |
|--------|------|
| [Notion Work Inbox](https://www.notion.so/7f471e8dcba44e878b96cfae8d0de083) | 작업 등록 |
| [webscout-next](https://github.com/dalgoms/webscout-next) | 메인 프로젝트 |
| [Vercel](https://webscout-next-8veo.vercel.app) | 배포 |
| Telegram (@seyounginboxbot) | 알림 |

## Architecture

```
┌─────────┐     ┌──────────┐     ┌───────────┐     ┌──────┐     ┌──────────┐
│  Notion  │────▶│  GitHub   │────▶│ GPT Codex │────▶│  PR  │────▶│ CI Build │
│  Inbox   │     │  Issue    │     │  Agent    │     │      │     │  Check   │
└─────────┘     └──────────┘     └───────────┘     └──────┘     └──────────┘
     │                                                                │
     │                                                          ┌─────▼─────┐
     │               ┌──────────┐     ┌────────┐               │  Telegram  │
     │               │  Vercel  │◀────│ Merge  │◀──── 사람 승인 │  Notify   │
     │               │  Deploy  │     │        │               └───────────┘
     │               └──────────┘     └────────┘
     │                    │
     └────────────────────┘── Telegram 배포 알림
```

## Workflows

| 파일 | 트리거 | 동작 |
|------|--------|------|
| `notion-sync.yml` | 5분 cron | Notion → Issue 생성 → Codex 트리거 |
| `codex.yml` | Issue/comment/dispatch | GPT Codex 실행 → PR 생성 |
| `ci.yml` | PR | npm run build 검증 |
| `deploy-notify.yml` | push/workflow_run | Telegram 알림 |
| `auto-merge.yml` | 비활성 | 향후 L4 자동화용 |
