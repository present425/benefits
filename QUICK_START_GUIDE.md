# 🚀 혜택 통합 관리 시스템 - 빠른 시작 가이드

> **목표**: 30분 안에 MVP 시스템 구동하기

---

## 📦 제공된 파일 목록

### 워크플로우 (n8n)
- `5_benefits_data_collector.json` - 혜택 데이터 수집 크롤러
- `6_benefits_calendar_api.json` - 혜택 캘린더 API (MVP 핵심)
- `7_ai_recommendation_api.json` - AI 추천 API (선택)

### 샘플 데이터
- `benefits_sample_data.csv` - 13개 혜택 샘플
- `user_payments_sample_data.csv` - 10개 결제 이력 샘플

### 문서
- `README_BENEFITS_SYSTEM.md` - 전체 시스템 상세 가이드
- `SETUP_GOOGLE_SHEETS.md` - Google Sheets 설정 가이드
- `API_TEST_EXAMPLES.md` - API 테스트 예시
- `ARCHITECTURE_DIAGRAM.md` - 시스템 아키텍처
- `QUICK_START_GUIDE.md` - 이 문서

---

## ⚡ 30분 빠른 시작

### 1단계: Google Sheets 준비 (10분)

#### 1.1 스프레드시트 생성
1. [Google Sheets](https://sheets.google.com) 접속
2. 빈 스프레드시트 만들기
3. 이름: **"혜택 통합 관리 시스템 DB"**

#### 1.2 데이터 임포트
**첫 번째 시트 (Benefits):**
1. 기본 시트 이름을 `Benefits`로 변경
2. File → Import → Upload → `benefits_sample_data.csv` 선택
3. Import location: **Replace current sheet**
4. ✅ 완료: 13행 데이터 확인

**두 번째 시트 (UserPayments):**
1. 하단 `+` 버튼으로 시트 추가
2. 시트 이름: `UserPayments`
3. File → Import → Upload → `user_payments_sample_data.csv` 선택
4. Import location: **Replace current sheet**
5. ✅ 완료: 10행 데이터 확인

#### 1.3 스프레드시트 ID 복사
URL에서 ID 복사:
```
https://docs.google.com/spreadsheets/d/1bxVmRx_dIaQVIxkFEGGEErWDGwJ-XckETNNV6rLlRps/edit
```

**예시:**
```
https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit
                                      ↑ 이 부분이 ID ↑
```

💾 **메모장에 저장!**

---

### 2단계: Google API 설정 (5분)

#### 2.1 프로젝트 생성
1. [Google Cloud Console](https://console.cloud.google.com) 접속
2. 상단 프로젝트 선택 → **새 프로젝트**
3. 프로젝트 이름: `Benefits System`
4. **만들기** 클릭

#### 2.2 API 활성화
1. **API 및 서비스** → **라이브러리**
2. 검색: "Google Sheets API"
3. **사용 설정** 클릭

#### 2.3 OAuth 인증 정보
1. **API 및 서비스** → **사용자 인증 정보**
2. **사용자 인증 정보 만들기** → **OAuth 클라이언트 ID**
3. 동의 화면 구성 (처음이라면):
   - User Type: **외부**
   - 앱 이름: `Benefits System`
   - 나머지 기본값 → 저장
4. 애플리케이션 유형: **데스크톱 앱**
5. 이름: `n8n Benefits`
6. **만들기** → **JSON 다운로드**

💾 **credentials.json 파일 저장!**

---

### 3단계: n8n 실행 및 설정 (10분)

#### 3.1 n8n 시작
```bash
cd week9
docker-compose up -d
```

**접속**: http://localhost:5678

#### 3.2 Google Sheets Credential 등록
1. n8n 좌측 메뉴 → **Credentials** → **Add Credential**
2. 검색: "Google Sheets"
3. 타입: **Google Sheets OAuth2 API** 선택
4. 이름: `Benefits System`
5. credentials.json에서 **Client ID**와 **Client Secret** 복사
6. **OAuth Redirect URL** 복사 (예: `http://localhost:5678/rest/oauth2-credential/callback`)
7. Google Cloud Console로 돌아가서:
   - 방금 만든 OAuth 클라이언트 편집
   - **승인된 리디렉션 URI** 추가 → 복사한 URL 붙여넣기
   - **저장**
8. n8n으로 돌아와서 **Connect my account** 클릭
9. Google 계정 선택 → 권한 승인
10. ✅ 완료: "Connected" 표시 확인

#### 3.3 워크플로우 임포트

**워크플로우 2 임포트 (캘린더 API - MVP):**
1. 좌측 메뉴 → **Workflows**
2. 우측 상단 **...** → **Import from File**
3. `6_benefits_calendar_api.json` 선택
4. 임포트 완료 후 워크플로우 열기

**Google Sheets 노드 설정:**
1. "Google Sheets: Read Benefits" 노드 클릭
2. **Credential**: `Benefits System` 선택
3. **Document** 클릭 → **From list** 선택 → 방금 만든 스프레드시트 선택
   - 또는 **By ID** → 1단계에서 복사한 ID 붙여넣기
4. **Sheet**: `Benefits` 선택
5. **Save** 클릭 (우측 상단)

**워크플로우 활성화:**
1. 우측 상단 **Active** 토글 ON
2. ✅ 완료: "Workflow is now active" 메시지 확인

**Webhook URL 확인:**
1. "Webhook: GET /benefits/calendar" 노드 클릭
2. **Production URL** 복사
   - 예: `http://localhost:5678/webhook/benefits/calendar`

💾 **이 URL을 메모!**

---

### 4단계: API 테스트 (5분)

#### 테스트 1: 기본 캘린더 조회

**Windows (PowerShell):**
```powershell
Invoke-RestMethod -Uri "http://localhost:5678/webhook/benefits/calendar?period=week" -Method Get | ConvertTo-Json -Depth 10
```

**Mac/Linux (Terminal):**
```bash
curl "http://localhost:5678/webhook/benefits/calendar?period=week"
```

**브라우저:**
```
http://localhost:5678/webhook/benefits/calendar?period=week
```

#### 예상 결과
```json
{
  "meta": {
    "period": "week",
    "totalBenefits": 10,
    "generatedAt": "2025-10-13T..."
  },
  "calendar": [
    {
      "date": "2025-10-13",
      "dayOfWeek": "월",
      "benefitCount": 5,
      "benefits": [...]
    }
  ],
  "summary": {
    "byProvider": {
      "KT": 4,
      "BC카드": 3
    }
  }
}
```

✅ **성공!** 이제 MVP가 작동합니다!

#### 테스트 2: 카테고리 필터링
```
http://localhost:5678/webhook/benefits/calendar?period=week&category=쇼핑
```

#### 테스트 3: 월간 조회
```
http://localhost:5678/webhook/benefits/calendar?period=month
```

---

## 🎯 다음 단계 (선택)

### 옵션 A: 크롤링 워크플로우 추가
1. `5_benefits_data_collector.json` 임포트
2. Google Sheets 노드 설정 (동일한 Credential 사용)
3. 수동 실행으로 테스트
4. 실제 웹사이트 HTML 구조에 맞게 파싱 로직 수정

### 옵션 B: AI 추천 API 추가
1. [OpenAI API Key](https://platform.openai.com/api-keys) 발급
2. n8n에 OpenAI Credential 등록
3. `7_ai_recommendation_api.json` 임포트
4. Google Sheets + OpenAI 노드 설정
5. POST 요청으로 테스트

### 옵션 C: 프론트엔드 연동
API를 사용하는 간단한 웹 페이지 제작:
```html
<!DOCTYPE html>
<html>
<head>
  <title>혜택 캘린더</title>
</head>
<body>
  <h1>이번 주 혜택</h1>
  <div id="calendar"></div>
  
  <script>
    fetch('http://localhost:5678/webhook/benefits/calendar?period=week')
      .then(res => res.json())
      .then(data => {
        const html = data.calendar.map(day => `
          <h2>${day.date} (${day.dayOfWeek})</h2>
          <ul>
            ${day.benefits.map(b => `
              <li>
                ${b.urgencyTag || ''} ${b.name} 
                - ${b.value.toLocaleString()}원
              </li>
            `).join('')}
          </ul>
        `).join('');
        document.getElementById('calendar').innerHTML = html;
      });
  </script>
</body>
</html>
```

---

## 🐛 문제 해결 (자주 발생하는 이슈)

### "Webhook not found" 에러
**원인**: 워크플로우가 비활성화되어 있음
**해결**: 워크플로우 우측 상단 Active 토글 ON

### Google Sheets 연결 실패
**원인**: Credential 설정 오류
**해결**:
1. Credential 재생성
2. OAuth 리디렉션 URI 확인
3. 브라우저 쿠키/캐시 삭제 후 재인증

### 빈 결과 반환 (totalBenefits: 0)
**원인**: 날짜 필터링 또는 데이터 형식 문제
**해결**:
1. Google Sheets에서 데이터 확인
2. StartDate, EndDate 형식 확인 (YYYY-MM-DD)
3. Status 컬럼이 'active' 또는 'scheduled'인지 확인

### n8n이 실행되지 않음
```bash
# 상태 확인
docker-compose ps

# 재시작
docker-compose restart n8n

# 로그 확인
docker-compose logs -f n8n
```

---

## 📚 추가 학습 자료

### 핵심 문서
1. **README_BENEFITS_SYSTEM.md** - 전체 시스템 상세 설명
2. **ARCHITECTURE_DIAGRAM.md** - 시스템 구조 이해
3. **API_TEST_EXAMPLES.md** - API 활용 예시

### n8n 학습
- [n8n 공식 문서](https://docs.n8n.io/)
- [n8n Community](https://community.n8n.io/)
- [n8n YouTube](https://www.youtube.com/c/n8n-io)

### Google Sheets API
- [Google Sheets API 문서](https://developers.google.com/sheets/api)
- [OAuth 2.0 가이드](https://developers.google.com/identity/protocols/oauth2)

---

## 🎉 축하합니다!

혜택 통합 관리 시스템 MVP를 성공적으로 구축했습니다!

### 지금까지 만든 것:
- ✅ Google Sheets 기반 데이터베이스
- ✅ 혜택 캘린더 REST API
- ✅ 주간/월간 조회 기능
- ✅ 카테고리 필터링
- ✅ 긴급도 자동 태그
- ✅ 통계 요약 기능

### 이제 할 수 있는 것:
- 🚀 웹/앱에서 API 호출
- 📊 혜택 데이터 시각화
- 🤖 AI 추천 기능 추가 (옵션)
- 🔔 알림 시스템 구축 (향후)
- 🌐 프로덕션 배포 (향후)

---

## 💡 팁

### 개발 효율 높이기
1. **n8n Workflow 복사**: 노드 우클릭 → Duplicate
2. **테스트 데이터 고정**: Pin Data 기능 활용
3. **에러 디버깅**: 각 노드별 Execute Node로 단계별 테스트

### 데이터 관리
1. Google Sheets에서 직접 수정 → 즉시 API 반영
2. 새 혜택 추가: 마지막 행에 추가하면 자동 포함
3. 혜택 마감: Status를 'expired'로 변경

### 성능 최적화
1. 캐싱 활용 (현재 5분)
2. 필요한 컬럼만 조회
3. 불필요한 데이터 정기 삭제

---

**문서 버전**: 1.0
**최종 업데이트**: 2025-10-13

**도움이 필요하신가요?** README_BENEFITS_SYSTEM.md의 문제 해결 섹션을 참고하세요!

