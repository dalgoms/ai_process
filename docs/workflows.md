# Workflows - GitHub Actions 상세

## 전체 워크플로우 맵

```
                    ai_process 레포
                    ┌────────────────────┐
                    │ notion-sync.yml    │
                    │ (중앙 오케스트레이터)│
                    └───────┬────────────┘
                            │ Project 라우팅
               ┌────────────┼────────────┐
               ▼            ▼            ▼
          webscout-next  ai_process   new-project
          ┌──────────┐  ┌──────────┐  ┌──────────┐
          │codex.yml │  │codex.yml │  │codex.yml │
          │ci.yml    │  │notify.yml│  │ci.yml    │
          │notify.yml│  └──────────┘  │notify.yml│
          └──────────┘                └──────────┘
```

---

## 1. notion-sync.yml (ai_process 레포)

**역할**: Notion Work Inbox를 5분마다 확인하고, Project 필드 기반으로 올바른 레포에 Issue를 생성

| 항목 | 값 |
|------|-----|
| 위치 | `ai_process/.github/workflows/notion-sync.yml` |
| 트리거 | `schedule: */5 * * * *`, `workflow_dispatch` |
| 필요 시크릿 | `GH_PAT`, `NOTION_TOKEN`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` |

### 동작 흐름

```
1. config/projects.json을 로드 (프로젝트 → 레포 매핑)
2. Notion API로 DB 쿼리 (Processed = false 필터)
3. 미처리 항목 발견 시:
   a. Project 필드로 TARGET_REPO 결정
   b. 매핑 없으면 Telegram으로 에러 알림 → skip
   c. TARGET_REPO에 GitHub Issue 생성 (ai-task 라벨)
   d. TARGET_REPO의 Codex 워크플로우 dispatch (issue_number 전달)
   e. Notion에 Processed = true 업데이트
   f. Notion에 PR URL = Issue URL 기록
   g. Telegram 알림 발송 (레포 정보 포함)
4. 미처리 항목 없으면 조용히 종료
```

### 중복 방지

Issue body에 `notion:{page-id}`를 포함하고, 생성 전 동일 ID로 검색하여 중복 생성 방지

### Cross-repo 접근

`GITHUB_TOKEN`은 현재 레포만 접근 가능하므로, `GH_PAT` (Fine-grained PAT)를 사용하여 다른 레포에 Issue 생성 및 workflow dispatch를 수행

---

## 2. codex.yml

**역할**: GPT Codex를 실행하여 Issue의 요구사항대로 코드를 수정하고 PR 생성

| 항목 | 값 |
|------|-----|
| 트리거 | `issues: opened`, `issue_comment: created`, `workflow_dispatch` |
| 필요 시크릿 | `OPENAI_API_KEY`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` |
| 권한 | contents: write, pull-requests: write, issues: write |

### 동작 흐름

```
1. Issue 정보 조회 (issue_number로 title, body 가져오기)
2. openai/codex-action@v1 실행
   - sandbox: workspace-write
   - --full-auto 모드
3. 결과 분기:
   ├── FAILURE → Issue 코멘트 + Telegram 알림
   ├── NO CHANGES → Issue 코멘트
   └── SUCCESS → 브랜치 생성 → PR 생성 → Issue 코멘트 + Telegram 알림
```

### Issue 코멘트 형식

모든 코멘트는 테이블 형식으로 구조화:

```markdown
## Codex Result: SUCCESS / FAILED / NO CHANGES

| Field | Value |
|-------|-------|
| Status | ... |
| Issue | #N |
| PR | URL (성공 시) |
| Files changed | N (성공 시) |
| Run | [View logs](URL) |

### Next Action
1. ...
```

### 브랜치 네이밍

- 기본: `codex/issue-{number}`
- 이미 존재 시: `codex/issue-{number}-{timestamp}`

---

## 3. ci.yml

**역할**: PR의 코드가 빌드 가능한지 검증

| 항목 | 값 |
|------|-----|
| 트리거 | `pull_request: branches: [master]` |
| 필요 시크릿 | `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` |

### 동작 흐름

```
1. Node.js 20 + npm ci
2. tsc --noEmit (타입 체크, non-blocking)
3. npm run build (빌드, blocking)
4. 결과:
   ├── 성공 → PR에 PASSED 코멘트
   └── 실패 → PR에 FAILED 코멘트 (빌드 로그 포함) + Telegram 알림
```

### Branch Protection 연동

GitHub에서 `build` job이 required check로 설정되어 있어, CI 실패 시 PR 머지가 차단됩니다.

---

## 4. deploy-notify.yml

**역할**: 모든 중요 이벤트를 Telegram으로 알림

| 항목 | 값 |
|------|-----|
| 트리거 | `push: master`, `workflow_run: [GPT Codex Agent, CI Build Check]` |
| 필요 시크릿 | `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` |

### 알림 유형

| 이벤트 | 메시지 형식 |
|--------|-------------|
| master push | `[DEPLOY SUCCESS] repo, branch, commit, author, SHA, run URL` |
| Codex 성공 | `[SUCCESS] GPT Codex Agent, repo, branch, trigger, SHA, run URL` |
| Codex 실패 | `[FAILED] GPT Codex Agent, repo, branch, trigger, SHA, run URL` |
| CI 성공 | `[SUCCESS] CI Build Check, ...` |
| CI 실패 | `[FAILED] CI Build Check, ...` |

---

## 5. auto-merge.yml (비활성)

**역할**: 빌드 성공 시 auto-merge 라벨이 붙은 PR을 자동 머지

| 항목 | 값 |
|------|-----|
| 트리거 | `workflow_dispatch` (현재 비활성) |
| 상태 | `if: false` - L4 전환 시 활성화 |

---

## 6. Make.com 시나리오 (비즈니스 자동화)

코드 파이프라인과 별도로 Make.com이 비즈니스 자동화를 처리합니다.

### 전체 흐름

```
웹사이트 /contact 폼
    → /api/webhook/contact (Vercel)
    → Make.com Webhook
    → Notion Leads DB 저장
    → Telegram 알림
```

### 시나리오 1: 리드 캡처 (라이브)

| 항목 | 값 |
|------|-----|
| 트리거 | Custom Webhook (Vercel에서 POST) |
| 모듈 | Webhook → Notion Create Item → Telegram Send Message |
| 자동화 | 완전 자동 |
| 환경변수 | `MAKE_CONTACT_WEBHOOK_URL` (Vercel) |

### 시나리오 2: 팔로업 리마인더 (설계 완료)

| 항목 | 값 |
|------|-----|
| 트리거 | Schedule (매일 08:00) |
| 모듈 | Schedule → Notion Search → Iterator → Telegram |
| 필터 | Follow-up Date = 오늘, Status != won/lost |

### 시나리오 3: 콘텐츠 파이프라인 (설계 완료)

| 항목 | 값 |
|------|-----|
| 트리거 | Notion Watch (Content Pipeline, Status=idea) |
| 모듈 | Notion Watch → OpenAI Completion → Notion Update → Telegram |
| 자동화 | AI 생성 → 사람 승인 |

상세 설정은 [make-scenarios.md](./make-scenarios.md) 참조

---

## GITHUB_TOKEN 제한 사항

GitHub Actions의 `GITHUB_TOKEN`으로 생성된 이벤트(Issue, Comment 등)는 다른 워크플로우를 트리거하지 않습니다. 이 보안 정책 때문에:

- `notion-sync`가 Issue를 만든 후 `workflow_dispatch`로 Codex를 직접 호출
- 향후 더 복잡한 체이닝이 필요하면 PAT(Personal Access Token) 사용 검토
