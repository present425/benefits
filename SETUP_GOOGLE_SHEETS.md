# 📊 Google Sheets 설정 가이드

## 1. Google Sheets 생성

1. [Google Sheets](https://sheets.google.com) 접속
2. 새 스프레드시트 만들기
3. 이름: **"혜택 통합 관리 시스템 DB"**

## 2. 시트 구성

### Sheet 1: Benefits (혜택 마스터 데이터)

1. 첫 번째 시트 이름을 **"Benefits"**로 변경
2. `benefits_sample_data.csv` 파일 내용을 복사하여 붙여넣기
3. 헤더 행에 필터 적용 (데이터 > 필터 만들기)
4. 날짜 열 형식 지정 (StartDate, EndDate 열 선택 > 형식 > 숫자 > 날짜)

**컬럼 설명:**
- **ID**: 혜택 고유 ID (예: B001, B002...)
- **Provider**: 혜택 제공사 (KT, BC카드, G마켓, 넷플릭스)
- **Name**: 혜택명
- **Category**: 카테고리 (카페, 쇼핑, 영화, OTT, 통신, 배달, 편의점, 뷰티)
- **StartDate**: 시작일 (YYYY-MM-DD)
- **EndDate**: 종료일 (YYYY-MM-DD)
- **OpenTime**: 오픈 시간 (HH:MM)
- **CloseTime**: 마감 시간 (HH:MM)
- **Conditions**: 혜택 조건 (선착순, 등급 제한 등)
- **DeepLink**: 앱 딥링크 또는 웹 URL
- **Description**: 상세 설명
- **Value**: 혜택 가치 (원 단위, 할인/적립 금액)
- **Status**: 상태 (active, scheduled, expired)

### Sheet 2: UserPayments (사용자 결제 이력)

1. 시트 추가 (하단 + 버튼)
2. 시트 이름: **"UserPayments"**
3. `user_payments_sample_data.csv` 파일 내용을 복사하여 붙여넣기

**컬럼 설명:**
- **UserID**: 사용자 ID (예: U001, U002...)
- **Date**: 결제일 (YYYY-MM-DD)
- **Category**: 카테고리
- **Amount**: 결제 금액
- **Merchant**: 가맹점명
- **PaymentMethod**: 결제 수단

## 3. n8n 연동 준비

### Google Sheets API 활성화

1. [Google Cloud Console](https://console.cloud.google.com) 접속
2. 프로젝트 생성 또는 선택
3. "API 및 서비스" > "라이브러리" 이동
4. "Google Sheets API" 검색 및 활성화
5. "사용자 인증 정보" > "사용자 인증 정보 만들기"
   - 애플리케이션 유형: 데스크톱 앱
   - OAuth 2.0 클라이언트 ID 생성
6. JSON 다운로드 (나중에 n8n에서 사용)

### 스프레드시트 ID 복사

URL에서 스프레드시트 ID 복사:
```
https://docs.google.com/spreadsheets/d/[이 부분이 스프레드시트 ID]/edit
```

예시:
```
https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit
→ 스프레드시트 ID: 1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms
```

이 ID는 n8n 워크플로우에서 사용됩니다.

## 4. 데이터 업데이트 규칙

- **Status 값**:
  - `active`: 현재 사용 가능한 혜택
  - `scheduled`: 예정된 혜택
  - `expired`: 만료된 혜택

- **날짜 형식**: 반드시 `YYYY-MM-DD` 형식 사용
- **시간 형식**: 반드시 `HH:MM` 형식 사용 (24시간 형식)

## 5. 샘플 데이터 설명

총 13개의 샘플 혜택 데이터가 포함되어 있습니다:
- KT 멤버십 혜택: 5개
- BC카드 혜택: 5개
- G마켓 혜택: 2개
- 넷플릭스 혜택: 1개

카테고리별:
- 쇼핑: 4개
- 카페: 2개
- 영화, OTT, 통신, 배달, 편의점, 뷰티: 각 1개

이 데이터는 테스트용이며, 실제 크롤링 데이터로 점진적으로 교체됩니다.

