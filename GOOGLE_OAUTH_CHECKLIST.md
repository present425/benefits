# ✅ Google OAuth 설정 체크리스트

## 1단계: 웹 애플리케이션 유형 확인

### Google Cloud Console에서 확인:

1. https://console.cloud.google.com/apis/credentials 접속
2. "OAuth 2.0 클라이언트 ID" 섹션 확인
3. **웹 애플리케이션** 유형의 클라이언트 찾기 (데스크톱 앱 ❌)

---

## 2단계: 리디렉션 URI 정확히 확인

### 웹 애플리케이션 클라이언트 편집:

1. 클라이언트 이름 옆 **연필 아이콘(✏️)** 클릭
2. 아래로 스크롤하여 **"승인된 리디렉션 URI"** 섹션 찾기
3. 다음 URI가 **정확히** 있는지 확인:
   ```
   http://localhost:5678/rest/oauth2-credential/callback
   ```

### ⚠️ 주의사항:
- ✅ `http://` (https 아님!)
- ✅ `localhost` (127.0.0.1 아님!)
- ✅ `:5678` (포트 번호 포함)
- ✅ `/rest/oauth2-credential/callback` (정확한 경로)
- ❌ 끝에 `/` 없음
- ❌ 공백 없음

4. 없다면 **"+ URI 추가"** 클릭하여 추가
5. **"저장"** 클릭

---

## 3단계: OAuth 동의 화면 - 테스트 사용자 확인

1. 좌측 메뉴 → **"OAuth 동의 화면"** 클릭
2. 아래로 스크롤하여 **"테스트 사용자"** 섹션
3. 본인의 Gmail 주소가 추가되어 있는지 확인
4. 없다면:
   - **"+ ADD USERS"** 클릭
   - Gmail 주소 입력
   - **"저장"** 클릭

---

## 4단계: API 활성화 확인

1. 좌측 메뉴 → **"라이브러리"** 클릭
2. 검색: "Google Sheets API"
3. **"사용 설정됨"** 상태 확인
4. 검색: "Google Drive API"
5. **"사용 설정"** 클릭 (아직 안 되어있다면)

---

## 5단계: n8n Credential 재생성

### n8n에서 처음부터 다시:

1. **좌측 메뉴 → "Credentials"**
2. 기존 Google Sheets credential 있다면 **삭제**
3. **"Add Credential"** 클릭
4. **"Google Sheets OAuth2 API"** 선택
5. 정보 입력:
   ```
   Name: Benefits System New
   Client ID: [웹 애플리케이션의 Client ID]
   Client Secret: [웹 애플리케이션의 Client Secret]
   ```
6. **"Save"** 클릭
7. **잠깐!** "Connect my account" 클릭 전에:
   - 브라우저에서 Google 로그아웃
   - 시크릿 모드(Incognito) 창 열기
   - 시크릿 모드에서 http://localhost:5678 접속
   - 다시 로그인 후 "Connect my account" 클릭

---

## 🆘 여전히 안 되면?

### 캐시 완전 초기화:

1. **Google Cloud Console 변경 후 5분 대기**
2. **n8n Docker 재시작:**
   ```bash
   docker-compose restart n8n
   ```
3. **브라우저 캐시 완전 삭제:**
   - Chrome: Ctrl + Shift + Delete
   - "전체 기간" 선택
   - "쿠키 및 사이트 데이터" 체크
   - "캐시된 이미지 및 파일" 체크
   - "데이터 삭제"

4. **시크릿 모드에서 다시 시도**

---

## 💡 정확한 오류 메시지 확인

브라우저 F12 → Console 탭에서 오류 메시지 전체 복사하여 제공하면 더 정확한 진단 가능합니다.

