# 🤖 Bot Server

> 자동화된 봇 티케팅 시스템

## 📌 개요

Bot Server는 티케팅 게임에서 실제 사용자와 경쟁하는 자동화된 봇을 관리하는 Go 기반 서버입니다. 난이도별 봇 레벨 시스템, 지능형 좌석 선택 알고리즘, Kafka 기반 대기큐 관리를 제공합니다.

## 🛠️ 기술 스택

### Core
- **Go 1.25.3**
- **Gin** - 웹 프레임워크
- **net/http** - HTTP 클라이언트

### Infrastructure
- **Kafka (Sarama)** - 이벤트 기반 봇 시작/종료
- **Minio** - 객체 스토리지 (공연장 데이터)

### Libraries
- **Zap** - 구조화된 로깅
- **godotenv** - 환경변수 관리
- **UUID** - 고유 ID 생성

## 🏗️ 아키텍처

### 디렉토리 구조
```
bot-server/
├── main.go                   # 애플리케이션 진입점
├── api/                      # HTTP API 서버
│   ├── server.go            # Gin 서버 설정
│   └── system_handler.go    # 시스템 핸들러
├── bot/                      # 봇 관련 로직
│   ├── bot.go               # 봇 구조체 및 실행 로직
│   ├── config.go            # 봇 설정
│   ├── distribution.go      # 좌석 분배 로직
│   ├── handler.go           # 봇 HTTP 핸들러
│   ├── level.go             # 난이도별 레벨 시스템
│   ├── seat_selector.go     # 좌석 선택 알고리즘
│   └── service.go           # 봇 서비스 로직
├── client/                   # 외부 API 클라이언트
│   ├── http_client.go       # HTTP 클라이언트 (Ticketing API)
│   ├── minio_client.go      # Minio 클라이언트
│   ├── request.go           # 요청 DTO
│   ├── response.go          # 응답 DTO
│   └── error.go             # 에러 처리
├── config/                   # 설정 관리
│   └── config.go            # 환경변수 설정
├── kafka/                    # Kafka 컨슈머
│   └── consumer.go          # 봇 시작/디큐 이벤트 구독
├── logger/                   # 로깅
│   └── logger.go            # Zap 로거 설정
├── match/                    # 매치 관리
│   ├── context.go           # 매치 컨텍스트
│   ├── handler.go           # 매치 HTTP 핸들러
│   ├── service.go           # 매치 서비스
│   └── status.go            # 매치 상태
├── models/                   # 데이터 모델
│   ├── difficulty.go        # 난이도 Enum
│   ├── hall.go              # 공연장 모델
│   ├── local_datetime.go    # LocalDateTime 구현
│   ├── request.go           # 공통 요청 DTO
│   └── response.go          # 공통 응답 DTO
├── scheduler/                # 스케줄러
│   └── schedular.go         # 매치 스케줄링
├── stats/                    # 통계 수집
│   ├── aggregator.go        # 통계 집계
│   ├── collector.go         # 데이터 수집
│   └── sender.go            # 통계 전송
└── utils/                    # 유틸리티
    ├── httputil/
    │   └── response.go      # HTTP 응답 유틸
    └── random.go            # 랜덤 유틸
```

### 시스템 흐름도
```
                    [Room Server]
                          │
                          │ HTTP: 매치 생성
                          ↓
              ┌───────────────────────────┐
              │   Ticketing Server        │
              │   (매치 스케줄러)          │
              └───────────────────────────┘
                          │
                          │ 1) HTTP POST /matches/{matchId}/bots
                          │    (봇 시작 요청)
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                       Bot Server                            │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐    │
│  │ Match        │  │ Bot          │  │ Kafka Consumer  │    │
│  │ Service      │  │ Instances    │  │                 │    │
│  │              │→ │ (Goroutine)  │← │ bot-dequeued    │    │
│  └──────────────┘  └──────────────┘  └─────────────────┘    │
└─────────────────────────────────────────────────────────────┘
         │                    │                    ↑
         │ 2) Minio에서        │                    │
         │    공연장 데이터     │                    │ 4) Kafka
         │    로드             │                    │    bot-dequeued-publish
         ↓                    │                    │    (개별 봇 대기 해제)
    ┌────────┐                │                    │
    │ Minio  │                │ 3) 봇 생성 및       │
    │(공연장  │                │    대기 상태        │
    │ 데이터) │                │                    │
    └────────┘                ↓                    │
                    [대기 중 봇들]                  │
                              │                    │
                              └────────────────────┘
                              │
                              │ 5) 대기 해제 후 티케팅 시작
                              │    (HTTP 호출)
                              ↓
              ┌───────────────────────────────┐
              │   Ticketing Server API        │
              │  • POST /queue/{matchId}      │
              │    (대기열 참여)               │
              │  • GET /captcha/validate/bot  │
              │    (캡챠 검증)                 │
              │  • GET /sections/.../status   │
              │    (좌석 상태 조회)            │
              │  • POST /hold                 │
              │    (좌석 선점)                 │
              │  • POST /seats/confirm        │
              │    (좌석 확정)                 │
              └───────────────────────────────┘
                              │
                              │ 6) 결과 전송
                              ↓
                       [Stats Server]
```

