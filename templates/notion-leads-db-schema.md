# Notion Leads DB Schema Template

새 고객사 온보딩 시 Leads DB를 생성할 때 사용하는 범용 스키마.
업종에 맞게 커스텀 필드를 추가하세요.

---

## 기본 스키마

모든 업종에 공통으로 사용하는 필드.

| 필드 | 타입 | 용도 | 필수 |
|------|------|------|------|
| Name | title | 고객/회사명 | O |
| Email | email | 연락처 | O |
| Phone | phone_number | 전화번호 | |
| Company | text | 회사명 (B2B) | |
| Source | select | 유입 경로 | O |
| Type | select | b2b / b2c | O |
| Status | select | 진행 상태 | O |
| Score | number | 리드 점수 (1~100) | |
| Notes | text | 메모 | |
| Last Contact | date | 마지막 연락일 | |
| Follow-up Date | date | 다음 팔로업 예정일 | |
| Proposal | relation | 연결된 제안서 (Proposals DB) | |

### Source 옵션

```
website, referral, sns, ad, partner-page, app, other
```

### Status 옵션

```
new → contacted → proposal → negotiation → won / lost
```

---

## 업종별 커스텀 필드

### 헬스케어 (병원/진료)

| 필드 | 타입 | 용도 |
|------|------|------|
| Department | select | 진료과목 (내과, 피부과, 소아과 등) |
| Region | text | 소재지 |
| Doctor Count | number | 의사 수 |
| Monthly Cases | number | 월 진료 건수 |
| Has Telemedicine | checkbox | 기존 비대면 진료 경험 |

### 부동산

| 필드 | 타입 | 용도 |
|------|------|------|
| Property Type | select | 아파트, 오피스텔, 상가 등 |
| Desired Region | text | 희망 지역 |
| Budget | number | 예산 |
| Move-in Date | date | 입주 희망일 |

### 교육/학원

| 필드 | 타입 | 용도 |
|------|------|------|
| Subject | select | 수학, 영어, 코딩 등 |
| Grade | select | 초등, 중등, 고등, 성인 |
| Preferred Time | text | 상담 희망 시간 |

### 법률

| 필드 | 타입 | 용도 |
|------|------|------|
| Case Type | select | 민사, 형사, 이혼, 부동산 등 |
| Urgency | select | 긴급, 보통, 참고 |
| Court Date | date | 재판일 (있는 경우) |

### SaaS / IT

| 필드 | 타입 | 용도 |
|------|------|------|
| Company Size | select | 1~10, 11~50, 51~200, 200+ |
| Current Tool | text | 현재 사용 중인 솔루션 |
| Contract Value | number | 예상 계약 금액 |
| Decision Timeline | select | 즉시, 1개월, 3개월, 6개월 |

### 이커머스

| 필드 | 타입 | 용도 |
|------|------|------|
| Category | select | 문의 카테고리 (주문, 배송, 교환, 환불) |
| Order Number | text | 주문번호 |
| Product | text | 상품명 |

---

## Notion MCP로 생성하기

Cursor에서 Notion MCP를 사용하여 DB를 자동 생성하는 예시:

```
notion-create-database
  title: "Leads"
  schema: CREATE TABLE (
    "Name" TITLE,
    "Email" EMAIL,
    "Phone" PHONE_NUMBER,
    "Company" RICH_TEXT,
    "Source" SELECT('website':blue, 'referral':green, 'sns':pink, 'ad':orange, 'other':gray),
    "Type" SELECT('b2b':purple, 'b2c':blue),
    "Status" SELECT('new':gray, 'contacted':blue, 'proposal':yellow, 'negotiation':orange, 'won':green, 'lost':red),
    "Score" NUMBER,
    "Notes" RICH_TEXT,
    "Last Contact" DATE,
    "Follow-up Date" DATE
  )
```

업종별 커스텀 필드는 `notion-update-data-source`로 추가:

```
ALTER TABLE ADD COLUMN "Department" SELECT('내과':blue, '피부과':pink, '소아과':green)
```
