[README_BENEFITS_SYSTEM.md](https://github.com/user-attachments/files/23162202/README_BENEFITS_SYSTEM.md)
# 🎁 혜택 통합 관리 시스템 (Benefits Integration System)

## 📋 시스템 개요

통신사, 카드사, 가맹점의 흩어진 혜택 정보를 자동으로 수집하고, AI 기반 개인화 추천을 제공하며, 혜택 캘린더 API를 통해 원클릭 사용까지 연결하는 통합 서비스입니다.

### 🎯 핵심 가치

- **정보 통합**: KT, BC카드, G마켓, 넷플릭스 등 다양한 소스의 혜택을 한 곳에
- **자동화**: 매일 자동 크롤링으로 최신 정보 유지
- **AI 추천**: 사용자 결제 이력 기반 맞춤형 혜택 추천
- **실시간 알림**: 오픈/마감 임박 혜택 자동 알림

---

## 🏗️ 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────┐
│                  혜택 통합 관리 시스템                    │
└─────────────────────────────────────────────────────────┘

[데이터 소스]
  KT 멤버십 ──┐
  BC카드 ─────┤
  G마켓 ──────┼──→ [크롤링 워크플로우] ──→ [Google Sheets DB]
  넷플릭스 ───┘

[API 엔드포인트]
  GET /benefits/calendar ──→ 혜택 캘린더 조회
  POST /benefits/recommend ─→ AI 기반 개인화 추천

[향후 확장]
  알림 발송 워크플로우 (푸시, 카카오톡)
  사용자 대시보드
```

---

## 📦 구성 요소

### 워크플로우

1. **`5_benefits_data_collector.json`** - 혜택 데이터 수집 크롤러
   - 매일 새벽 5시 자동 실행
   - 4개 소스 병렬 크롤링 (KT, BC카드, G마켓, 넷플릭스)
   - Google Sheets에 자동 저장

2. **`6_benefits_calendar_api.json`** - 혜택 캘린더 API (MVP)
   - REST API 엔드포인트
   - 주간/월간 혜택 조회
   - 긴급도 태그 자동 생성
   - 카테고리 필터링 지원

3. **`7_ai_recommendation_api.json`** - AI 혜택 추천 API (향후 구현)
   - OpenAI GPT-4 기반 분석
   - 결제 이력 패턴 학습
   - TOP 3 추천 + 대체 플랜

### 데이터베이스

- **Google Sheets** (무료, 프로토타입 최적)
  - `Benefits` 시트: 혜택 마스터 데이터
  - `UserPayments` 시트: 사용자 결제 이력

### 샘플 데이터

- `benefits_sample_data.csv` - 13개 샘플 혜택
- `user_payments_sample_data.csv` - 10개 샘플 결제 이력

---

## 🚀 설치 및 설정 가이드

### 1단계: Google Sheets 설정

#### 1.1 스프레드시트 생성

1. [Google Sheets](https://sheets.google.com) 접속
2. 새 스프레드시트 생성: **"혜택 통합 관리 시스템 DB"**
3. 첫 번째 시트 이름: `Benefits`
4. 두 번째 시트 추가 후 이름: `UserPayments`

#### 1.2 샘플 데이터 임포트

**Benefits 시트:**
```
File → Import → Upload → benefits_sample_data.csv 선택
→ Replace current sheet
```

**UserPayments 시트:**
```
File → Import → Upload → user_payments_sample_data.csv 선택
→ Replace current sheet
```

#### 1.3 스프레드시트 ID 복사

URL에서 ID 추출:
```
https://docs.google.com/spreadsheets/d/[여기가 ID]/edit
```

예시:
```
https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit
→ ID: 1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms
```

**이 ID를 메모장에 저장하세요!** (나중에 n8n에서 사용)

---

### 2단계: Google Cloud 설정

#### 2.1 Google Sheets API 활성화

1. [Google Cloud Console](https://console.cloud.google.com) 접속
2. 프로젝트 생성 (예: "Benefits System")
3. **API 및 서비스** → **라이브러리** 이동
4. "Google Sheets API" 검색 → **사용 설정**

#### 2.2 OAuth 2.0 인증 정보 생성

1. **API 및 서비스** → **사용자 인증 정보**
2. **사용자 인증 정보 만들기** → **OAuth 클라이언트 ID**
3. 애플리케이션 유형: **데스크톱 앱**
4. 이름: "n8n Benefits System"
5. **만들기** 클릭
6. **JSON 다운로드** (credentials.json)

---

### 3단계: n8n 설정

#### 3.1 n8n 시작

```bash
cd week9
docker-compose up -d
```

n8n 접속: http://localhost:5678

#### 3.2 Google Sheets Credentials 등록

1. n8n 메뉴 → **Credentials** → **New**
2. 타입: **Google Sheets OAuth2 API**
3. 이름: "Benefits System Google Sheets"
4. **Client ID**와 **Client Secret** 입력 (credentials.json에서 복사)
5. **OAuth Redirect URL** 복사
6. Google Cloud Console로 돌아가서:
   - OAuth 클라이언트 편집
   - **승인된 리디렉션 URI**에 복사한 URL 추가
7. n8n으로 돌아와서 **Connect my account** 클릭
8. Google 계정 선택 및 권한 승인

#### 3.3 OpenAI Credentials 등록 (AI 추천용)

1. [OpenAI API Keys](https://platform.openai.com/api-keys)에서 API 키 생성
2. n8n에서 **Credentials** → **New**
3. 타입: **OpenAI API**
4. API Key 입력
5. **Save**

---

### 4단계: 워크플로우 임포트

#### 4.1 워크플로우 1: 혜택 데이터 수집 크롤러

1. n8n 메뉴 → **Workflows** → **Import from File**
2. `5_benefits_data_collector.json` 선택
3. 임포트 후 워크플로우 열기
4. **수정 필요한 노드:**

   **a) Google Sheets: Save Benefits 노드**
   - 클릭하여 설정 열기
   - **Credential**: 위에서 만든 "Benefits System Google Sheets" 선택
   - **Spreadsheet ID**: 1단계에서 복사한 ID 입력
   - **Sheet Name**: `Benefits` (이미 설정됨)
   - **Save** 클릭

   **b) 파싱 노드들 (Parse KT/BC/GMarket/Netflix Data)**
   - ⚠️ **중요**: 실제 HTML 구조를 확인한 후 파싱 로직 수정 필요
   - 현재는 샘플 데이터 반환 (테스트용)

5. **워크플로우 활성화**: 우측 상단 토글 ON
6. **수동 테스트**: **Execute Workflow** 클릭

#### 4.2 워크플로우 2: 혜택 캘린더 API (MVP)

1. `6_benefits_calendar_api.json` 임포트
2. **수정 필요한 노드:**

   **Google Sheets: Read Benefits 노드**
   - **Credential**: "Benefits System Google Sheets" 선택
   - **Spreadsheet ID**: 동일한 ID 입력
   - **Sheet Name**: `Benefits`
   - **Save**

3. **워크플로우 활성화**
4. **Webhook URL 확인**:
   - Webhook 노드 클릭
   - **Production URL** 복사 (예: `http://localhost:5678/webhook/benefits/calendar`)

#### 4.3 워크플로우 3: AI 추천 API (선택)

1. `7_ai_recommendation_api.json` 임포트
2. **수정 필요한 노드:**
   - Google Sheets 노드 2개 (UserPayments, Benefits)
   - OpenAI 노드: Credential 선택
3. 스프레드시트 ID 입력
4. **워크플로우 활성화**

---

## 🧪 테스트

### 테스트 1: 혜택 캘린더 API

#### 주간 캘린더 조회 (기본)

```bash
curl "http://localhost:5678/webhook/benefits/calendar?period=week"
```

**예상 응답:**
```json
{
  "meta": {
    "period": "week",
    "startDate": "2025-10-13",
    "endDate": "2025-10-20",
    "totalBenefits": 10,
    "category": "all",
    "generatedAt": "2025-10-13T12:00:00Z"
  },
  "calendar": [
    {
      "date": "2025-10-13",
      "dayOfWeek": "월",
      "benefitCount": 5,
      "benefits": [
        {
          "id": "B001",
          "provider": "KT",
          "name": "KT멤버십 스타벅스 50% 할인",
          "category": "카페",
          "startDate": "2025-10-13",
          "endDate": "2025-10-31",
          "openTime": "09:00",
          "closeTime": "23:59",
          "conditions": "VIP등급 이상",
          "deepLink": "kt://membership/benefits/starbucks",
          "description": "스타벅스 아메리카노 50% 할인 (1일 1회)",
          "value": 5000,
          "status": "active",
          "urgencyTag": "🆕 오늘오픈",
          "hoursRemaining": 264
        }
      ]
    }
  ],
  "summary": {
    "byProvider": {
      "KT": 4,
      "BC카드": 3,
      "G마켓": 2,
      "넷플릭스": 1
    },
    "byCategory": {
      "카페": 2,
      "쇼핑": 3,
      "영화": 1
    },
    "totalValue": 250000,
    "urgentBenefits": 2
  }
}
```

#### 월간 캘린더 조회

```bash
curl "http://localhost:5678/webhook/benefits/calendar?period=month"
```

#### 카테고리 필터링 (쇼핑만)

```bash
curl "http://localhost:5678/webhook/benefits/calendar?period=week&category=쇼핑"
```

#### 특정 날짜부터 조회

```bash
curl "http://localhost:5678/webhook/benefits/calendar?period=week&startDate=2025-10-15"
```

### 테스트 2: AI 추천 API (선택)

```bash
curl -X POST http://localhost:5678/webhook/benefits/recommend \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "U001",
    "context": "shopping"
  }'
```

**예상 응답:**
```json
{
  "userId": "U001",
  "context": "shopping",
  "recommendations": [
    {
      "rank": 1,
      "benefitId": "B002",
      "benefitName": "페이북 G마켓 최대 10만원 할인",
      "reason": "쇼핑 카테고리에서 최대 혜택이며, 과거 G마켓 이용 이력이 있음",
      "expectedValue": 100000,
      "actionGuide": "오늘 10시~14시 선착순, 10만원 이상 구매 필요"
    },
    {
      "rank": 2,
      "benefitId": "B010",
      "benefitName": "BC카드 이마트 10% 청구할인",
      "reason": "BC카드 보유 사용자이며, 월 1회 사용 가능한 안정적 혜택",
      "expectedValue": 50000,
      "actionGuide": "이마트 전점에서 BC카드 결제 시 자동 적용"
    }
  ],
  "alternativePlan": "1순위 혜택 마감 시 이마트 10% 할인 또는 G마켓 스마일클럽 무료배송 활용 권장",
  "insights": "주로 온라인 쇼핑을 선호하며, 평균 구매금액 7만원대로 중형 할인 혜택 최적",
  "generatedAt": "2025-10-13T12:00:00Z",
  "tokensUsed": 850
}
```

### 테스트 3: 데이터 수집 크롤러 (수동 실행)

1. n8n에서 워크플로우 1 열기
2. **Execute Workflow** 클릭
3. 각 노드 실행 결과 확인
4. Google Sheets에서 데이터 저장 확인

---

## 🔧 커스터마이징

### 웹 크롤링 로직 수정

현재 파싱 노드는 샘플 데이터를 반환합니다. 실제 HTML 구조를 분석하여 수정하세요.

#### 예시: KT 멤버십 파싱 (Parse KT Data 노드)

```javascript
const items = [];
const html = $input.item.json.data || '';

// Cheerio 또는 정규식으로 HTML 파싱
// 예시: 혜택 리스트가 <div class="benefit-item"> 안에 있다고 가정

const regex = /<div class="benefit-item">(.*?)<\/div>/gs;
const matches = [...html.matchAll(regex)];

matches.forEach(match => {
  const content = match[1];
  
  // 제목 추출
  const titleMatch = content.match(/<h3>(.*?)<\/h3>/);
  const title = titleMatch ? titleMatch[1] : '';
  
  // 날짜 추출
  const dateMatch = content.match(/(\d{4}-\d{2}-\d{2})/);
  const date = dateMatch ? dateMatch[1] : '';
  
  items.push({
    json: {
      id: `KT_${Date.now()}_${items.length}`,
      provider: 'KT',
      name: title,
      category: '카테고리 추출 로직',
      startDate: date,
      // ... 나머지 필드
    }
  });
});

return items;
```

### 스케줄 변경

**Daily Trigger 노드** 설정:
- 현재: `0 5 * * *` (매일 오전 5시)
- 변경 예시:
  - `0 */6 * * *` (6시간마다)
  - `0 8 * * 1` (매주 월요일 오전 8시)

### 알림 기능 추가

향후 구현할 푸시 알림 워크플로우:

1. **Schedule Trigger** - 30분마다 실행
2. **Google Sheets** - 오픈/마감 임박 혜택 조회
3. **Filter** - 알림 대상 필터링
4. **Firebase/OneSignal** - 푸시 발송

---

## 📊 데이터 구조

### Benefits 시트 컬럼

| 컬럼 | 타입 | 설명 | 예시 |
|------|------|------|------|
| ID | String | 혜택 고유 ID | B001 |
| Provider | String | 제공사 | KT, BC카드, G마켓, 넷플릭스 |
| Name | String | 혜택명 | KT멤버십 스타벅스 50% 할인 |
| Category | String | 카테고리 | 카페, 쇼핑, 영화, OTT 등 |
| StartDate | Date | 시작일 | 2025-10-13 |
| EndDate | Date | 종료일 | 2025-10-31 |
| OpenTime | Time | 오픈 시간 | 09:00 |
| CloseTime | Time | 마감 시간 | 23:59 |
| Conditions | String | 혜택 조건 | VIP등급 이상, 선착순 1000명 |
| DeepLink | String | 딥링크/URL | kt://membership/benefits/... |
| Description | String | 상세 설명 | 스타벅스 아메리카노 50% 할인... |
| Value | Number | 혜택 가치 (원) | 5000 |
| Status | String | 상태 | active, scheduled, expired |

### UserPayments 시트 컬럼

| 컬럼 | 타입 | 설명 |
|------|------|------|
| UserID | String | 사용자 ID |
| Date | Date | 결제일 |
| Category | String | 카테고리 |
| Amount | Number | 결제 금액 |
| Merchant | String | 가맹점명 |
| PaymentMethod | String | 결제 수단 |

---

## 🐛 문제 해결

### 일반적인 문제

#### 1. Google Sheets 연결 실패

**증상**: "Credentials not found" 에러

**해결:**
1. n8n Credentials에서 Google Sheets OAuth2 재설정
2. Google Cloud Console에서 리디렉션 URI 확인
3. 워크플로우 노드에서 Credential 재선택

#### 2. 웹 크롤링 실패 (HTTP 403/404)

**증상**: HTTP Request 노드에서 에러

**해결:**
1. URL 확인 (실제 접근 가능한지 브라우저에서 테스트)
2. User-Agent 헤더 추가:
   ```
   Headers:
   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
   ```
3. 타임아웃 시간 증가 (Options → Timeout: 60000)

#### 3. OpenAI API 할당량 초과

**증상**: "Rate limit exceeded" 에러

**해결:**
1. [OpenAI Usage](https://platform.openai.com/usage) 확인
2. 모델 변경: `gpt-4` → `gpt-3.5-turbo` (비용 절감)
3. 요청 빈도 줄이기

#### 4. 캘린더 API가 빈 결과 반환

**증상**: `totalBenefits: 0`

**확인 사항:**
1. Google Sheets에 샘플 데이터 있는지 확인
2. 날짜 형식 확인 (YYYY-MM-DD)
3. Status 컬럼이 `active` 또는 `scheduled`인지 확인
4. n8n 워크플로우 로그 확인 (Filter Benefits 노드)

---

## 🚀 향후 로드맵

### Phase 1: MVP (현재)
- [x] Google Sheets 데이터베이스
- [x] 혜택 데이터 수집 크롤러 (프레임워크)
- [x] 혜택 캘린더 API
- [x] AI 추천 API (프레임워크)

### Phase 2: 크롤링 고도화
- [ ] 실제 HTML 파싱 로직 구현
- [ ] 에러 핸들링 및 재시도 로직
- [ ] 중복 제거 알고리즘 개선
- [ ] 크롤링 성공률 모니터링

### Phase 3: 알림 시스템
- [ ] Firebase Cloud Messaging 연동
- [ ] 오픈/마감 임박 알림 워크플로우
- [ ] 사용자별 알림 설정 관리
- [ ] 카카오톡 알림톡 연동

### Phase 4: 프로덕션 전환
- [ ] PostgreSQL/MongoDB 마이그레이션
- [ ] 인증/인가 시스템 (JWT)
- [ ] API Rate Limiting
- [ ] 모니터링 및 로깅 (Grafana, Sentry)
- [ ] CI/CD 파이프라인

### Phase 5: 사용자 경험
- [ ] 웹/앱 프론트엔드
- [ ] 혜택 캘린더 UI
- [ ] 딥링크 원클릭 실행
- [ ] 사용 이력 대시보드

---

## 📈 성능 최적화

### 캐싱 전략

캘린더 API는 5분 캐싱 적용:
```
Cache-Control: public, max-age=300
```

변경 방법:
- Respond 노드 → Options → Response Headers
- `Cache-Control` 값 수정

### 데이터베이스 최적화

Google Sheets 한계:
- 최대 행: 약 40,000개
- 동시 접속: 제한적

**대용량 데이터 시 PostgreSQL 전환 권장**

---

## 🔐 보안 고려사항

### 1. API 키 관리

환경변수 사용 권장:
```bash
# docker-compose.yml 또는 .env
OPENAI_API_KEY=sk-...
GOOGLE_CREDENTIALS=...
```

### 2. Webhook 보안

Webhook 노드에 인증 추가:
- Header Authentication
- Query Parameter Token
- API Key 검증

### 3. Rate Limiting

향후 구현:
- 사용자당 API 호출 제한 (10 req/min)
- IP 기반 제한
- n8n의 Rate Limit 노드 활용

---

## 📞 지원 및 문의

### 문서
- 본 README
- `SETUP_GOOGLE_SHEETS.md` - 상세 Sheets 설정 가이드
- 각 워크플로우 JSON 파일의 노드 Notes

### 디버깅 팁

1. **n8n 실행 로그 확인**
   ```bash
   docker-compose logs -f n8n
   ```

2. **워크플로우 단계별 실행**
   - 각 노드별로 "Execute Node" 클릭
   - JSON 출력 확인

3. **Google Sheets 수동 확인**
   - 데이터 형식 검증
   - 필터 및 조건부 서식 활용

---

## 📄 라이선스

이 프로젝트는 학습 및 프로토타입 목적으로 제작되었습니다.

---

## 🎉 완료!

이제 혜택 통합 관리 시스템의 MVP가 준비되었습니다!

**다음 단계:**
1. Google Sheets 샘플 데이터 확인
2. n8n에 워크플로우 임포트
3. 캘린더 API 테스트
4. 실제 크롤링 로직 구현 시작

**추천 실습 순서:**
1. 캘린더 API로 샘플 데이터 조회 (가장 빠른 성취감!)
2. Google Sheets 데이터 수동 추가/수정 테스트
3. 크롤링 워크플로우 1개 소스부터 구현
4. AI 추천 API 테스트 (OpenAI 크레딧 필요)

**Happy Building! 🚀**

