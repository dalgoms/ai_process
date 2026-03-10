# Setup Guide - 처음부터 구축하기

이 파이프라인을 새 프로젝트에 적용하거나 다른 PC에서 동일하게 세팅할 때 참고하세요.

## 1. 사전 준비

### 필요한 계정
- GitHub 계정
- Vercel 계정 (GitHub 연동)
- Notion 계정
- Telegram 계정
- OpenAI API 키

### 필요한 도구 (PC 설치)
- Git
- GitHub CLI (`gh`)
- Node.js 20+
- AnyDesk (원격 접속용)

## 2. GitHub Repository 설정

### 2-1. 프로젝트 레포 생성

```bash
gh repo create my-project --public
cd my-project
```

### 2-2. GitHub Secrets 등록 (역할별)

```bash
# 프로젝트 레포 (codex.yml, deploy-notify.yml)
gh secret set OPENAI_API_KEY --body "sk-..."
gh secret set TELEGRAM_BOT_TOKEN --body "123456:ABC..."
gh secret set TELEGRAM_CHAT_ID --body "987654321"
```

```bash
# 중앙 오케스트레이터 레포 (ai_process/notion-sync.yml)
gh secret set NOTION_TOKEN --body "ntn_..."
gh secret set GH_PAT --body "github_pat_..."
```

### 2-3. Branch Protection 설정

```bash
echo '{
  "required_status_checks": {
    "strict": true,
    "contexts": ["build"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": null,
  "restrictions": null
}' | gh api repos/OWNER/REPO/branches/master/protection -X PUT --input -
```

### 2-4. GitHub Labels 생성

```bash
gh label create "ai-task" --description "GPT Codex가 자동으로 처리할 작업" --color "5319e7"
gh label create "auto-merge" --description "빌드 성공 시 자동 머지" --color "0e8a16"
```

### 2-5. Actions 설정 (GitHub Web)

Settings → Actions → General:
- **Workflow permissions**: "Read and write permissions" 선택
- **Allow GitHub Actions to create and approve pull requests**: 체크
- Save

## 3. Telegram Bot 설정

### 3-1. Bot 생성

1. Telegram에서 `@BotFather` 검색
2. `/newbot` 입력
3. 봇 이름과 username 설정
4. 받은 토큰을 저장

### 3-2. Chat ID 확인

1. 만든 봇에게 아무 메시지 전송
2. 아래 URL 접속 (TOKEN 부분을 실제 토큰으로 교체):

```
https://api.telegram.org/bot{TOKEN}/getUpdates
```

3. `chat.id` 값이 Chat ID

## 4. Notion 설정

### 4-1. Integration 생성

1. https://www.notion.so/profile/integrations 접속
2. "New integration" 클릭
3. 이름 설정 (예: "GitHub Sync")
4. Capabilities: Read content, Update content 체크
5. Submit → Internal Integration Secret 복사

### 4-2. Work Inbox 데이터베이스 생성

Notion에서 새 데이터베이스를 생성하고 아래 필드를 추가:

| 필드 | 타입 |
|------|------|
| Title | Title |
| Task Type | Select (planning, design-fix, dev-task, bug-fix, refactor, deploy) |
| Goal | Text |
| Target Files | Text |
| Priority | Select (urgent, high, medium, low) |
| Project | Select (프로젝트별) |
| Approval Required | Checkbox |
| PR URL | URL |
| Error Log | Text |
| Processed | Checkbox |

### 4-3. Integration 연결

Work Inbox 페이지 → 우측 상단 `...` → Connections → 만든 Integration 추가

## 5. GitHub Actions 워크플로우 배포

`.github/workflows/` 폴더에 아래 파일들을 생성:

- `notion-sync.yml` - Notion 폴링 + Issue 생성
- `codex.yml` - GPT Codex 실행 + PR 생성
- `ci.yml` - 빌드 검증
- `deploy-notify.yml` - Telegram 알림
- `auto-merge.yml` - (비활성, 향후 사용)

각 파일의 상세 내용은 [workflows.md](./workflows.md) 참조

## 6. Vercel 연동

1. https://vercel.com 에서 GitHub 레포 Import
2. Framework: Next.js 자동 감지
3. Deploy → 도메인 확인
4. `deploy-notify.yml`에서 SITE_URL 수정

## 7. 검증 체크리스트

- [ ] 프로젝트 레포에 `OPENAI_API_KEY`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` 등록 확인
- [ ] 중앙 레포(`ai_process`)에 `NOTION_TOKEN`, `GH_PAT`, `TELEGRAM_*` 등록 확인
- [ ] Notion Work Inbox에 테스트 항목 등록
- [ ] `gh workflow run "Notion to GitHub Issue Sync"`로 수동 트리거
- [ ] Issue 생성 → Codex 실행 → PR 생성 확인
- [ ] PR 머지 → Vercel 배포 → Telegram 알림 확인

## 8. 새 PC에서 동일 환경 구축

```bash
# 도구 설치
winget install Git.Git
winget install GitHub.cli
winget install OpenJS.NodeJS.LTS
winget install AnyDesk.AnyDesk

# GitHub 로그인
gh auth login

# 프로젝트 클론
mkdir D:\github
cd D:\github
gh repo clone dalgoms/webscout-next
gh repo clone dalgoms/ai_process

# AnyDesk 상시 실행 설정
# AnyDesk → Settings → Security → 무인 접속 비밀번호 설정
```
