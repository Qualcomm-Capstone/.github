# SpeedCam — 실시간 과속탐지 및 알림 시스템 with Qualcomm

<div align=center>
<img src="https://github.com/user-attachments/assets/cadc57b1-1806-4580-954d-ef898bd68f5f"/>

<h4>https://auto-notify.vercel.app/</h4>
<h2>SpeedCam — 실시간 과속탐지 및 알림 시스템</h2>
<h3>3-Service Event-Driven Architecture 기반 스마트 교통 시스템.<br>
Rubik Pi 엣지 디바이스에서 MQTT로 과속 감지 데이터를 전송하면,<br>
OCR 번호판 인식 → FCM 푸시 알림까지 자동 처리.<br>
Database per Service 패턴(4개 독립 DB) 적용.</h3>
</div>
<br />

## 📑 Contents
- [Demo](#demo)
- [System Architecture](#system-architecture)
- [Tech Stack](#tech-stack)
- [Notification System Design](#notification-system-design)
- [Rubik Pi 3](#rubik-pi-3)
- [API](#api)
- [Monitoring](#monitoring)
- [CI/CD](#ci-cd)
- [Member](#member)

<a id="demo"></a>
<h2>🖥️ Demo</h2>
<h3>시연 영상</h3>
https://www.youtube.com/watch?v=FDzbjOeika8

<h3>메인페이지(과속 차량 목록보기)</h3>
<img src="https://github.com/user-attachments/assets/812776f4-7dfe-4638-9ca1-0b6490662785" width="70%" >
<h3>과속 차량 개별 보기</h3>
<img src="https://github.com/user-attachments/assets/5c0a0853-0a12-4578-bc16-22947f388412" width="70%">
<h3>알림 확인하기</h3>
<img src="https://github.com/user-attachments/assets/c8415375-8a53-45be-b3bb-a0e9a2b20437" width="70%">

<br />
<br />

<a id="system-architecture"></a>
<h2>🏛️ System Architechture</h2>
<img width="1616" height="646" alt="image" src="https://github.com/user-attachments/assets/1215a1dc-a232-4837-b8c2-53ae3c9f7150" />

[자세히 보기](https://www.notion.so/2f33187fa1c980e1895cfef39b2c8ec7?pvs=21)

<blockquote>
3개 서비스(Main/OCR Worker/Alert Worker)로 구성된 Event-Driven Architecture. Rubik Pi가 MQTT로 과속 감지 데이터를 전송하면, Main Service가 수신 → OCR Worker가 번호판 인식 → Alert Worker가 FCM 푸시 알림을 전송한다. Database per Service 패턴(4개 독립 DB), Choreography 패턴, Dead Letter Queue를 적용.
<br /><br />
<a href="../docs/ARCHITECTURE-EVOLUTION.md">Architecture Evolution 상세 보기 →</a>
</blockquote>

<a id="tech-stack"></a>
<h2>🛠️ Tech Stack</h2>
<div align=center>
<h4>Frontend</h4>
<img src="https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=white">
<img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white">
<img src="https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white">
<img src="https://img.shields.io/badge/Prettier-F7B93E?style=for-the-badge&logo=prettier&logoColor=white">
<img src="https://img.shields.io/badge/Axios-5A29E4?style=for-the-badge&logo=Axios&logoColor=white">

<br />
<br />
<h4>Backend</h4>
<img src="https://img.shields.io/badge/Django-092E20?style=for-the-badge&logo=django&logoColor=white">
<img src="https://img.shields.io/badge/DRF-A30000?style=for-the-badge&logo=django&logoColor=white">
<img src="https://img.shields.io/badge/Gunicorn-499848?style=for-the-badge&logo=gunicorn&logoColor=white">
<img src="https://img.shields.io/badge/Celery-37814A?style=for-the-badge&logo=celery&logoColor=white">
<img src="https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white">
<img src="https://img.shields.io/badge/RabbitMQ-FF6600?style=for-the-badge&logo=rabbitmq&logoColor=white">
<img src="https://img.shields.io/badge/EasyOCR-4285F4?style=for-the-badge&logo=python&logoColor=white">


<br />
<br />
<h4>Infra</h4>
<img src="https://img.shields.io/badge/Google Cloud-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white"/>
<img src="https://img.shields.io/badge/Google Cloud Storage-AECBFA?style=for-the-badge&logo=googlecloudstorage&logoColor=white"/>
<img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white">
<img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white">
<img src="https://img.shields.io/badge/Traefik-24A1C1?style=for-the-badge&logo=traefikproxy&logoColor=white">
<img src="https://img.shields.io/badge/Portainer-13BEF9?style=for-the-badge&logo=portainer&logoColor=white">

<br />
<br />
<h4>Monitoring</h4>
<img src="https://img.shields.io/badge/OpenTelemetry-000000?style=for-the-badge&logo=opentelemetry&logoColor=white">
<img src="https://img.shields.io/badge/Jaeger-66CFE3?style=for-the-badge&logo=jaeger&logoColor=white">
<img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white">
<img src="https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white">
<img src="https://img.shields.io/badge/Loki-F46800?style=for-the-badge&logo=grafana&logoColor=white">

<br />
<br />
<h4>etc</h4>
<img src="https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white">
<img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white">
<img src="https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white">
<img src="https://img.shields.io/badge/Notion-000000?style=for-the-badge&logo=notion&logoColor=white">
<img src="https://img.shields.io/badge/Figma-F24E1E?style=for-the-badge&logo=figma&logoColor=white">
<img src="https://img.shields.io/badge/Postman-FF6C37?style=for-the-badge&logo=postman&logoColor=white">
<img src="https://img.shields.io/badge/Swagger-85EA2D?style=for-the-badge&logo=swagger&logoColor=white">
<br />
<br />
</div>

<br />

<a id="notification-system-design"></a>
<h2>Notification System Design</h2>

Choreography 패턴 기반 알림 시스템. OCR 완료 시 domain_events exchange에 이벤트를 발행하면, Kombu Consumer가 수신하여 Celery gevent Worker가 FCM 푸시를 전송한다. Dead Letter Queue와 Ack/Nack 메커니즘으로 메시지 유실을 방지.

> [Notification System Design 상세 보기 →](../docs/NOTIFICATION-SYSTEM-DESIGN.md)

<a id="rubik-pi-3"></a>
<h2>Rubik Pi 3</h2>

Qualcomm 기반 Rubik Pi에서 YOLO 객체 탐지와 GStreamer를 활용한 실시간 과속 차량 감지 엣지 시스템. 카메라 입력부터 추론, 트래킹, 속도 측정까지 모든 과정을 로컬에서 처리.

> [Rubik Pi Edge System 상세 보기 →](../docs/RUBIK-PI-EDGE-SYSTEM.md)

<br />

<a id="api"></a>
<h2>📁 API</h2>

Django REST Framework 기반 API. Vehicle CRUD, Detection 조회/통계, Notification 이력 등 15+ 엔드포인트 제공. Swagger/ReDoc 자동 문서화.

<h3>Swagger</h3>
<img src="https://github.com/user-attachments/assets/32f4389e-ef31-4ce1-b385-b369759a1f14">
<h3>Postman</h3>
<img src="https://github.com/user-attachments/assets/d319a2ae-ee50-462e-8224-131d4656af48">

<br />

<a id="monitoring"></a>
<h2>🔍 Monitoring</h2>

OpenTelemetry + Jaeger 분산 트레이싱, Prometheus + Grafana 메트릭 대시보드, Loki + Promtail 로그 수집, Flower Celery 모니터링, RabbitMQ Management 큐 대시보드로 구성된 풀스택 관측성.

<h3>Grafana Dashboard</h3>
<img src="../assets/monitoring/grafana-dashboard.png" alt="Grafana Dashboard" width="70%">

<h3>Jaeger Tracing</h3>

<!-- TODO: 수동 캡처 필요 — http://localhost:16686 에서 트레이스 검색 후 캡처 -->
<img src="../assets/monitoring/jaeger-tracing.png" alt="Jaeger Tracing" width="70%">

<h3>RabbitMQ Management</h3>
<img src="../assets/monitoring/rabbitmq-grafana.png" alt="RabbitMQ Queue Monitoring (Grafana)" width="70%">

<h3>Loki + Promtail Logs</h3>

<!-- TODO: 수동 캡처 필요 — Promtail 로그 수집 후 Grafana Explore에서 캡처 -->
<img src="../assets/monitoring/loki-logs.png" alt="Loki Logs" width="70%">

<br />

<a id="ci-cd"></a>
<h2>⚙️ CI/CD</h2>

GitHub Actions 기반 CI 파이프라인. Lint(flake8/black/isort), Test(pytest + MySQL), Docker Build(3개 이미지) 자동 검증. main 브랜치 push 시 GCP Artifact Registry에 이미지 배포.

<a id="member"></a>
<h2>Member</h2>

<table width="1000">
    <thead>
    </thead>
    <tbody>
    <tr>
        <th>Pictures</th>
         <td width="100" align="center">
            <a href="https://github.com/lsh1215">
                <img src="https://github.com/user-attachments/assets/5acaee97-6742-4f73-b47f-262cfdade814" width="80" height="80">
            </a>
        </td>
        <td width="100" align="center">
            <a href="https://github.com/Jminu">
                <img src="https://github.com/user-attachments/assets/27154e48-de61-4584-8309-3dab052b594d" width="80" height="80">
            </a>
        </td>
        <td width="100" align="center">
            <a href="https://github.com/choimh331">
                <img src="https://github.com/user-attachments/assets/aa6221bd-de3e-4926-8f98-ec0ecc1cbf72" width="80" height="80">
            </a>
        </td>
        <td width="100" align="center">
            <a href="https://github.com/Jeongchan-Seo">
                <img src="https://avatars.githubusercontent.com/u/147306987?v=4" width="80" height="80">
            </a>
        </td>
    </tr>
    <tr>
        <th>Name</th>
        <td width="100" align="center">이상훈</td>
        <td width="100" align="center">진민우</td>
        <td width="100" align="center">최명헌</td>
        <td width="100" align="center">서정찬</td>
    </tr>
    <tr>
        <th>Position</th>
        <td width="10" align="center">
            Leader<br>
            Backend<br>
            DevOps<br>
            Design<br/>
        </td>
        <td width="100" align="center">
            Rubik Pi<br>
            Tracking<br>
            Calculate<br>
            YOLO<br>
        </td>
        <td width="100" align="center">
            Backend<br>
        </td>
        <td width="100" align="center">
            Frontend<br>
        </td>
    </tr>
    <tr>
        <th>GitHub</th>
        <td width="100" align="center">
            <a href="https://github.com/lsh1215">
                <img src="http://img.shields.io/badge/lsh1215-green?style=social&logo=github"/>
            </a>
        </td>
        <td width="100" align="center">
            <a href="https://github.com/Jminu">
                <img src="http://img.shields.io/badge/Jminu-green?style=social&logo=github"/>
            </a>
        </td>
        <td width="100" align="center">
            <a href="https://github.com/choimh331">
                <img src="http://img.shields.io/badge/choimh331-green?style=social&logo=github"/>
            </a>
        </td>
        <td width="100" align="center">
            <a href="https://github.com/Jeongchan-Seo">
                <img src="http://img.shields.io/badge/JeongchanSeo-green?style=social&logo=github"/>
            </a>
        </td>
     </tr>
    </tbody>
</table>



<br />
<br />
