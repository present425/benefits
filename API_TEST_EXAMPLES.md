# 🧪 API 테스트 예시

## 혜택 캘린더 API 테스트

### 기본 요청

#### 1. 주간 캘린더 조회 (기본)
```bash
curl -X GET "http://localhost:5678/webhook/benefits/calendar?period=week"
```

#### 2. 월간 캘린더 조회
```bash
curl -X GET "http://localhost:5678/webhook/benefits/calendar?period=month"
```

#### 3. 카테고리 필터 - 쇼핑
```bash
curl -X GET "http://localhost:5678/webhook/benefits/calendar?period=week&category=쇼핑"
```

#### 4. 카테고리 필터 - 카페
```bash
curl -X GET "http://localhost:5678/webhook/benefits/calendar?period=week&category=카페"
```

#### 5. 특정 날짜부터 조회
```bash
curl -X GET "http://localhost:5678/webhook/benefits/calendar?period=week&startDate=2025-10-15"
```

#### 6. 복합 필터 (월간 + 쇼핑)
```bash
curl -X GET "http://localhost:5678/webhook/benefits/calendar?period=month&category=쇼핑"
```

---

## PowerShell용 명령어 (Windows)

### 주간 캘린더 조회
```powershell
Invoke-RestMethod -Uri "http://localhost:5678/webhook/benefits/calendar?period=week" -Method Get | ConvertTo-Json -Depth 10
```

### 월간 캘린더 조회
```powershell
Invoke-RestMethod -Uri "http://localhost:5678/webhook/benefits/calendar?period=month" -Method Get | ConvertTo-Json -Depth 10
```

### 카테고리 필터링
```powershell
$uri = "http://localhost:5678/webhook/benefits/calendar?period=week&category=쇼핑"
Invoke-RestMethod -Uri $uri -Method Get | ConvertTo-Json -Depth 10
```

---

## Postman/Insomnia 설정

### Collection 구성

**Environment Variables:**
```
base_url: http://localhost:5678
webhook_path: /webhook/benefits
```

**Request 1: 주간 캘린더**
```
Method: GET
URL: {{base_url}}{{webhook_path}}/calendar
Params:
  - period: week
```

**Request 2: 월간 캘린더 (쇼핑)**
```
Method: GET
URL: {{base_url}}{{webhook_path}}/calendar
Params:
  - period: month
  - category: 쇼핑
```

---

## 예상 응답 구조

### 성공 응답 (200 OK)

```json
{
  "meta": {
    "period": "week",
    "startDate": "2025-10-13",
    "endDate": "2025-10-20",
    "totalBenefits": 10,
    "category": "all",
    "generatedAt": "2025-10-13T12:34:56.789Z"
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
        },
        {
          "id": "B004",
          "provider": "BC카드",
          "name": "BC카드 넷플릭스 3개월 무료",
          "category": "OTT",
          "startDate": "2025-10-13",
          "endDate": "2025-12-31",
          "openTime": "00:00",
          "closeTime": "23:59",
          "conditions": "신규가입자 한정",
          "deepLink": "https://netflix.com/bccard-promo",
          "description": "넷플릭스 스탠다드 요금제 3개월 무료",
          "value": 45000,
          "status": "active",
          "urgencyTag": null,
          "hoursRemaining": 1896
        }
      ]
    },
    {
      "date": "2025-10-14",
      "dayOfWeek": "화",
      "benefitCount": 2,
      "benefits": [...]
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
      "영화": 1,
      "OTT": 1
    },
    "totalValue": 248000,
    "urgentBenefits": 2
  }
}
```

### 에러 응답 (500 Internal Server Error)

```json
{
  "error": "Internal Server Error",
  "message": "Failed to read Google Sheets",
  "timestamp": "2025-10-13T12:34:56.789Z"
}
```

---

## AI 추천 API 테스트

### 기본 추천 요청

```bash
curl -X POST http://localhost:5678/webhook/benefits/recommend \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "U001",
    "context": "shopping"
  }'
```

### 위치 정보 포함

```bash
curl -X POST http://localhost:5678/webhook/benefits/recommend \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "U001",
    "location": {
      "lat": 37.5665,
      "lng": 126.9780
    },
    "context": "lunch"
  }'
```

### PowerShell 버전

