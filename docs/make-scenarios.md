# Make.com 시나리오 가이드

## 사전 준비

### 1. Make.com 연결 설정

Make.com에 로그인 후 아래 서비스를 연결하세요.

**Notion 연결:**
1. Make.com → Connections → Add → Notion
2. OAuth 인증으로 워크스페이스 연결
3. Leads, Content Pipeline, Proposals DB에 접근 허용

**Telegram 연결:**
1. Make.com → Connections → Add → Telegram Bot
2. Bot Token 입력: `@BotFather`에서 받은 토큰

**OpenAI 연결 (콘텐츠 파이프라인용):**
1. Make.com → Connections → Add → OpenAI
2. API Key 입력

### 2. Webhook URL 생성

Make.com에서 시나리오 1(리드 캡처)을 만들 때 Custom Webhook 모듈의 URL을 복사하여
Vercel 환경변수에 등록합니다:

```
MAKE_CONTACT_WEBHOOK_URL=https://hook.us1.make.com/xxxxx
MAKE_INQUIRY_WEBHOOK_URL=https://hook.us1.make.com/yyyyy
```

Vercel Dashboard → webscout-next → Settings → Environment Variables에서 등록

---

## 시나리오 1: 리드 캡처

**목적:** 웹사이트 문의폼 데이터를 자동으로 Notion에 저장하고 Telegram으로 알림

**자동화 레벨:** 완전 자동

**Make.com 설정:**

```
[Webhook] → [Notion: Create Page] → [Telegram: Send Message]
```

### 모듈 1: Webhooks - Custom webhook

- 이름: "Lead Capture Webhook"
- 생성된 URL을 Vercel 환경변수에 등록

### 모듈 2: Notion - Create a Database Item

- Database: Leads
- 필드 매핑:

| Notion 필드 | Webhook 데이터 |
|-------------|---------------|
| Name | `{{name}}` |
| Email | `{{email}}` |
| Phone | `{{phone}}` |
| Company | `{{company}}` |
| Source | `{{source}}` |
| Type | `{{type}}` |
| Status | `new` (고정) |
| Notes | `{{message}}` |

### 모듈 3: Telegram Bot - Send a Text Message

- Chat ID: 본인 Chat ID
- 메시지:

```
[신규 리드] {{name}}
이메일: {{email}}
회사: {{company}}
유형: {{type}}
메시지: {{message}}
Notion: (Notion Create Page의 URL)
```

---

## 시나리오 2: 팔로업 리마인더

**목적:** 팔로업 예정일이 된 리드를 매일 아침 Telegram으로 알림

**자동화 레벨:** 완전 자동

**Make.com 설정:**

```
[Schedule] → [Notion: Search] → [Iterator] → [Telegram: Send Message]
```

### 모듈 1: Schedule

- 매일 08:00 (KST)
- 주말 제외 가능

### 모듈 2: Notion - Search Objects

- Database: Leads
- Filter:
  - Follow-up Date = Today
  - Status != won, lost

### 모듈 3: Flow Control - Iterator

- 검색 결과를 개별 항목으로 분리

### 모듈 4: Telegram Bot - Send a Text Message

```
[팔로업 필요] {{Name}}
회사: {{Company}}
이메일: {{Email}}
상태: {{Status}}
메모: {{Notes}}
Notion: {{URL}}
```

---

## 시나리오 3: 콘텐츠 파이프라인

**목적:** 콘텐츠 아이디어를 등록하면 AI가 초안을 작성하고 검토 요청

**자동화 레벨:** AI 생성 → 사람 승인

**Make.com 설정:**

```
[Notion: Watch Items] → [OpenAI: Create Completion] → [Notion: Update Page] → [Telegram: Send Message]
```

### 모듈 1: Notion - Watch Database Items

- Database: Content Pipeline
- Filter: Status = idea
- 폴링 간격: 15분

### 모듈 2: OpenAI - Create a Completion

- Model: gpt-4o
- Prompt:

```
아래 주제에 대한 {{Type}} 콘텐츠 초안을 작성해주세요.

제목: {{Title}}
플랫폼: {{Platform}}
키워드: {{Keywords}}

요구사항:
- {{Platform}}에 적합한 톤과 길이
- SEO 키워드 자연스럽게 포함
- blog: 1000-1500자, sns-post: 200-300자, newsletter: 500-800자
```

### 모듈 3: Notion - Update a Page