**통신 프로토콜:**
- **HTTP**: Ticketing Server ↔ Bot Server (봇 시작 요청, 티케팅 API 호출)
- **Kafka**: Ticketing Server → Bot Server (개별 봇 대기 해제 이벤트)
- **Minio**: Bot Server → Minio (공연장 좌석 데이터 로드)

## 🚀 주요 기능

### 1. 봇 관리 시스템

**봇 생성 및 가용량 관리**
- 동시 실행 가능한 최대 봇 수 관리 (기본 50,000개)
- 매치별 봇 생성 및 할당
- 가용 봇 수 조회 API

**봇 라이프사이클**
```
생성 → 대기 → 시작 → 티케팅 → 종료
```

### 2. 난이도 기반 레벨 시스템

**3가지 난이도**
- **EASY (쉬움)**
  - 반응 속도: 2000~4000ms
  - Captcha 실패율: 40%
  - 좌석 선호도: 랜덤

- **NORMAL (보통)**
  - 반응 속도: 1000~2000ms
  - Captcha 실패율: 20%
  - 좌석 선호도: 중간 선호

- **HARD (어려움)**
  - 반응 속도: 300~1000ms
  - Captcha 실패율: 5%
  - 좌석 선호도: 최적 좌석 선택

**동적 난이도 조절**
```go
type BotLevel struct {
    MinDelay           time.Duration  // 최소 반응 시간
    MaxDelay           time.Duration  // 최대 반응 시간
    CaptchaFailRate    float64        // Captcha 실패율
    SeatPreference     string         // 좌석 선호도
}
```

### 3. 지능형 좌석 선택 알고리즘

**우선순위 기반 선택**
1. **Best Seats (최고 좌석)**
   - 중앙 앞쪽 좌석
   - VIP 구역
   - 무대와의 최적 거리

2. **Good Seats (좋은 좌석)**
   - 중앙 중간 좌석
   - 시야가 좋은 일반석

3. **Normal Seats (일반 좌석)**
   - 측면 좌석
   - 뒷좌석

**순환 할당 (Round-Robin)**
```go
// 같은 등급 내에서 순환 할당하여 분산
currentIndex := (lastIndex + 1) % len(seats)
```

**난이도별 선택 전략**
- HARD: Best → Good → Normal 순서로 선택
- NORMAL: Good → Best → Normal 순서로 선택
- EASY: 완전 랜덤 선택

### 4. Kafka 이벤트 기반 실행

**구독하는 이벤트**
- `bot.match.start` - 매치 시작 (봇 티케팅 시작)
- `bot.user.dequeued` - 유저 디큐 (개별 봇 대기 해제)

**이벤트 처리 흐름**
```go
// Match Start Event
1. 이벤트 수신
2. Match 정보 파싱
3. 해당 Match의 모든 봇 시작
4. 공연장 데이터 Minio에서 로드
5. 봇별로 Goroutine 생성
6. 티케팅 프로세스 실행

// User Dequeued Event
1. 이벤트 수신
2. userId 확인
3. 해당 봇만 대기 해제
4. 티케팅 시작
```

### 5. 티케팅 프로세스

**단계별 실행**
```
1. 대기큐 진입
   ↓
2. 매치 시작 대기
   ↓
3. Captcha 검증 (난이도별 실패율 적용)
   ↓
4. 좌석 선택 (알고리즘 기반)
   ↓
5. 좌석 선점 시도
   ↓
6. 좌석 확정
   ↓
7. 결과 수집 및 통계 전송
```

**Ticketing API 호출**
- `POST /api/validate-captcha` - Captcha 검증
- `POST /api/seats/reserve` - 좌석 선점
- `POST /api/seats/confirm` - 좌석 확정

### 6. Minio 공연장 데이터 관리

**데이터 구조**
```json
{
  "hallId": "charlotte-theater",
  "hallName": "샬롯 씨어터",
  "totalSeats": 1000,
  "sections": [
    {
      "name": "VIP",
      "rows": [
        {
          "rowNumber": 1,
          "seats": [
            {"seatNumber": 1, "grade": "BEST"},
            {"seatNumber": 2, "grade": "BEST"}
          ]
        }
      ]
    }
  ]
}
```

