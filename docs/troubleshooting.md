# Troubleshooting & FAQ

> 최종 업데이트: 2026-03-10

## 자주 발생하는 문제

### 1. Notion에 등록했는데 Issue가 안 생겨요

**원인 1: 5분 대기**
- notion-sync는 5분 cron으로 실행됩니다
- 즉시 테스트하려면: Actions 탭 → Notion to GitHub Issue Sync → Run workflow

**원인 2: 기존 행을 수정한 경우**
- 기존 행은 이미 Processed = true입니다
- 반드시 **새 행(+ New)**을 추가해야 합니다

**원인 3: Notion Integration 연결 안 됨**
- Work Inbox 페이지 → `...` → Connections에서 Integration이 연결되어 있는지 확인
- 에러: `Could not find database with ID` → Integration에 DB를 공유해야 합니다

**원인 4: NOTION_TOKEN이 등록되지 않음**
```bash
gh secret list --repo dalgoms/webscout-next
```

### 2. Issue는 생겼는데 Codex가 실행 안 돼요

**원인 1: GITHUB_TOKEN 제한**
- GitHub Actions가 만든 Issue는 다른 워크플로우를 트리거하지 않습니다
- notion-sync가 `workflow_dispatch`로 Codex를 직접 호출해야 합니다
- `notion-sync.yml`에 `gh workflow run` 호출이 있는지 확인

**원인 2: ai-task 라벨 누락**
```bash
gh issue view {NUMBER} --json labels
```

**수동 트리거:**
```bash
gh workflow run "GPT Codex Agent" -f issue_number={NUMBER}
```

### 3. Codex가 "NO CHANGES"로 응답해요

**원인: 프롬프트가 불충분**
- Goal 필드에 **구체적인 파일 경로**와 **변경 내용**을 포함하세요
- Target Files 필드를 반드시 채우세요

```
# 나쁜 예
버튼 바꿔주세요

# 좋은 예
app/analyze/page.tsx에서 "분석 시작" 버튼의 텍스트를 "Start Analysis"로 변경해주세요.
className에서 bg-blue-500을 찾아 bg-indigo-600으로 수정하세요.
```

### 4. Telegram 알림이 안 와요

**확인 1: 시크릿 등록**
```bash
gh secret list --repo dalgoms/webscout-next
# TELEGRAM_BOT_TOKEN과 TELEGRAM_CHAT_ID가 있는지 확인
```

**확인 2: 봇에게 먼저 메시지를 보냈는지**
- Telegram 봇은 사용자가 먼저 대화를 시작해야 메시지를 보낼 수 있습니다
- 봇에게 `/start` 메시지를 보내세요

**확인 3: Actions 로그 확인**
```bash
gh run list --repo dalgoms/webscout-next --limit 5
gh run view {RUN_ID} --log
```

### 5. CI 빌드가 실패해요

**확인: 빌드 로그**
- PR의 코멘트에 빌드 로그 마지막 30줄이 포함됩니다
- Actions 탭에서 전체 로그 확인 가능

**일반적 원인:**
- TypeScript 타입 에러 (tsc는 non-blocking이므로 이것 때문은 아님)
- import 경로 오류
- 존재하지 않는 컴포넌트 참조
- 환경 변수 누락

### 6. PR 머지가 안 돼요

**원인: Branch Protection**
- `build` check가 통과해야 머지 가능합니다
- CI Build Check 워크플로우가 성공했는지 확인하세요

### 7. Codex 실행이 실패해요 (npm 404 에러)

**원인: @openai/codex 패키지 버전 문제**
- openai/codex-action의 특정 버전이 npm에서 404를 반환할 수 있습니다
- `codex.yml`에서 `codex-version`을 특정 버전으로 고정하거나, 최신 action 버전을 사용하세요

### 8. 같은 Issue에 Codex를 다시 실행하고 싶어요

Issue에 `@codex` 코멘트를 달면 재실행됩니다.

```
@codex 다시 실행해주세요. 이번에는 app/layout.tsx의 footer만 수정해주세요.
```

---

## FAQ

**Q: 비용이 발생하나요?**
- GitHub Actions: Public 레포는 무료, Private는 월 2,000분 무료
- OpenAI API: Codex 실행마다 토큰 소비 (작업당 약 $0.01-0.10)
- Vercel: Hobby 플랜 무료
- Notion: 무료 플랜으로 충분

**Q: 여러 프로젝트에 적용할 수 있나요?**
- Notion DB의 Project 필드로 프로젝트를 구분합니다
- 각 프로젝트 레포에 동일한 워크플로우 파일을 배포하세요
- notion-sync의 NOTION_DB_ID를 공유하면 하나의 Inbox로 여러 프로젝트 관리 가능

**Q: auto-merge를 켜도 되나요?**
- L3 단계에서는 수동 승인을 권장합니다
- AI가 생성한 코드를 검토 없이 배포하면 위험할 수 있습니다
- 충분히 검증된 후 L4로 전환할 때 활성화하세요

**Q: 보안은 괜찮나요?**
- Codex sandbox는 `workspace-write`로 제한 (파일 시스템만 접근)
- API 키는 GitHub Secrets에 암호화 저장
- Branch Protection으로 빌드 실패 코드 머지 차단
- Issue 내용이 prompt injection에 취약할 수 있으므로 Public 레포에서는 주의
