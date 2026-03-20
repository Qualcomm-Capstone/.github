# System Architecture Evolution

> [< Back to README](../profile/README.md#system-architecture)

## 기존 아키텍처의 문제점

기존 모놀리식 구조에서 다음 **4가지 핵심 문제**가 발생했습니다.

- OCR 동기 처리로 인한 서버 처리량 저하
- 느린 응답으로 인한 Edge Device 블로킹
- HTTP 기반 IoT 통신의 구조적 한계
- OCR 장애가 전체 서비스에 전파

## Before → After 비교

| 영역 | Before | After |
|------|--------|-------|
| **OCR 처리** | Django 동기 (블로킹) | OCR Worker 비동기 (Celery prefork) |
| **응답 시간** | 3초+ | < 100ms |
| **IoT 프로토콜** | HTTP (오버헤드) | MQTT (경량, QoS 1) |
| **메시지 보장** | 없음 | At-least-once |
| **장애 격리** | 전체 영향 | 컴포넌트 격리 |
| **확장성** | 서버 전체 확장 | Worker별 독립 확장 |
| **데이터베이스** | 단일 DB | 서비스별 4개 DB |
| **Alert 처리** | Celery 직접 호출 (Orchestration) | Kombu Consumer + Celery gevent (Choreography) |

## 새로운 아키텍처: 3-서비스 모델

Event-Driven Architecture로 전환하여 **Main / OCR Worker / Alert Worker** 3개 서비스로 책임을 분리했습니다.

| 서비스 | 역할 | 핵심 컴포넌트 |
|--------|------|--------------|
| **Main Service** | REST API + MQTT 이벤트 수신 | Django, Gunicorn, MQTT Subscriber |
| **OCR Worker** | 번호판 인식 (CPU-bound) | Celery prefork, EasyOCR, OpenCV |
| **Alert Worker** | FCM 푸시 알림 (I/O-bound) | Kombu Consumer + Celery gevent |

## 배포 구조: 6개 GCE 인스턴스

모든 인스턴스는 `asia-northeast3-a` 리전에 배치됩니다.

```
┌──────────────────────────────────────────────────────────────────────┐
│                       GCP (asia-northeast3-a)                        │
├───────────┬───────────┬───────────┬───────────┬───────────┬─────────┤
│  app      │  db       │  mq       │  ocr      │  alert    │  mon    │
│           │           │           │           │           │         │
│  Django   │  MySQL 8  │  RabbitMQ │  Celery   │  Kombu    │  Prome- │
│  Gunicorn │  4 DBs    │  MQTT     │  prefork  │  Consumer │  theus  │
│  MQTT Sub │           │  AMQP     │           │  + Celery │  Grafana│
│           │           │           │           │  gevent   │  Loki   │
│           │           │           │           │           │  Jaeger │
└───────────┴───────────┴───────────┴───────────┴───────────┴─────────┘
```

| 인스턴스 | 역할 |
|----------|------|
| `speedcam-app` | API 서버 + MQTT 이벤트 수신 |
| `speedcam-db` | MySQL 8.0 (4개 DB) |
| `speedcam-mq` | RabbitMQ (MQTT Plugin + AMQP) |
| `speedcam-ocr` | Celery Worker (prefork pool) |
| `speedcam-alert` | Kombu Consumer + Celery Worker (gevent pool) |
| `speedcam-mon` | Prometheus, Grafana, Loki, Jaeger |

## End-to-End 데이터 흐름

```
Rubik Pi → MQTT (detections/new, QoS 1) → Main Service
  → Detection pending 생성 → ocr_queue (AMQP)
    → OCR Worker (EasyOCR) → detections_db 업데이트
      → domain_events exchange (detections.completed)
        → Kombu Consumer → fcm_queue (AMQP)
          → Celery gevent Worker → FCM push → notifications_db 저장
```

## 메시징 아키텍처: MQTT + AMQP 이중 프로토콜

두 프로토콜을 목적에 따라 분리하여 사용합니다.

| 프로토콜 | 용도 | 특징 |
|----------|------|------|
| **MQTT** (Port 1883) | IoT → Main Service (`detections/new`) | 경량, QoS 1, At-least-once |
| **AMQP** (Port 5672) | Task 분배 + 도메인 이벤트 | Exchange/Queue 라우팅, DLQ 지원 |

## Database per Service (4개 DB)

MSA 원칙에 따라 각 서비스가 독립적인 DB를 사용합니다.

| DB | 담당 서비스 | 저장 데이터 |
|----|-------------|------------|
| `speedcam` | Django Core | Auth, Admin, Sessions, Celery Results |
| `speedcam_vehicles` | Vehicles | 차량 정보, FCM 토큰 |
| `speedcam_detections` | Detections | 과속 감지 내역, OCR 결과 |
| `speedcam_notifications` | Notifications | 알림 전송 이력 |

Cross-DB ForeignKey 대신 **ID Reference** 를 사용하여 서비스 간 결합을 제거했습니다.

---

[< Back to Main README](../profile/README.md)
