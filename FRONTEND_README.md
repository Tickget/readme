# Tickget 프론트엔드

> 함께하는 티켓팅 연습 플랫폼

Tickget은 실제 티켓팅 환경을 시뮬레이션하여 사용자가 티켓팅 연습을 할 수 있는 웹 애플리케이션입니다. 여러 사용자가 동시에 참여하는 멀티플레이어 환경에서 실시간으로 좌석을 선택하고, 게임 결과를 분석하며, 개인 통계를 확인할 수 있는 플랫폼을 구축했습니다.

## 📋 목차

- [기술 스택](#기술-스택)
- [주요 기능](#주요-기능)
- [프로젝트 구조](#프로젝트-구조)
- [핵심 기능 상세](#핵심-기능-상세)
- [시작하기](#시작하기)
- [빌드 및 배포](#빌드-및-배포)
- [테스트](#테스트)
- [환경 변수](#환경-변수)

## 🛠 기술 스택

### Core

- **React 19.2.0** - UI 라이브러리
- **TypeScript 5.9.3** - 타입 안정성
- **Vite 7.1.10** - 빌드 도구 및 개발 서버

### 상태 관리

- **Zustand 5.0.8** - 경량 상태 관리 라이브러리
  - roomStore: 방 상태 관리
  - matchStore: 매치 상태 관리
  - authStore: 인증 상태 관리

### 라우팅

- **React Router DOM 7.9.4** - 클라이언트 사이드 라우팅

### UI/스타일링

- **Material-UI (MUI) 7.3.4** - 컴포넌트 라이브러리
- **Tailwind CSS 4.1.14** - 유틸리티 기반 CSS 프레임워크
- **Emotion** - CSS-in-JS 스타일링

### 실시간 통신

- **@stomp/stompjs 7.2.1** - STOMP 프로토콜 클라이언트
- **sockjs-client 1.6.1** - WebSocket 폴백 라이브러리

### 데이터 시각화

- **Recharts 3.3.0** - 차트 라이브러리
  - 개인 기록 그래프
  - 통계 데이터 시각화

### 날짜 처리

- **dayjs 1.11.18** - 경량 날짜 라이브러리
- **@mui/x-date-pickers 7.29.4** - 날짜 선택 컴포넌트

### HTTP 클라이언트

- **Axios** - HTTP 클라이언트 (API 통신)

### 테스트

- **Cypress 15.4.0** - E2E 테스트 프레임워크
- **Jest 30.2.0** - 유닛 테스트 프레임워크
- **Testing Library** - React 컴포넌트 테스트

## ✨ 주요 기능

### 1. 티켓팅 연습

- 실시간 티켓팅 시뮬레이션
- 여러 사용자가 동시에 참여하는 멀티플레이어 환경
- WebSocket을 통한 실시간 좌석 상태 업데이트
- 좌석 선점 상태 처리 및 동기화

### 2. 공연장 지원

- **소규모 공연장**: 샤롯데시어터
  - 좌석 배치도 및 선택 기능
  - 좌석 등급별 색상 시스템
  - 좌석 숨김 처리 기능

- **중규모 공연장**: 올림픽홀
  - R석, S석, VIP석, 스탠딩 구현
  - 좌석 행/열 정보 시스템
  - 좌석 선점 상태 반영

- **대규모 공연장**: 인스파이어 아레나
  - R석, S석, VIP석, 스탠딩 구현
  - 대규모 좌석 시스템 처리
  - 실시간 좌석 상태 관리

### 3. 예매 사이트 통합

- 익스터파크(Exterpark) 예매 시뮬레이션
  - 익스터파크 UI 레이아웃 구현
  - 캡챠 모달 UI
  - 예매 대기 페이지
  - 예매 프로세스 시간 측정

### 4. 사용자 기능

- Google 소셜 로그인
  - OAuth 인증 플로우
  - 로그인 성공 라우팅 처리
  - 자동 로그아웃 방지 로직

- 개인 통계 및 분석
  - 사용자 랭킹/퍼센타일 통계
  - 주간 랭킹 시스템
  - 개인 기록 그래프 시각화 (Recharts)
  - 경기 기록 페이지네이션

- 매칭 히스토리 조회
  - 경기 기록 상세 정보 표시
  - 공연장 툴팁 및 유저 정보
  - 경기 기록 색상 구분

- 예매 내역 관리
- 프로필 이미지 관리
  - 프로필 이미지 업로드
  - 헤더 프로필 이미지 반영
  - 기본 프로필 이미지 지정

### 5. 통계 및 랭킹

- 주간 랭킹 시스템
- 개인 성과 분석
  - 그래프를 통한 시각화
  - 통계 프로필 및 봇 제한
- 예매 성공률 통계
- 실패한 유저 정보 표시

### 6. AI 분석 기능

- 게임 결과 AI 분석
  - AI 경기 분석 리포트 조회
  - AI 좌석 등급 기능
  - AI 공연장 TSX 생성 API 연동
  - AI 생성 상태 표시 및 렌더링

### 7. 방 관리 시스템

- 방 생성 및 관리
  - 방 생성 API 연동
  - 방 설정 저장/불러오기
  - 방 썸네일 S3 URL 관리
  - 방 목록 새로고침 기능

- 관리자 기능
  - 관리자 권한 관리
  - 방장 표시 기능
  - 관리자만 방 생성 제한

- 방 입장/퇴장
  - 방 입장/퇴장 로직
  - 방 구독 기능 (WebSocket)
  - 경기 시작 30초 전 입장 차단
  - 경기 중 뒤로가기 및 자동 나가기 처리

### 8. 실시간 통신

- WebSocket 연결 및 관리
  - 방 생성 시 WebSocket 연결
  - 유저 방 구독 기능
  - 실시간 좌석 상태 업데이트
  - WebSocket 연결 끊김 처리

- 대기열 시스템
  - 실시간 대기열 관리
  - 대기열 순위 계산 (ahead, behind)
  - 대기 페이지 진행도 표시
  - 큐 등록 로직

### 9. 게임 결과 및 통계

- 게임 결과창
  - 결과창 표시 및 섹션 렌더링
  - 로딩 애니메이션
  - 경기 기록 페이지 이동 로직

- 통계 계산
  - 계산 로직 구현
  - 실패 계산 로직
  - 통계 프로필 및 봇 제한

## 📁 프로젝트 구조

```text
src/
├── app/                    # 애플리케이션 설정
│   ├── layouts/           # 레이아웃 컴포넌트
│   │   ├── AuthLayout.tsx
│   │   ├── MainLayout.tsx
│   │   └── PlainLayout.tsx
│   └── routes/            # 라우팅 설정
│       ├── guards/        # 라우트 가드
│       ├── index.tsx
│       └── paths.ts
├── components/            # 공통 컴포넌트
│   └── SeatTakenAlert.tsx
├── features/              # 기능별 모듈
│   ├── auth/             # 인증 관련
│   │   ├── api.ts
│   │   ├── store.ts
│   │   └── types.ts
│   ├── booking-site/     # 예매 사이트 관련
│   ├── home/             # 홈 페이지 관련
│   ├── performance-hall/ # 공연장 관련
│   ├── room/             # 방(게임) 관련
│   │   ├── api.ts
│   │   ├── store.ts
│   │   └── types.ts
│   ├── search/           # 검색 관련
│   ├── site/             # 사이트 관련
│   ├── stats/            # 통계 관련
│   └── user-page/        # 사용자 페이지 관련
├── pages/                 # 페이지 컴포넌트
│   ├── auth/             # 인증 페이지
│   │   ├── login/
│   │   └── sign-up/
│   ├── booking-site/     # 예매 사이트 페이지
│   │   ├── exterpark-site/
│   │   └── watermelon-site/
│   ├── home/             # 홈 페이지
│   ├── performance-halls/# 공연장 페이지
│   │   ├── large-venue/  # 대형 공연장 (인스파이어 아레나)
│   │   ├── medium-venue/ # 중형 공연장 (올림픽홀)
│   │   └── small-venue/  # 소형 공연장 (샤롯데시어터)
│   ├── room/             # 방 생성/설정 페이지
│   ├── stats/            # 통계 페이지
│   └── user-page/        # 마이페이지
├── shared/                # 공유 리소스
│   ├── api/              # API 유틸리티
│   ├── hooks/            # 커스텀 훅
│   │   ├── useAlert.tsx
│   │   ├── useBlockBackButtonDuringGame.ts
│   │   ├── useConfirm.tsx
│   │   └── useSeatStatsFailedOnUnload.ts
│   ├── lib/              # 라이브러리 설정
│   │   ├── http.ts
│   │   ├── websocket.ts
│   │   └── websocket-store.ts
│   ├── providers/        # Context Provider
│   │   └── AlertProvider.tsx
│   ├── ui/               # UI 컴포넌트
│   │   ├── base/
│   │   ├── common/
│   │   └── dev/
│   └── utils/            # 유틸리티 함수
│       ├── alert.ts
│       ├── confirm.ts
│       ├── imageCompression.ts
│       ├── profileImageUrl.ts
│       └── reserveMetrics.ts
└── types/                # 타입 정의
```

## 🔧 핵심 기능 상세

### 공연장 좌석 시스템

#### 좌석 구조

- **좌석 등급**: VIP, R석, S석, 스탠딩
- **좌석 정보**: 행/열 정보, 섹션 정보
- **좌석 상태**: 선택 가능, 선점됨, 숨김 처리

#### 좌석 선택 로직

- 실시간 좌석 선점 상태 반영
- 좌석 ID 처리 로직
- 좌석 등급별 색상 시스템
- 좌석 재선택 시 취소 처리

### 실시간 통신 시스템

#### WebSocket 연결

- STOMP 프로토콜을 통한 실시간 통신
- 방 구독 및 이벤트 처리
- 연결 끊김 시 자동 재연결
- 사용자 정보 포함 연결

#### 대기열 시스템

- 대기열 순위 계산 (ahead, behind)
- 대기 페이지 진행도 표시
- 큐 등록 및 상태 관리
- 대기열 게이지바 계산

### 데이터 시각화

#### Recharts 활용

- 개인 기록 그래프 (라인 차트)
- 통계 데이터 시각화
- 랭킹 정보 표시

#### 통계 시스템

- 사용자 랭킹/퍼센타일 계산
- 주간 랭킹 시스템
- 경기 기록 통계
- 실패 유저 통계

### AI 분석 시스템

#### AI 기능

- 게임 결과 AI 분석
- AI 경기 분석 리포트 조회
- AI 공연장 TSX 생성
- AI 좌석 등급 분석
- AI 생성 상태 표시

### 인증 시스템

#### OAuth 인증

- Google 소셜 로그인
- OAuth 인증 플로우
- 인증 결과 처리
- 로그인 성공 라우팅

#### 세션 관리

- 자동 로그아웃 방지
- WebSocket 연결 끊김 시 로그아웃 처리
- 경기 종료 시 로그아웃 방지

### 예매 플로우

#### 예매 프로세스

- 예매 플로우 초기 구현
- 캡챠 모달 UI
- 예매 대기 페이지
- 좌석 선택 페이지 이동

#### 시간 측정

- 예매 클릭 버튼 소요시간 체크
- 좌석 선정 소요시간 체크
- 보안 문자 입력 시간 체크
- 클릭 실수 추적 로직

#### 결제 처리

- 결제하기 버튼 중복 방지
- 결제 상태 메시지
- 예약 정보 처리

## 🚀 시작하기

### 필수 요구사항

- Node.js 20 이상
- npm 또는 yarn

### 설치

```bash
# 의존성 설치
npm install

# 또는
npm ci --legacy-peer-deps
```

### 개발 서버 실행

```bash
npm run dev
```

개발 서버는 기본적으로 `http://localhost:5173`에서 실행됩니다.

### 빌드

```bash
# 프로덕션 빌드
npm run build

# 타입 체크 포함 빌드
npm run build:check
```

### 미리보기

```bash
# 빌드된 앱 미리보기
npm run preview
```

## 🐳 Docker를 사용한 배포

### Docker 이미지 빌드

```bash
docker build \
  --build-arg VITE_API_PREFIX_DEV=/api/v1/dev \
  --build-arg VITE_API_ORIGIN=https://tickget.kr \
  -t tickget-frontend .
```

### Docker 컨테이너 실행

```bash
docker run -d -p 80:80 tickget-frontend
```

## 🧪 테스트

### E2E 테스트 (Cypress)

```bash
# Cypress GUI 실행
npm run cypress:open

# 헤드리스 모드로 실행
npm run cypress:run

# 또는
npm run test:e2e
```

### 린팅

```bash
npm run lint
```

## ⚙️ 환경 변수

개발 환경에서 API 프록시 설정은 `vite.config.ts`에서 관리됩니다.

프로덕션 빌드 시 다음 환경 변수를 사용할 수 있습니다:

- `VITE_API_PREFIX_DEV`: 개발 API 프리픽스
- `VITE_API_ORIGIN`: API 서버 오리진

## 📝 주요 스크립트

| 명령어                 | 설명                  |
| ---------------------- | --------------------- |
| `npm run dev`          | 개발 서버 실행        |
| `npm run build`        | 프로덕션 빌드         |
| `npm run build:check`  | 타입 체크 포함 빌드   |
| `npm run preview`      | 빌드된 앱 미리보기    |
| `npm run lint`         | ESLint 실행           |
| `npm run cypress:open` | Cypress GUI 실행      |
| `npm run cypress:run`  | Cypress 헤드리스 실행 |
| `npm run test:e2e`     | E2E 테스트 실행       |

## 🎯 기술적 특징

### 성능 최적화

- 이미지 WebP 변환으로 로딩 속도 개선
- API 호출 최적화 (useCallback 활용)
- 컴포넌트 리팩토링을 통한 코드 품질 향상
- 불필요한 패키지 제거 및 빌드 최적화

### 코드 품질

- TypeScript를 통한 타입 안정성 확보
- 컴포넌트 구조 개선 및 재사용성 향상
- 코드 컨벤션 수립 및 일관성 유지
- 문서화를 통한 유지보수성 향상

### 실시간 처리

- WebSocket을 활용한 실시간 상태 동기화
- 좌석 선점 상태 실시간 반영
- 대기열 상태 실시간 업데이트
- 방 구독을 통한 실시간 정보 전달

## 🔗 관련 링크

- [프로젝트 홈페이지](https://tickget.kr)
- [React 공식 문서](https://react.dev)
- [Vite 공식 문서](https://vite.dev)
- [Material-UI 문서](https://mui.com)
- [Recharts 문서](https://recharts.org)