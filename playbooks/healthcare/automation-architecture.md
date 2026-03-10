# Healthcare AI Automation Architecture

헬스케어 플랫폼의 자동화 기술 구조.

---

## 전체 구조

```
┌─────────────────────────────────────────────────────────────────┐
│                         INPUT LAYER                             │
│                                                                 │
│  웹사이트 문의폼    앱 내 문의    파트너 제휴 폼    Notion 수동  │
└───────┬──────────────┬──────────────┬──────────────┬────────────┘
        │              │              │              │
        ▼              ▼              ▼              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      WEBHOOK LAYER                              │
│                                                                 │
│  /api/webhook/contact    /api/webhook/inquiry    Make.com 직접  │
│  (API Key 인증 + Rate Limit + 입력 검증)                         │
└───────────────────────────┬─────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AUTOMATION LAYER (Make.com)                   │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌────────────────────────┐  │
│  │ 리드 캡처   │  │ 팔로업      │  │ 콘텐츠 파이프라인      │  │
│  │ Webhook →   │  │ Schedule →  │  │ Watch → OpenAI →       │  │
│  │ Notion →    │  │ Query →     │  │ Notion Update →        │  │
│  │ Telegram    │  │ Telegram    │  │ Telegram               │  │
│  └─────────────┘  └─────────────┘  └────────────────────────┘  │
│                                                                 │
└───────────────────────────┬─────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      DATA LAYER (Notion)                        │
│                                                                 │
│  ┌──────────┐  ┌──────────────┐  ┌──────────┐  ┌───────────┐  │
│  │ Leads DB │  │Content Pipe. │  │Proposals │  │Work Inbox │  │
│  │ (CRM)    │  │(마케팅)      │  │(제안서)  │  │(코드작업) │  │
│  └──────────┘  └──────────────┘  └──────────┘  └───────────┘  │
│                                                                 │
└───────────────────────────┬─────────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    NOTIFICATION LAYER                            │
│                                                                 │
│  Telegram    Teams/Slack    Gmail    카카오 알림톡    SMS        │
│  (내부)      (팀)           (고객)   (고객)          (긴급)     │
└─────────────────────────────────────────────────────────────────┘
```

---

## B2B: 파트너 영업 자동화

### 리드 캡처

```
병원/약국 제휴 문의폼
    → /api/webhook/contact (Vercel)
    → Make.com Webhook
    → Notion Leads DB 저장
        - Name: 병원명
        - Type: b2b
        - Source: partner-page
        - Status: new
    → Telegram: "[신규 파트너 문의] OO병원"
    → Gmail: 자동 응답 "제휴 검토 후 연락드리겠습니다"
```

### 팔로업 자동화

```
Schedule (매일 08:00)
    → Notion Leads 쿼리
        - Follow-up Date = 오늘
        - Status != won, lost
    → 결과별 분기:
        - 3일 이상 미연락: Telegram "긴급 팔로업 필요"
        - 오늘 예정: Telegram "팔로업 예정: OO병원"
```

### 리드 스코어링 (향후)

```
신규 리드 도착
    → OpenAI 분석
        - 병원 규모 (의사 수, 월 진료 건수)
        - 문의 내용 긴급도
        - 기존 비대면 진료 경험
    → Score 자동 산정 (1~100)
    → 고스코어(80+): 영업팀 즉시 알림
    → 저스코어: 자동 이메일 + 30일 후 팔로업
```

---

## B2C: 환자 문의 자동화

### AI 자동 분류

```
환자 문의 도착
    → Make.com Webhook
    → OpenAI 분석 (키워드 + 의도 파악)
    → 카테고리 분류:
        ┌─ 결제 이슈 → CS팀 즉시 (긴급)
        ├─ 서비스 불만 → QA팀 (긴급)
        ├─ 일반 문의 → FAQ 자동 응답
        ├─ 배송 문의 → 물류팀
        └─ B2B 문의 → 영업팀
    → Notion에 저장 + 담당자 알림
```

### 환자 여정 자동 알림

```
진료 예약 → "예약 확인" 카카오 알림톡
진료 완료 → "처방약 준비 중" 알림
약 발송   → "배송 시작" 알림 + 배송 추적 링크
약 도착   → "만족도 조사" 알림
```

---

## 콘텐츠 생산 자동화

### 파이프라인

```
Content Pipeline DB에 아이디어 등록 (Status: idea)
    → Make.com Watch (15분 간격)
    → OpenAI GPT-4o:
        "{{Type}} 콘텐츠 초안을 작성하세요.
         제목: {{Title}}
         플랫폼: {{Platform}}
         키워드: {{Keywords}}"
    → Notion Draft 필드에 저장
    → Status: review
    → Telegram: "[콘텐츠 검토 필요] {{Title}}"
    → 마케터 검토 → Approved → 게시
```

### 콘텐츠 유형별 프롬프트

| 유형 | 길이 | 톤 |
|------|------|-----|
| blog | 1000~1500자 | 전문적이면서 친근한 |
| sns-post | 200~300자 | 캐주얼, 이모지 활용 |
| newsletter | 500~800자 | 정보 전달, 구조화 |
| landing-page | 300~500자 | 설득력, CTA 포함 |

---

## 내부 개발 자동화

### 버그 수정

```
CS팀 Notion Work Inbox에 버그 등록
    → ai_process notion-sync 감지 (5분)
    → 해당 레포에 GitHub Issue 생성 (ai-task 라벨)
    → GPT Codex 코드 분석 + 수정
    → PR 생성 → CI 검증
    → Telegram 알림
    → 개발자 승인 → 배포
```

### 기능 요청

```
PM이 Notion Work Inbox에 기능 요청 등록
    → GitHub Issue 생성
    → Codex가 코드 구현
    → PR + CI
    → 코드 리뷰 → 승인 → 배포
```

---

## 환경변수 요약

| 변수 | 용도 | 위치 |
|------|------|------|
| `MAKE_CONTACT_WEBHOOK_URL` | 문의폼 → Make.com | Vercel |
| `MAKE_INQUIRY_WEBHOOK_URL` | 고객문의 → Make.com | Vercel |
| `WEBHOOK_API_KEY` | webhook 인증 | Vercel |
| `ALLOWED_ORIGINS` | CORS 화이트리스트 | Vercel |
| `NOTION_TOKEN` | Notion API 접근 | GitHub Secrets |
| `GH_PAT` | Cross-repo 접근 | GitHub Secrets |
| `TELEGRAM_BOT_TOKEN` | Telegram 알림 | GitHub Secrets + Make.com |
| `OPENAI_API_KEY` | AI 콘텐츠 생성, Codex | GitHub Secrets + Make.com |
