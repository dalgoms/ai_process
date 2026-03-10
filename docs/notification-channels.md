# 알림 채널 가이드

Telegram 외에 다른 채널로 알림을 받는 방법.
Make.com 시나리오에서 모듈만 추가/교체하면 됩니다.

---

## 현재 구성

```
Make.com → Telegram Bot (내부 알림)
```

## 확장 가능 구성

```
Make.com → Telegram (내부 실시간)
         → Teams / Slack (팀 채널)
         → Gmail (자동 응답)
         → 카카오 알림톡 (고객 알림)
         → SMS (긴급 알림)
```

모듈은 **병렬로 추가** 가능. 기존 Telegram을 제거할 필요 없음.

---

## 1. Microsoft Teams

**적합**: 팀 내부 알림 (영업팀 채널에 리드 도착 알림)

**비용**: 무료

### Make.com 설정

1. 시나리오 편집 → Telegram 모듈 뒤 `+`
2. **Microsoft Teams** → **Create a Message**
3. Connection: Microsoft 계정 OAuth 인증
4. Team: 대상 팀 선택
5. Channel: 알림 받을 채널 선택
6. Message: webhook 변수 매핑

### 대안: Incoming Webhook (간단)

1. Teams 채널 → 커넥터 → **Incoming Webhook** 추가
2. URL 복사
3. Make.com → **HTTP** → **Make a request**
   - URL: 복사한 webhook URL
   - Method: POST
   - Body:

```json
{
  "@type": "MessageCard",
  "summary": "신규 리드",
  "sections": [{
    "activityTitle": "[신규 리드] {{name}}",
    "facts": [
      {"name": "이메일", "value": "{{email}}"},
      {"name": "회사", "value": "{{company}}"},
      {"name": "메시지", "value": "{{message}}"}
    ]
  }]
}
```

---

## 2. Slack

**적합**: Slack을 사용하는 팀의 내부 알림

**비용**: 무료

### Make.com 설정

1. 시나리오 편집 → `+` → **Slack** → **Create a Message**
2. Connection: Slack OAuth 인증
3. Channel: 알림 받을 채널 선택 (예: `#leads`)
4. Text: webhook 변수 매핑

```
:bell: *신규 리드*
이름: {{name}}
이메일: {{email}}
회사: {{company}}
유형: {{type}}
메시지: {{message}}
```

---

## 3. 카카오 알림톡

**적합**: 고객(문의자)에게 접수 확인 메시지 발송

**비용**: 건당 약 8~15원

### 사전 준비

1. [카카오 비즈니스](https://business.kakao.com) 가입
2. 카카오톡 채널 개설
3. 알림톡 발신 프로필 등록
4. 메시지 템플릿 등록 + 심사 승인 (1~3일 소요)

### 템플릿 예시

```
[문의 접수 완료]

안녕하세요, #{이름}님.
문의가 정상 접수되었습니다.

■ 접수 내용: #{메시지}
■ 접수 시각: #{시각}

담당자가 영업일 기준 1일 이내에 연락드리겠습니다.
감사합니다.
```

### Make.com 설정

카카오 알림톡은 Make.com 기본 모듈이 없으므로 HTTP 모듈을 사용:

1. `+` → **HTTP** → **Make a request**
2. URL: 카카오 알림톡 API 엔드포인트 (또는 Solapi, NHN Cloud 등 중계 서비스)
3. Method: POST
4. Headers: `Authorization: Bearer {{API_KEY}}`
5. Body: 카카오 API 형식에 맞게 매핑

### 중계 서비스 추천

직접 카카오 API 연동이 복잡하면 아래 서비스를 사용:

| 서비스 | 장점 |
|--------|------|
| [Solapi](https://solapi.com) | 카카오 알림톡 + SMS 통합, 건당 9원 |
| [NHN Cloud](https://www.nhncloud.com) | 대량 발송에 유리, 기업용 |
| [알리고](https://smartsms.aligo.in) | 저렴, 소규모에 적합 |

---

## 4. SMS (문자)

**적합**: 긴급 알림, 카카오톡 미사용 고객

**비용**: 국내 건당 약 20원 (Twilio: $0.05)

### 방법 A: Twilio (글로벌)

1. [Twilio](https://twilio.com) 가입 + 전화번호 구매
2. Make.com → **Twilio** → **Create a Message**
3. From: Twilio 번호
4. To: 수신자 번호
5. Body: 알림 내용

### 방법 B: Solapi (국내)

1. [Solapi](https://solapi.com) 가입 + 발신번호 등록
2. Make.com → **HTTP** → **Make a request**
3. URL: `https://api.solapi.com/messages/v4/send`
4. Method: POST
5. Headers: Solapi API 인증
6. Body:

```json
{
  "message": {
    "to": "01012345678",
    "from": "발신번호",
    "text": "[신규 리드] {{name}} / {{email}} / {{company}}"
  }
}
```

---

## 5. 이메일 (Gmail)

**적합**: 고객 자동 응답, 내부 일일 리포트

**비용**: 무료

### Make.com 설정

1. `+` → **Gmail** → **Send an Email**
2. Connection: Gmail OAuth 인증
3. To: 수신자 이메일 (고객: webhook의 `email`, 내부: 고정 이메일)
4. Subject / Content 매핑

상세 설정: [make-scenarios.md](./make-scenarios.md) 의 "자동 응답 이메일 설정" 참조

---

## 추천 조합

### 1인 운영

```
Make.com → Telegram (모든 알림)
         → Gmail (고객 자동 응답)
```

### 소규모 팀 (2~5명)

```
Make.com → Teams 또는 Slack (팀 채널)
         → Gmail (고객 자동 응답)
         → 카카오 알림톡 (고객 접수 확인)
```

### B2B 서비스 제공 시

```
Make.com → Teams/Slack (고객사별 채널)
         → 카카오 알림톡 (최종 고객 알림)
         → SMS (긴급 알림)
         → Gmail (리포트)
```

---

## 채널별 비교

| 채널 | 비용 | 속도 | 적합 대상 | Make.com 모듈 |
|------|------|------|----------|--------------|
| Telegram | 무료 | 즉시 | 내부 (개인) | 기본 제공 |
| Teams | 무료 | 즉시 | 내부 (팀) | 기본 제공 |
| Slack | 무료 | 즉시 | 내부 (팀) | 기본 제공 |
| Gmail | 무료 | 1~2분 | 고객 응답, 리포트 | 기본 제공 |
| 카카오 알림톡 | 건당 ~10원 | 즉시 | 고객 알림 | HTTP 모듈 |
| SMS | 건당 ~20원 | 즉시 | 긴급, 고객 | Twilio 또는 HTTP |