- Page ID: 모듈 1의 Page ID
- 업데이트할 필드:
  - Draft: OpenAI 응답 내용
  - Status: `review`

### 모듈 4: Telegram Bot - Send a Text Message

```
[콘텐츠 검토 필요] {{Title}}
유형: {{Type}}
플랫폼: {{Platform}}
상태: idea → review
Notion: {{URL}}
AI가 초안을 작성했습니다. 검토해주세요.
```

---

## Notion DB 참조 정보

| DB | Notion URL | Data Source ID |
|----|-----------|----------------|
| Leads | https://www.notion.so/49f20ec826ed41ff9f5cc137af122ed3 | b398c377-2698-4624-9b00-2a1926f9c790 |
| Proposals | https://www.notion.so/3e86d8a31dfd4fc5829a6ed4482b3708 | 7371c260-62c2-46bd-92bc-8e957abbb80d |
| Content Pipeline | https://www.notion.so/69a39bc304664c658ec1b1fb7117c100 | 4be97f57-aec9-47bd-b616-105f4bcd3c96 |
| Work Inbox | https://www.notion.so/7f471e8dcba44e878b96cfae8d0de083 | fd4a1b17-f297-409b-81e7-8d9e40c45257 |

## Vercel 웹훅 엔드포인트

| 엔드포인트 | 용도 | 메서드 |
|-----------|------|--------|
| `/api/webhook/contact` | 문의폼 → Make.com 리드 캡처 | POST |
| `/api/webhook/inquiry` | 고객 문의 → Make.com 리드 캡처 | POST |

### 요청 형식

```json
{
  "name": "홍길동",
  "email": "hong@example.com",
  "phone": "010-1234-5678",
  "company": "ABC Corp",
  "message": "문의 내용",
  "source": "website",
  "type": "b2b"
}
```

### 응답

```json
{ "success": true, "message": "Contact received" }
```

---

## 자동 응답 이메일 설정

리드 캡처 시나리오에 Gmail 모듈을 추가하면 문의자에게 자동 확인 이메일을 발송할 수 있습니다.

### Make.com 설정

기존 시나리오 (Webhook → Notion → Telegram)에 모듈 추가:

```
Webhook → Notion → Telegram
                 → Gmail (자동 응답)   ← 병렬 추가
```

Telegram 모듈 뒤에 `+` → **Gmail** → **Send an Email** 선택

### Gmail 모듈 설정

| 필드 | 값 |
|------|-----|
| To | webhook의 `email` (패널에서 클릭) |
| Subject | `문의가 접수되었습니다 - [회사명]` |
| Content Type | HTML |
| Content | 아래 템플릿 |

### 이메일 템플릿

```html
<div style="font-family: sans-serif; max-width: 600px; margin: 0 auto;">
  <h2>문의가 접수되었습니다</h2>
  <p>안녕하세요, {{name}}님.</p>
  <p>문의 내용을 확인했습니다. 담당자가 영업일 기준 1일 이내에 연락드리겠습니다.</p>
  <hr style="border: none; border-top: 1px solid #eee; margin: 20px 0;" />
  <p style="color: #666; font-size: 14px;">
    <strong>접수 내용</strong><br />
    이름: {{name}}<br />
    이메일: {{email}}<br />
    메시지: {{message}}
  </p>
  <hr style="border: none; border-top: 1px solid #eee; margin: 20px 0;" />
  <p style="color: #999; font-size: 12px;">본 메일은 자동 발송되었습니다.</p>
</div>
```

`{{name}}`, `{{email}}`, `{{message}}`는 Make.com 패널에서 webhook 변수를 클릭하여 삽입하세요.

---

## 보안 설정 (Webhook API Key)

외부에서 webhook URL을 악용하는 것을 방지하기 위해 API Key 인증을 사용할 수 있습니다.

### Vercel 환경변수

```
WEBHOOK_API_KEY=your-secret-key-here
ALLOWED_ORIGINS=https://webscout-next.vercel.app,https://timbel.com
```

### API Key 포함 호출

```
POST /api/webhook/contact
Header: X-API-Key: your-secret-key-here
```

또는 쿼리 파라미터:
```
POST /api/webhook/contact?key=your-secret-key-here
```

API Key가 환경변수에 설정되지 않으면 인증을 건너뜁니다 (개발 편의).

### Rate Limiting

IP당 분당 10건으로 제한됩니다. 초과 시 `429 Too Many Requests` 응답.
