# Healthcare AI Automation Architecture

헬스케어 플랫폼의 자동화 기술 구조.

---

## 전체 구조

```mermaid
flowchart TB
    subgraph input ["Input Layer"]
        WebForm["Website Contact Form"]
        AppInquiry["App Inquiry"]
        PartnerForm["Partner Form"]
        NotionManual["Notion Manual"]
    end

    subgraph webhook ["Webhook Layer"]
        ContactAPI["/api/webhook/contact"]
        InquiryAPI["/api/webhook/inquiry"]
    end

    subgraph automation ["Make.com Automation"]
        LeadCapture["Lead Capture"]
        FollowUp["Follow-up Reminder"]
        ContentPipe["Content Pipeline"]
    end

    subgraph data ["Notion Data Layer"]
        LeadsDB["Leads DB"]
        ContentDB["Content Pipeline DB"]
        ProposalsDB["Proposals DB"]
        WorkInbox["Work Inbox"]
    end

    subgraph notify ["Notification Layer"]
        Telegram["Telegram"]
        Teams["Teams / Slack"]
        Gmail["Gmail"]
        Kakao["KakaoTalk"]
        SMS["SMS"]
    end

    WebForm --> ContactAPI
    AppInquiry --> InquiryAPI
    PartnerForm --> ContactAPI
    NotionManual --> WorkInbox

    ContactAPI --> LeadCapture
    InquiryAPI --> LeadCapture
    LeadCapture --> LeadsDB
    LeadCapture --> Telegram
    LeadCapture --> Gmail

    LeadsDB --> FollowUp
    FollowUp --> Telegram

    ContentDB --> ContentPipe
    ContentPipe --> Telegram
```

---

## B2B: 파트너 영업 자동화

### 리드 캡처

```mermaid
flowchart LR
    Form["Partner Inquiry Form"] --> Webhook["/api/webhook/contact"]
    Webhook --> Make["Make.com"]
    Make --> Notion["Notion Leads DB"]
    Make --> Telegram["Telegram Alert"]
    Make --> Gmail["Auto-reply Email"]
```

Notion Leads DB 필드: Name(병원명), Type(b2b), Source(partner-page), Status(new)

### 팔로업 자동화

```mermaid
flowchart LR
    Schedule["Schedule 08:00"] --> Query["Notion Leads Query"]
    Query --> Filter["Follow-up Date = Today"]
    Filter --> Router{"Overdue?"}
    Router -- "3+ days" --> Urgent["Telegram: Urgent Follow-up"]
    Router -- "Today" --> Normal["Telegram: Follow-up Scheduled"]
```

### 리드 스코어링 (향후)

```mermaid
flowchart LR
    Lead["New Lead"] --> AI["OpenAI Analysis"]
    AI --> Score["Score (1~100)"]
    Score --> High{"Score >= 80?"}
    High -- Yes --> SalesAlert["Sales Team Immediate Alert"]
    High -- No --> AutoEmail["Auto Email + 30-day Follow-up"]
```

---

## B2C: 환자 문의 자동화

### AI 자동 분류

```mermaid
flowchart TB
    Inquiry["Patient Inquiry"] --> Make["Make.com Webhook"]
    Make --> AI["OpenAI Classification"]
    AI --> Router{"Category"}
    Router -- "Payment" --> CS["CS Team (Urgent)"]
    Router -- "Complaint" --> QA["QA Team (Urgent)"]
    Router -- "General" --> FAQ["Auto FAQ Response"]
    Router -- "Delivery" --> Logistics["Logistics Team"]
    Router -- "B2B" --> Sales["Sales Team"]
    CS --> Notion["Save to Notion"]
    QA --> Notion
    FAQ --> Notion
    Logistics --> Notion
    Sales --> Notion
```

### 환자 여정 자동 알림

```mermaid
flowchart LR
    Booking["진료 예약"] --> Confirm["예약 확인 알림톡"]
    Confirm --> Complete["진료 완료"]
    Complete --> Shipped["배송 시작 + 추적 링크"]
    Shipped --> Arrived["만족도 조사 알림"]
```

---

## 콘텐츠 생산 자동화

### 파이프라인

```mermaid
flowchart LR
    Idea["Register Idea (Status: idea)"] --> Watch["Make.com Watch (15min)"]
    Watch --> AI["OpenAI GPT-4o Draft"]
    AI --> Save["Save to Notion Draft"]
    Save --> Review["Status: review"]
    Review --> Alert["Telegram Alert"]
    Alert --> Approve["Marketer Review"]
    Approve --> Publish["Publish"]
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

```mermaid
flowchart LR
    Bug["CS Bug Report (Notion)"] --> Sync["notion-sync (5min)"]
    Sync --> Issue["GitHub Issue (ai-task)"]
    Issue --> Codex["GPT Codex Fix"]
    Codex --> PR["PR + CI"]
    PR --> Alert["Telegram Alert"]
    Alert --> Approve["Developer Approval"]
    Approve --> Deploy["Deploy"]
```

### 기능 요청

```mermaid
flowchart LR
    Feature["PM Feature Request (Notion)"] --> Issue["GitHub Issue"]
    Issue --> Codex["Codex Implementation"]
    Codex --> PR["PR + CI"]
    PR --> Review["Code Review"]
    Review --> Deploy["Approve + Deploy"]
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
