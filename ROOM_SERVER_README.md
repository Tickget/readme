# 🎫 Room Server

> 실시간 티케팅 방 관리 및 대기큐 알림 시스템

## 📌 개요

Room Server는 티케팅 게임의 방(Room) 관리, WebSocket 기반 실시간 통신, 대기큐 알림 관리를 담당하는 MSA 기반 백엔드 서버입니다.

## 🛠️ 기술 스택

### Core
- **Java 21**
- **Spring Boot 3.5.7**
- **Spring WebSocket** - 실시간 양방향 통신
- **Spring Data JPA** - ORM
- **MySQL** - RDB

### Infrastructure
- **Redis** - 세션 관리, 캐싱, 동시성 제어 (Lua Script)
- **Kafka** - 이벤트 기반 MSA 통신
- **Minio** - 객체 스토리지 (썸네일)

### Libraries
- **Resilience4j** - 서킷 브레이커
- **Thumbnailator** - 이미지 리사이징
- **Lombok** - 보일러플레이트 코드 제거

## 🏗️ 아키텍처

### 디렉토리 구조
```
src/main/java/com/tickget/roomserver/
├── config/               # 설정
│   ├── KafkaConfig
│   ├── RedisConfig
│   ├── RedisLuaScriptConfig
│   ├── WebSocketConfig
│   └── MinioConfig
├── controller/           # REST API & WebSocket
│   ├── RoomController
│   ├── ThumbnailController
│   ├── HallController
│   └── HealthController
├── domain/
│   ├── entity/          # JPA 엔티티
│   │   ├── Room
│   │   ├── PresetHall
│   │   └── BaseTimeEntity
│   ├── enums/           # Enum 정의
│   │   ├── RoomStatus (WAITING, PLAYING, CLOSED)
│   │   ├── RoomType
│   │   ├── HallType
│   │   ├── HallSize
│   │   ├── Difficulty
│   │   └── EventType
│   └── repository/      # Repository
│       ├── RoomRepository
│       ├── RoomCacheRepository
│       └── PresetHallRepository
├── dto/
│   ├── request/         # 요청 DTO
│   ├── response/        # 응답 DTO
│   └── cache/           # Redis 캐시용 DTO
├── event/               # 도메인 이벤트
│   ├── UserJoinedRoomEvent
│   ├── UserLeftRoomEvent
│   ├── HostChangedEvent
│   ├── RoomPlayingStartedEvent
│   ├── RoomPlayingEndedEvent
│   ├── RoomSettingUpdatedEvent
│   ├── SessionCloseEvent
│   └── UserDequeuedEvent
├── exception/           # 커스텀 예외
│   ├── RoomNotFoundException
│   ├── RoomFullException
│   ├── RoomClosedException
│   ├── RoomPlayingException
│   ├── UnauthorizedException
│   └── GlobalExceptionHandler
├── kafka/               # Kafka Producer/Consumer
│   ├── RoomEventProducer
│   ├── RoomEventConsumer
│   ├── RoomEventMessage
│   └── payload/         # Kafka 메시지 페이로드
├── listener/            # WebSocket 이벤트 리스너
│   └── WebSocketEventListener
├── service/             # 비즈니스 로직
│   ├── RoomService
│   ├── RoomEventHandler
│   ├── RoomNotificationScheduler
│   ├── HallService
│   ├── ThumbnailService
│   ├── MinioService
│   └── TicketingServiceClient
├── session/             # WebSocket 세션 관리
│   ├── WebSocketSessionManager
│   └── SessionInfo
└── util/                # 유틸리티
    ├── TimeConverter
    └── ServerIdProvider
```

### MSA 통신 구조
```
                           [Client]
                              │
                              │ WebSocket (STOMP)
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                       Room Server                           │
│  ┌────────────────┐  ┌─────────────┐  ┌─────────────────┐   │
│  │ WebSocket      │  │ RoomService │  │ Kafka Producer/ │   │
│  │ Handler        │←→│             │←→│ Consumer        │   │
│  └────────────────┘  └─────────────┘  └─────────────────┘   │
└─────────────────────────────────────────────────────────────┘
         │                    │                    │
         │                    │                    │ Kafka (자체 이벤트)
         │                    │                    │ - room-user-joined-events                  
         │                    │                    │ - room-user-left-events                     
         │                    │                    │ - room-host-changed-events                  
         │                    │                    │ - session-close-events                     
         │                    │                    │ - room-setting-updated-events               
         │                    │                    │ - room-playing-started/ended-events        
         │                    │                    ↓
         │                    │              [Room Server 모든 인스턴스]
         │                    │
         │                    │ HTTP (양방향)
         │                    │ ────────────────────────────────┐
         │                    │                                 │
         │                    ↓                                 ↓
         │         ┌──────────────────────────────────────────────────┐
         │         │            Ticketing Server                      │
         │         │  • Room → Ticketing: 매치 생성, 유저 퇴장 알림     │
         │         │  • Ticketing → Room: 방 시작/종료, 방 정보 조회    │
         │         └──────────────────────────────────────────────────┘
         │                    │
         │                    │ Kafka
         │                    │ user-dequeued-publish
         │                    │ (대기열 이탈 이벤트)
         │                    ↓
         │              [Room Server]
         │              (WebSocket 브로드캐스트)
         │
         ↓
    ┌────────┐     ┌────────┐     ┌────────┐
    │ Redis  |     │ MySQL  │     │ Minio  │
    │(세션    |     │(영구   │     │(썸네일) │
    │ 캐시)   │     │ 저장)  │     │        │
    └────────┘     └────────┘     └────────┘
```

