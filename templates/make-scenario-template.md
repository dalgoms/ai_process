# Make.com Scenario Template

새 자동화 시나리오를 설계할 때 사용하는 범용 템플릿.
아래 양식을 복사하여 구체적인 시나리오로 채우세요.

---

## 시나리오 설계 양식

### 기본 정보

| 항목 | 값 |
|------|-----|
| 시나리오 이름 | |
| 목적 | |
| 자동화 레벨 | 완전 자동 / AI+사람 승인 / 사람 트리거 |
| 트리거 | Webhook / Schedule / Watch |
| 실행 빈도 | 즉시 / 매시간 / 매일 / 매주 |

### 모듈 구성

```
[트리거] → [처리] → [저장] → [알림]
```

| 순서 | 모듈 | 설정 |
|------|------|------|
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |

### 데이터 매핑

| 입력 필드 | 출력 필드 | 변환 |
|----------|----------|------|
| | | |

### 에러 처리

| 에러 유형 | 대응 |
|----------|------|
| 모듈 실패 | |
| 데이터 누락 | |
| API 한도 초과 | |

### 테스트 체크리스트

- [ ] 정상 데이터 테스트
- [ ] 필수 필드 누락 테스트
- [ ] 에러 시 알림 확인
- [ ] 시나리오 ON 상태 확인

---

## 자주 사용하는 시나리오 패턴

### 패턴 1: 리드 캡처

```
Webhook → Notion Create Item → Telegram + Gmail
```

| 모듈 | 설정 |
|------|------|
| Webhooks - Custom webhook | 데이터 구조: name, email, phone, company, message, source, type |
| Notion - Create a Database Item | Database: Leads DB, 필드 매핑 |
| Telegram Bot - Send a Text Message | Chat ID, 메시지 템플릿 |
| Gmail - Send an Email | To: webhook email, 자동 응답 |

### 패턴 2: 스케줄 리마인더

```
Schedule → Notion Search → Iterator → Telegram
```

| 모듈 | 설정 |
|------|------|
| Schedule | 매일 08:00 (KST) |
| Notion - Search Objects | Filter: Follow-up Date = Today |
| Flow Control - Iterator | 검색 결과 분리 |
| Telegram Bot - Send a Text Message | 개별 알림 발송 |

### 패턴 3: AI 콘텐츠 생성

```
Notion Watch → OpenAI → Notion Update → Telegram
```

| 모듈 | 설정 |
|------|------|
| Notion - Watch Database Items | Filter: Status = idea, 15분 간격 |
| OpenAI - Create a Completion | Model: gpt-4o, 프롬프트 템플릿 |
| Notion - Update a Page | Draft 필드 업데이트, Status = review |
| Telegram Bot - Send a Text Message | 검토 요청 알림 |

### 패턴 4: 데이터 동기화

```
Webhook → Router → [분기1: Notion] + [분기2: Google Sheets]
```

| 모듈 | 설정 |
|------|------|
| Webhooks - Custom webhook | 데이터 수신 |
| Flow Control - Router | 조건별 분기 |
| Notion - Create/Update | DB 저장 |
| Google Sheets - Add a Row | 스프레드시트 백업 |

### 패턴 5: 조건부 알림

```
Webhook → Router → [긴급: SMS] + [일반: Telegram] + [고객: Gmail]
```

| 모듈 | 설정 |
|------|------|
| Webhooks - Custom webhook | urgency 필드 포함 |
| Flow Control - Router | urgency = urgent → SMS, 나머지 → Telegram |
| Twilio - Create a Message | 긴급 SMS |
| Telegram Bot - Send a Text Message | 일반 알림 |
| Gmail - Send an Email | 고객 자동 응답 |

---

## Make.com 팁

### 에러 핸들링

- 각 모듈에 **Error Handler** 추가 가능
- Resume: 에러 무시하고 계속
- Rollback: 이전 모듈 결과 되돌림
- Break: 시나리오 중단 + 알림

### 비용 최적화

- 1 시나리오 실행 = 각 모듈 수 만큼 operations 소비
- 불필요한 모듈 최소화
- Filter를 앞쪽에 배치하여 불필요한 실행 방지
- Schedule 간격을 적절히 설정 (5분 → 15분 → 1시간)

### 디버깅

- **Run once**: 1회 실행으로 데이터 흐름 확인
- **History**: 실행 이력에서 각 모듈의 입출력 확인
- **Incomplete executions**: 실패한 실행 재시도 가능