```powershell
$body = @{
    userId = "U001"
    context = "shopping"
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:5678/webhook/benefits/recommend" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body | ConvertTo-Json -Depth 10
```

---

## 응답 데이터 활용 예시

### JavaScript (프론트엔드)

```javascript
// 캘린더 데이터 가져오기
async function fetchBenefitsCalendar(period = 'week', category = null) {
  const params = new URLSearchParams({ period });
  if (category) params.append('category', category);
  
  const response = await fetch(
    `http://localhost:5678/webhook/benefits/calendar?${params}`
  );
  const data = await response.json();
  
  return data;
}

// 사용 예시
fetchBenefitsCalendar('week', '쇼핑').then(data => {
  console.log(`총 ${data.meta.totalBenefits}개 혜택 발견`);
  
  data.calendar.forEach(day => {
    console.log(`\n[${day.date} ${day.dayOfWeek}] ${day.benefitCount}개`);
    
    day.benefits.forEach(benefit => {
      console.log(`  ${benefit.urgencyTag || ''} ${benefit.name}`);
      console.log(`  💰 ${benefit.value.toLocaleString()}원 혜택`);
      console.log(`  🔗 ${benefit.deepLink}`);
    });
  });
});
```

### Python (데이터 분석)

```python
import requests
import pandas as pd

# API 호출
response = requests.get(
    'http://localhost:5678/webhook/benefits/calendar',
    params={'period': 'month'}
)
data = response.json()

# 데이터프레임 변환
benefits = []
for day in data['calendar']:
    for benefit in day['benefits']:
        benefits.append({
            'date': day['date'],
            'provider': benefit['provider'],
            'name': benefit['name'],
            'category': benefit['category'],
            'value': benefit['value'],
            'urgency': benefit['urgencyTag']
        })

df = pd.DataFrame(benefits)

# 분석
print("제공사별 혜택 수:")
print(df['provider'].value_counts())

print("\n카테고리별 평균 혜택 가치:")
print(df.groupby('category')['value'].mean().sort_values(ascending=False))

print("\n긴급 혜택:")
print(df[df['urgency'].notna()][['name', 'urgency', 'value']])
```

---

## 테스트 체크리스트

### 혜택 캘린더 API

- [ ] 주간 캘린더 조회 성공
- [ ] 월간 캘린더 조회 성공
- [ ] 카테고리 필터링 동작 확인
- [ ] 날짜 범위 정확성 검증
- [ ] 긴급도 태그 올바르게 표시
- [ ] 통계 데이터 정확성 확인
- [ ] 빈 결과 처리 (필터 조건 충족 혜택 없음)
- [ ] 캐싱 헤더 확인 (Cache-Control)

### AI 추천 API

- [ ] 기본 추천 요청 성공
- [ ] 사용자별 맞춤 추천 동작
- [ ] 결제 이력 기반 분석 작동
- [ ] TOP 3 추천 생성 확인
- [ ] 대체 플랜 제공 확인
- [ ] OpenAI 토큰 사용량 적정 수준
- [ ] 에러 핸들링 (존재하지 않는 userId)

### 성능 테스트

- [ ] 응답 시간 < 2초 (캘린더 API)
- [ ] 응답 시간 < 5초 (AI 추천 API)
- [ ] 동시 요청 10개 처리 가능
- [ ] Google Sheets 읽기 성능 양호

---

## 문제 해결

### "Webhook not found" 에러

**원인**: 워크플로우가 비활성화되어 있음

**해결**: n8n에서 워크플로우 활성화 (우측 상단 토글 ON)

### "Connection refused" 에러

**원인**: n8n 서버가 실행되지 않음

**해결**:
```bash
cd week9
docker-compose up -d
docker-compose ps  # n8n 상태 확인
```

### 빈 응답 반환 (totalBenefits: 0)

**원인**: Google Sheets에 데이터 없음 또는 날짜 형식 오류

**해결**:
1. Google Sheets 확인
2. 날짜 형식 YYYY-MM-DD 확인
3. Status 컬럼 값 확인 (active/scheduled)

---

## 다음 단계

테스트 성공 후:
1. 프론트엔드 연동
2. 실제 크롤링 로직 구현
3. 알림 기능 추가
4. 프로덕션 배포 준비