## 🚀 주요 기능

### 1. 방(Room) 관리
- **방 생성/조회/입장/퇴장**
  - 공연장(Hall) 기반 방 생성
  - 난이도 설정 (EASY, NORMAL, HARD)
  - 최대 인원 검증
  - 방 상태 관리 (WAITING, PLAYING, CLOSED)

- **방장 승계**
  - 방장이 나가면 자동으로 다음 유저에게 승계
  - Kafka를 통한 방장 변경 이벤트 전파

- **Redis 기반 캐싱**
  - 방 정보 조회 성능 최적화
  - TTL 기반 자동 만료

### 2. WebSocket 실시간 통신
- **실시간 대기큐 정보 브로드캐스트**
  - 현재 대기 인원 실시간 업데이트
  - 매치 시작까지 남은 시간 표시

- **유저 입장/퇴장 알림**
  - 방 내 모든 유저에게 실시간 알림
  - 현재 인원 정보 업데이트

- **중복 세션 방지**
  - 같은 유저의 다중 접속 차단
  - 기존 세션 강제 종료 로직

- **재연결 처리**
  - 일시적 네트워크 끊김 시 방 유지
  - Grace Period 적용 (일정 시간 대기)

- **Version 기반 세션 관리**
  - 세션 버전을 통한 정합성 보장
  - 동시 접속 시 최신 세션만 유지

### 3. Redis 캐싱 & 동시성 제어
- **캐싱 전략**
  - 방 목록 캐싱 (조회 성능 최적화)
  - 방 상세 정보 캐싱
  - 세션 정보 캐싱
  - 대기큐 상태 캐싱

- **Lua Script를 통한 Race Condition 방지**
  - 웹소켓 동시 접속 시 원자적 연산 보장
  - Redis 트랜잭션 처리

- **TTL 관리**
  - 자동 만료를 통한 메모리 최적화
  - 방 상태별 차등 TTL 적용

### 4. Kafka 이벤트 시스템

#### 발행하는 이벤트
- `USER_JOINED_ROOM` - 유저가 방에 입장
- `USER_LEFT_ROOM` - 유저가 방에서 퇴장
- `HOST_CHANGED` - 방장이 변경됨
- `ROOM_SETTING_UPDATED` - 방 설정이 변경됨
- `MATCH_STARTED` - 매치(티케팅)가 시작됨
- `MATCH_ENDED` - 매치가 종료됨
- `USER_DEQUEUED` - 유저가 대기큐에 진입
- `FORCE_DISCONNECT` - 세션 강제 종료 요청

#### 구독하는 이벤트
- Bot Server로부터 봇 관련 이벤트 수신
- Ticketing Server로부터 매치 상태 이벤트 수신

### 5. 대기큐 시스템
- **실시간 대기 인원 조회**
  - WebSocket을 통한 실시간 업데이트
  - 매치별 대기 인원 관리

- **주기적 대기큐 정보 업데이트**
  - Spring Scheduler를 통한 정기 브로드캐스트
  - Redis 기반 큐 상태 관리

- **매치 시작 시 디큐 알림**
  - 대기 중인 유저에게 시작 알림
  - 자동 방 상태 전환 (WAITING → PLAYING)

### 6. 썸네일 관리
- **Minio를 통한 이미지 업로드**
  - 방 썸네일 이미지 저장
  - 공연장 이미지 관리

- **Thumbnailator 리사이징**
  - 자동 이미지 리사이징
  - 용량 최적화

- **방 생성과 분리된 업로드 로직**
  - 방 생성 후 별도 업로드
  - 실패 시에도 방 생성 유지

### 7. MSA 통신 & 장애 대응
- **Ticketing Server와 HTTP 통신**
  - 매치 생성 요청
  - RestTemplate 기반 통신

- **SAGA 패턴 적용**
  - 분산 트랜잭션 관리
  - 보상 트랜잭션 구현
  - DB 정합성 보장

- **Resilience4j 서킷 브레이커**
  - 장애 전파 방지
  - 타임아웃 설정
  - Fallback 처리

## 📡 API 명세

