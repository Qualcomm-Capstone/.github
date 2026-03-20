# Notification System Design

> [< Back to README](../profile/README.md#notification-system-design)

<h4>Django - RabbitMQ - Celery - FCM(3rd party Service) feat. Ack & Nack</h4>
<img src="https://github.com/user-attachments/assets/671d75ad-a1ab-441e-8159-f55080f38cb4">
<h4>Dead Letter Queue & Dead Letter Consumer</h4>
<img src="https://github.com/user-attachments/assets/75dc90c3-89d8-435b-a36e-5addbeaaf3ce">

## Choreography 패턴 선택 이유

알림 처리에서 **Orchestration 대신 Choreography** 패턴을 선택했습니다.

| 항목 | Choreography (선택) | Orchestration |
|------|:---:|:---:|
| **결합도** | 느슨한 결합 | 강한 결합 |
| **확장성** | 서비스별 독립 확장 | Orchestrator 병목 가능 |
| **장애 격리** | 영향 최소 | 중앙 장애 시 전체 중단 |
| **디버깅** | 흐름 추적 어려움 | 중앙 추적 용이 |

OCR Worker가 작업 완료 후 `domain_events` exchange에 이벤트를 발행하면, Alert Worker가 이를 자율적으로 구독하여 처리합니다. Main Service는 이 흐름에 관여하지 않으며, 흐름 추적의 어려움은 OpenTelemetry 분산 트레이싱으로 보완합니다.

## Alert Worker: 2개 프로세스 구조

```
┌─────────────────────────────┐    ┌──────────────────────────────────┐
│  프로세스 1: Kombu Consumer  │    │  프로세스 2: Celery gevent Worker │
│  단일 스레드                  │    │  --pool=gevent                   │
│                              │    │  --concurrency=50 (기본값)        │
│  domain_events exchange에서  │    │                                   │
│  detections.completed 구독   │    │  send_notification 태스크 처리     │
│                              │    │  greenlet으로 동시 FCM 전송       │
│  → send_notification.delay() │──→│  (I/O-bound 병렬 처리)            │
│    (즉시 반환, 비동기)        │    │                                   │
└─────────────────────────────┘    └──────────────────────────────────┘
```

Kombu Consumer는 도메인 이벤트를 수신하는 즉시 FCM 태스크를 큐에 투입하고 반환합니다. 실제 FCM API 호출은 gevent Worker의 greenlet이 I/O 대기 시간을 활용해 병렬로 처리합니다.

## Exchange / Queue 설계

알림 흐름은 `domain_events` topic exchange를 중심으로 구성됩니다.

```
OCR Worker
    │ detections.completed
    ▼
domain_events (topic exchange)
    │ routing key: detections.*
    ▼
alert_domain_events (queue)  ← Kombu Consumer 구독
    │ send_notification.delay()
    ▼
fcm_queue (queue)            ← Celery gevent Worker 소비
    │
    ▼
Firebase FCM
```

| Exchange | Type | Routing Key | 용도 |
|----------|------|-------------|------|
| `domain_events` | topic | `detections.completed` | Choreography 도메인 이벤트 |
| `fcm_exchange` | direct | `fcm` | FCM 태스크 라우팅 |
| `dlq_exchange` | fanout | - | Dead Letter 처리 |

| Queue | DLQ | Prefetch | 설명 |
|-------|:---:|:---:|------|
| `alert_domain_events` | ✅ | 1 | Kombu Consumer 전용 |
| `fcm_queue` | ✅ | 10 | gevent Worker (I/O 병렬) |
| `dlq_queue` | - | 1 | 최종 실패 메시지 보관 |

## FCM 푸시 전략

과속 감지 알림은 두 가지 채널로 동시 전송됩니다.

1. **토픽 브로드캐스트**: `dashboard_alerts` 토픽 구독자 전체에게 전송 (관제 대시보드)
2. **개별 푸시**: OCR로 인식된 번호판에 매핑된 차량 소유자의 FCM 토큰으로 직접 전송

## Dead Letter Queue 메커니즘

처리 실패한 메시지가 유실되지 않도록 DLQ를 운영합니다.

- `task_acks_late = True`: Worker 비정상 종료 시 메시지 재전달
- `task_reject_on_worker_lost = True`: Worker 소실 시 메시지 reject → DLQ 전달
- `ocr_queue`, `fcm_queue`, `alert_domain_events` 모두 DLQ 연결
- `dlq_queue`에 쌓인 메시지는 `process_dlq_message` 태스크로 별도 처리

---

<h4>reference</h4>

- System Design Interview (Alex Xu)
- <a href="https://youtu.be/uk5fRLUsBfk?si=n6--PVE2CH4pQt_4">분산 시스템에서 데이터를 전달하는 효율적인 방법 - nhn 김병부</a>

---

[< Back to Main README](../profile/README.md)
