# Changelog - 구축 이력

## 2026-03-10 (Business Automation)

### Phase 5: 비즈니스 자동화 기반
- Notion Leads DB 생성 (B2B+B2C CRM, Proposals relation 연결)
- Notion Content Pipeline DB 생성 (마케팅 콘텐츠 관리)
- Notion Proposals DB 생성 (B2B 제안서 관리)
- Vercel 웹훅 엔드포인트 추가 (`/api/webhook/contact`, `/api/webhook/inquiry`)
- Make.com 시나리오 1 (리드 캡처) 라이브 배포: Webhook → Notion Leads → Telegram
- Make.com 시나리오 2, 3 설계 완료 (팔로업 리마인더, 콘텐츠 파이프라인)
- `/contact` 문의 페이지 생성 + 헤더 네비게이션 링크 추가
- Vercel 환경변수 `MAKE_CONTACT_WEBHOOK_URL` 등록
- Notion에 Make.com 연동 가이드 페이지 생성
- 전체 문서 업데이트 (architecture, workflows, setup-guide, troubleshooting)

---

## 2026-03-10 (Multi-Project Upgrade)

### Phase 4: 멀티 프로젝트 확장
- `notion-sync.yml`을 `webscout-next`에서 `ai_process`로 이동 (중앙 오케스트레이터)
- `config/projects.json` 프로젝트 라우팅 설정 추가
- Notion Project 필드 → 자동 레포 라우팅 로직 구현
- `GH_PAT` (Fine-grained PAT) 도입 → cross-repo Issue/workflow 접근
- `codex.yml` 프롬프트 범용화 (`.codex-context` 파일 기반)
- `ai_process` 레포에 `codex.yml`, `deploy-notify.yml` 추가
- `ai_process` 레포에 `ai-task` 라벨 생성
- README, architecture, workflows 문서 멀티 프로젝트 구조로 업데이트

---

## 2026-03-10 (Initial Build)

### Phase 0: 기반 구축
- AnyDesk 원격 접속 설정 (PC ↔ Mobile)
- GitHub 계정 연동 (`dalgoms`)
- Vercel 연동 (`aiagenty`)
- `webscout-next` 프로젝트 GitHub + Vercel 배포

### Phase 1: 알림 자동화
- Telegram Bot 생성 (`@seyounginboxbot`)
- `deploy-notify.yml` 생성 → master push 시 Telegram 알림
- Notion Work Inbox 데이터베이스 생성

### Phase 2: AI 코딩 에이전트
- OpenAI Codex Action 설정 (`codex.yml`)
- GitHub Issue (`ai-task` 라벨) → Codex 자동 실행 → PR 생성
- 테스트: footer 연도 표시 추가 (Issue #1 → PR #2)

### Phase 3: 파이프라인 통합
- Notion → GitHub Issue 자동 생성 (`notion-sync.yml`)
- CI 빌드 검증 (`ci.yml`) + Branch Protection
- Codex 결과 피드백 (Issue 코멘트: SUCCESS/FAILED/NO CHANGES)
- 전체 워크플로우 Telegram 알림 (push + workflow_run)
- Codex/CI 실패 시 Telegram + Issue 코멘트 동시 발송

### L2 → L3 업그레이드
- `deploy-notify.yml` 수정: step-level secrets 버그 수정
- `codex.yml` 수정: sandbox `danger-full-access` → `workspace-write`
- `codex.yml` 수정: 중복 실행 방지 (`opened` + `labeled` → `opened`만)
- `codex.yml` 수정: `workflow_dispatch` 지원 추가
- `notion-sync.yml` 수정: Status → Processed 체크박스 방식 전환
- `notion-sync.yml` 수정: Issue 생성 후 Codex `workflow_dispatch` 직접 호출
- Notion DB 스키마 업데이트 (Task Type, Goal, Target Files, PR URL, Error Log, Processed 등)
- Branch Protection 설정 (`build` required check)
- `auto-merge.yml` 비활성화 (L3 = 승인형 반자동)

### E2E 검증 완료
- Notion 등록 → Issue 자동 생성 → Codex 코드 수정 → PR 생성 → CI 검증 → 수동 머지 → Vercel 배포 → Telegram 알림
- 테스트: "분석 버튼 텍스트 변경" (Notion → Issue #8 → PR #9 → 배포)

---

## 해결한 버그들

| 버그 | 원인 | 해결 |
|------|------|------|
| deploy-notify 매번 실패 | step-level `if`에서 secrets 직접 비교 불가 | job-level `env`로 매핑 후 `env.VAR != ''` 사용 |
| deploy-notify YAML 파싱 에러 | 멀티라인 이모지+HTML 조합 | 단순 텍스트로 변경 |
| Codex 2번 실행 | `issues: [opened, labeled]` 동시 발생 | `[opened]`만 사용 |
| Codex sandbox 에러 | `unsafe` 는 유효하지 않은 값 | `workspace-write` 사용 |
| Notion sync 0건 | Status 속성이 API에서 미지원 | Processed 체크박스 방식 전환 |
| Notion API 404 | Integration이 DB에 미연결 | Connections에서 Integration 추가 |
| Issue → Codex 미트리거 | GITHUB_TOKEN 이벤트는 워크플로우 트리거 불가 | `workflow_dispatch`로 직접 호출 |
| `mkdir -p` 실패 | PowerShell에서 미지원 | `New-Item -ItemType Directory -Force` 사용 |
| git push 거부 | 원격에 새 커밋 존재 | `git pull --no-edit` 후 push |

## 참조 링크

- [OpenAI Codex GitHub Action](https://github.com/openai/codex-action)
- [Codex Action 공식 문서](https://developers.openai.com/codex/github-action/)
- [Notion API 공식 문서](https://developers.notion.com/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [GitHub Actions 공식 문서](https://docs.github.com/en/actions)
- [GitHub Branch Protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-a-branch-protection-rule)
