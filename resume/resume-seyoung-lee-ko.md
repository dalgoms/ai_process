# 이세영

## 요약

AI 기반 업무 자동화 아키텍처를 설계하고 구축하는 Growth Marketer.
Notion, GitHub Actions, GPT Codex, Make.com, Vercel을 연결한 엔드투엔드 자동화 파이프라인을 직접 설계·배포했습니다.
마케팅 실무 경험을 바탕으로 리드 확보부터 콘텐츠 생산, 알림 자동화까지 비즈니스 전 과정을 시스템화합니다.

---

## 핵심 역량

| 영역 | 내용 |
|------|------|
| Growth Marketing | SEO, 퍼포먼스 마케팅, 콘텐츠 마케팅, 전환율 최적화 |
| Product Marketing | 사용자 여정 설계, 랜딩페이지 UX, 서비스 메시지 전략 |
| AI 워크플로우 설계 | Notion → AI Agent → Deploy 엔드투엔드 파이프라인 |
| 비즈니스 자동화 | CRM 리드 캡처, 팔로업 리마인더, AI 콘텐츠 자동 생성 |
| No-Code / Low-Code | Make.com, Notion API, Vercel Serverless, GitHub Actions |
| 문서화 | 플레이북 기반 지식 체계, 재사용 가능한 템플릿 설계 |

---

## 경력

### timbel

**홍보마케팅팀 팀장** (Growth Marketing & Product Strategy)
2017.07 ~ 현재

속기 서비스 및 음성 기록 시장을 중심으로 B2B/B2C 마케팅 전략을 수립하고, 서비스 문의 리드 확보 및 매출 성장을 위한 퍼포먼스 마케팅과 콘텐츠 마케팅을 운영했습니다.

SEO, 콘텐츠 마케팅, 랜딩페이지 UX 개선 등을 통해 서비스 문의 유입 구조를 구축하고 지속적으로 개선하며 리드 증가 및 매출 성장에 기여했습니다.

마케팅 실행뿐만 아니라 제품 메시지와 사용자 여정을 고려한 Product Marketing 관점에서 랜딩페이지 구조와 서비스 메시지를 설계했습니다.

최근에는 Cursor, GPT, Notion, Make 등의 AI 도구를 활용해 마케팅 데이터 수집, 콘텐츠 생성, 리드 관리 등을 자동화하는 마케팅 운영 프로세스를 구축하며 업무 효율과 실행 속도를 개선하고 있습니다.

**주요 성과**

- 속기 서비스 관련 SEO 및 콘텐츠 마케팅을 통해 서비스 문의 리드 유입 채널 구축
- 랜딩페이지 및 마케팅 메시지 개선을 통한 문의 전환율 개선
- 퍼포먼스 및 콘텐츠 마케팅 운영을 통한 서비스 매출 성장 기여
- AI 기반 마케팅 자동화 도입을 통해 콘텐츠 제작 및 데이터 수집 업무 효율 개선

속기 서비스 마케팅 경험을 통해 음성 데이터를 기반으로 한 서비스 시장을 이해하게 되었으며, 이후 AI 음성 기술(STT)과 데이터 활용 영역에 대한 관심을 확장하게 되었습니다.

---

## 프로젝트

### AI Process Playbook
**[github.com/dalgoms/ai_process](https://github.com/dalgoms/ai_process)**

하나의 Notion Work Inbox에서 여러 프로젝트를 관리하는 중앙 자동화 허브.

- Notion → GitHub Issue → GPT Codex → PR → CI → Deploy 파이프라인
- `config/projects.json` 기반 멀티 프로젝트 라우팅
- Make.com 연동 리드 캡처 및 콘텐츠 자동화
- Telegram 실시간 알림 (파이프라인 전 단계)
- 아키텍처, 워크플로우, 트러블슈팅 문서화 완비

### WebScout
**[github.com/dalgoms/webscout-next](https://github.com/dalgoms/webscout-next)**

AI 기반 웹사이트 구조 분석기 (Next.js).

- URL 입력 → 사이트맵, SEO 신호, UX 인사이트 분석
- Vercel 자동 CI/CD 배포
- 문의폼 → Make.com 리드 캡처 파이프라인 연동
- Webhook 엔드포인트: API Key 인증, Rate Limiting, CORS 화이트리스트

### 비즈니스 자동화 시스템

마케팅/세일즈 전 과정을 자동화하는 비즈니스 레이어 구축.

- **리드 캡처**: 웹사이트 문의폼 → Make.com → Notion CRM → Telegram + Email
- **팔로업 리마인더**: 매일 스케줄 → 미연락 리드 알림 → 팀 노티
- **콘텐츠 파이프라인**: 아이디어 등록 → AI 초안 생성 → 사람 검토
- **멀티채널 알림**: Telegram, Email, Teams, Slack, 카카오톡, SMS

---

## 기술 스택

| 분류 | 기술 |
|------|------|
| AI / 자동화 | OpenAI GPT-4o, Codex Action, Make.com, GitHub Actions |
| 프론트엔드 | Next.js, React, Tailwind CSS, TypeScript |
| 백엔드 | Vercel Serverless Functions, Next.js API Routes |
| 데이터 | Notion API, Notion MCP |
| DevOps | Git, GitHub CLI, Vercel, CI/CD 파이프라인 |
| 커뮤니케이션 | Telegram Bot API, Gmail, KakaoTalk API |
| 도구 | Cursor IDE, AnyDesk, VS Code |

---

## 일하는 방식

1. **자동화 우선** — 반복되는 작업이면 파이프라인을 만든다
2. **문서화 철저** — 암묵지가 아닌 플레이북으로 남긴다
3. **빠르게 실행** — 완벽한 다음 달보다 작동하는 오늘이 낫다
4. **AI 활용** — 코드 생성, 콘텐츠 초안, 리드 스코어링에 AI를 쓴다
5. **모바일 워크플로우** — 폰에서 작업 등록, 어디서든 결과 확인

---

## 학력

아주대학교 미디어학과 졸업 (학사)

---

## 연락처

- GitHub: [github.com/dalgoms](https://github.com/dalgoms)
- Email: seyoung67@timbel.net
