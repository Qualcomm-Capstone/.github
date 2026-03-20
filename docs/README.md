# Documentation

Web Caps 프로젝트의 상세 기술 문서입니다.

## Documents

| Document | Description |
|----------|-------------|
| [Architecture Evolution](ARCHITECTURE-EVOLUTION.md) | 모놀리식 → EDA 전환 과정. Before/After 비교표, 3-서비스 모델, 6개 GCE 인스턴스 배포 구조, E2E 데이터 흐름, Database per Service 설계 |
| [Notification System Design](NOTIFICATION-SYSTEM-DESIGN.md) | Choreography 패턴 기반 알림 설계. Alert Worker 2-프로세스 구조(Kombu Consumer + Celery gevent), Exchange/Queue 설계, FCM 푸시 전략, Dead Letter Queue |
| [Rubik Pi Edge System](RUBIK-PI-EDGE-SYSTEM.md) | Qualcomm Rubik Pi 기반 엣지 과속 탐지 시스템. YOLO 추론, IoU 트래킹, 속도 측정, GCS 이미지 업로드 및 MQTT 이벤트 발행 |

---

[< Back to Main README](../profile/README.md)