### Room API
```http
POST   /api/v1/rooms
# 방 생성
# Request Body: CreateRoomRequest
# Response: CreateRoomResponse

GET    /api/v1/rooms
# 방 목록 조회
# Query: page, size
# Response: List<RoomResponse>

GET    /api/v1/rooms/{roomId}
# 방 상세 조회
# Response: RoomDetailResponse

POST   /api/v1/rooms/{roomId}/join
# 방 입장
# Request Body: JoinRoomRequest
# Response: JoinRoomResponse

POST   /api/v1/rooms/{roomId}/exit
# 방 퇴장
# Request Body: ExitRoomRequest
# Response: ExitRoomResponse

PUT    /api/v1/rooms/{roomId}/settings
# 방 설정 변경
# Request Body: MatchSettingUpdateRequest
# Response: void
```

### Thumbnail API
```http
POST   /api/v1/thumbnails
# 썸네일 업로드
# Content-Type: multipart/form-data
# Response: ThumbnailUploadResponse
```

### Hall API
```http
POST   /api/v1/halls
# 공연장 생성
# Request Body: CreateHallRequest
# Response: CreateHallResponse

GET    /api/v1/halls/{hallId}
# 공연장 조회
# Response: PresetHall
```

### Health Check
```http
GET    /health
# 헬스 체크
# Response: { "status": "UP" }

GET    /actuator/health
# Actuator 헬스 체크
# Response: Actuator Health Info
```

### WebSocket
```
STOMP  /ws
# WebSocket 연결

구독 경로:
/topic/queue/{matchId}      # 대기큐 실시간 정보
/topic/room/{roomId}        # 방 이벤트 알림
```

## ⚙️ 환경 설정

### 필수 환경변수

```properties
# Database
SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/tickget?serverTimezone=Asia/Seoul
SPRING_DATASOURCE_USERNAME=root
SPRING_DATASOURCE_PASSWORD=password

# Redis
SPRING_DATA_REDIS_HOST=localhost
SPRING_DATA_REDIS_PORT=6379

# Kafka
SPRING_KAFKA_BOOTSTRAP_SERVERS=localhost:9092

# Minio
MINIO_ENDPOINT=http://localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET_NAME=tickget

# Ticketing Server
TICKETING_SERVER_URL=http://localhost:8081
```

### application.yml 예시

```yaml
spring:
  application:
    name: room-server

  datasource:
    url: ${SPRING_DATASOURCE_URL}
    username: ${SPRING_DATASOURCE_USERNAME}
    password: ${SPRING_DATASOURCE_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        show_sql: true
        format_sql: true
    open-in-view: false

  data:
    redis:
      host: ${SPRING_DATA_REDIS_HOST}
      port: ${SPRING_DATA_REDIS_PORT}

  kafka:
    bootstrap-servers: ${SPRING_KAFKA_BOOTSTRAP_SERVERS}
    consumer:
      group-id: room-server-group
      auto-offset-reset: latest
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer

server:
  port: 8080

minio:
  endpoint: ${MINIO_ENDPOINT}
  access-key: ${MINIO_ACCESS_KEY}
  secret-key: ${MINIO_SECRET_KEY}
  bucket-name: ${MINIO_BUCKET_NAME}

resilience4j:
  circuitbreaker:
    instances:
      ticketingService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10000
```

## 🏃 실행 방법

### 1. 사전 요구사항
- Java 21 이상
- MySQL 8.0 이상
- Redis 7.0 이상
- Kafka 3.0 이상
- Minio

### 2. 의존성 설치
```bash
./gradlew build
```

### 3. 로컬 실행
```bash
# 환경변수 설정 후
./gradlew bootRun
```

### 4. Docker 실행
```bash
# 이미지 빌드
docker build -t room-server .

# 컨테이너 실행
docker run -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://host.docker.internal:3306/tickget \
  -e SPRING_DATA_REDIS_HOST=host.docker.internal \
  -e SPRING_KAFKA_BOOTSTRAP_SERVERS=host.docker.internal:9092 \
  room-server
```

## 🔒 동시성 제어

### Redis Lua Script를 통한 원자적 연산
웹소켓 동시 접속 시 Race Condition을 방지하기 위해 Redis Lua Script를 사용하여 세션 등록과 기존 세션 체크를 원자적으로 처리합니다.

### Version 기반 세션 관리
타임스탬프 기반 버전을 통해 최신 세션만 유효한 것으로 인정하여 중복 접속을 방지합니다.

## 📊 모니터링

### 주요 지표
- WebSocket 연결 수
- 방 생성/입장/퇴장 TPS
- Redis 캐시 히트율
- Kafka 메시지 처리 지연
- API 응답 시간

Spring Boot Actuator를 통해 `/actuator/health`, `/actuator/metrics` 등의 엔드포인트로 모니터링할 수 있습니다.