**캐싱 전략**
- 매치 시작 시 공연장 데이터 한 번만 로드
- 메모리 캐싱으로 반복 로드 방지
- 매치 종료 시 캐시 해제

### 7. 통계 수집 및 전송

**수집하는 통계**
- 봇별 티케팅 성공/실패 여부
- 선택한 좌석 정보
- 소요 시간
- Captcha 검증 결과

**통계 전송**
```go
// Stats Server로 HTTP POST
POST /api/stats/ticketing
{
  "matchId": 123,
  "botId": -1,
  "success": true,
  "seatNumber": "A-15",
  "duration": 1234
}
```

### 8. 스케줄링 시스템

**매치 시작 시간 기반 자동 실행**
- 매치 생성 시 시작 시간 등록
- 시작 시간 30초 전부터 대기
- 정확한 시간에 봇 티케팅 시작

**Goroutine 기반 동시 실행**
```go
// 각 봇은 독립적인 Goroutine에서 실행
for _, bot := range bots {
    go bot.StartTicketing()
}
```

## 📡 API 명세

### Match API

```http
POST   /api/matches
# 매치 생성 및 봇 할당
# Request Body:
{
  "matchId": 123,
  "hallId": "charlotte-theater",
  "difficulty": "NORMAL",
  "botCount": 100,
  "startTime": "2024-01-15T19:00:00"
}
# Response:
{
  "matchId": 123,
  "botsCreated": 100,
  "status": "SCHEDULED"
}

GET    /api/matches/{matchId}
# 매치 상태 조회
# Response:
{
  "matchId": 123,
  "status": "RUNNING",
  "totalBots": 100,
  "activeBots": 85,
  "completedBots": 15
}

DELETE /api/matches/{matchId}
# 매치 강제 종료
# Response:
{
  "matchId": 123,
  "stopped": true
}
```

### Bot API

```http
GET    /api/bots/capacity
# 봇 가용량 조회
# Response:
{
  "maxCapacity": 50000,
  "currentUsage": 1234,
  "available": 48766
}

GET    /api/bots/stats
# 봇 통계 조회
# Response:
{
  "totalExecuted": 10000,
  "successRate": 0.85,
  "averageDuration": 2345
}
```

### System API

```http
GET    /health
# 헬스 체크
# Response:
{
  "status": "UP",
  "timestamp": "2024-01-15T10:30:00Z"
}

GET    /api/system/info
# 시스템 정보
# Response:
{
  "version": "1.0.0",
  "goVersion": "1.25.3",
  "uptime": 3600
}
```

## ⚙️ 환경 설정

### 필수 환경변수

```bash
# Server Configuration
SERVER_PORT=8080
ENVIRONMENT=development

# External API URLs
TICKETING_API_URL=http://localhost:3000
STATS_SERVER_URL=http://localhost:4000
ROOM_SERVER_URL=http://localhost:8080

# Kafka Configuration
KAFKA_BROKERS=localhost:9092
KAFKA_GROUP_ID=bot-server-group
KAFKA_TOPIC_MATCH_START=bot.match.start
KAFKA_TOPIC_USER_DEQUEUED=bot.user.dequeued

# Minio Configuration
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET_NAME=tickget
MINIO_USE_SSL=false

# Bot Configuration
MAX_CONCURRENT_BOTS=50000
BOT_ID_START=-1
BOT_ID_END=-50000

# Logging
LOG_LEVEL=debug
LOG_OUTPUT=stdout
```

### .env 파일 생성

```bash
# .env.example을 복사하여 .env 생성
cp .env.example .env

# 환경에 맞게 수정
vim .env
```

## 🏃 실행 방법

### 1. 사전 요구사항
- Go 1.25 이상
- Kafka 3.0 이상
- Minio
- Ticketing Server 실행 중

### 2. 의존성 설치
```bash
go mod download
```

### 3. 빌드
```bash
go build -o bot-server main.go
```

### 4. 로컬 실행
```bash
# 환경변수 설정 후
./bot-server
```

또는

```bash
# go run으로 직접 실행
go run main.go
```

### 5. Docker 실행
```bash
# 이미지 빌드
docker build -t bot-server .

# 컨테이너 실행
docker run -p 8080:8080 \
  --env-file .env \
  bot-server
```

## 📊 모니터링

### 주요 지표
- 봇 실행 수 및 성공률
- Captcha 성공률 (난이도별)
- 평균 티케팅 소요 시간
- 좌석 선택 분포

### 로깅
Zap 구조화 로깅을 사용하여 봇 동작을 추적합니다.
