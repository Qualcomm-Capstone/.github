# Rubik Pi 3 — Edge Computing System

> [< Back to README](../profile/README.md#rubik-pi-3)

Qualcomm 기반 Rubik Pi 하드웨어에서 YOLO 객체 탐지와 GStreamer를 활용해,
실시간으로 과속 차량을 감지하는 완전한 엣지 기반 시스템.
카메라 입력부터 추론, 트래킹, 속도 측정, 과속 차량 촬영까지 모든 과정을 로컬에서 처리하므로 클라우드 연산 불필요.

## Rubik Tech Stack

| Category             | Technologies                                                    |
|----------------------|-----------------------------------------------------------------|
| **Hardware**         | Rubik Pi 3, IMX477 image sensor, 10MP HQ Lens(16mm)             |
| **Object Detection** | YOLOv5m                                                         |
| **Acceleration**     | Qualcomm SNPE + TFLite delegate                                 |
| **Pipeline**         | GStreamer                                                       |
| **Programming**      | Python                                                          |
| **Features**         | On-device tracking, speed measurement, snapshot, multithreading |

## Object Tracking (IoU)

<img src="https://github.com/user-attachments/assets/868437d2-78e2-4d52-89a7-3d7f9e850517" width="300" height="200">

IoU를 계산하여, 다음프레임의 객체가 같은 객체인지 판단

## Speed Measurement

### Method 1 (Not Used)

<img src="https://github.com/user-attachments/assets/9980f43f-7990-47aa-a222-8e350e34666c" width="300" height="200">

프레임간 중심 좌표의 이동거리 변화로 속도를 측정

### Method 2 (✅Selected)

<img src="https://github.com/user-attachments/assets/e6d91e45-a950-47ad-8ef3-96aa008875cb" width="300" height="200">

가상의 두 선을 그어놓고, 두 선을 통과하는데 걸리는 시간을 측정

하지만, 이 방법은 가상의 두 선 사이의 실제 도로 거리를 알아야 정확히 측정 가능

## Multi Threading

병목 현상을 최소화 하기 위해서 멀티 스레딩을 사용

+ 메인 스레드
+ 트래킹, 속도 측정 스레드
+ 사진촬영 및 전송 스레드

## 백엔드 연동 방식

과속 차량 감지 후 다음 두 단계로 백엔드에 데이터를 전달합니다.

1. **이미지 업로드**: 과속 차량 스냅샷을 Google Cloud Storage에 업로드
2. **이벤트 발행**: MQTT `detections/new` 토픽에 감지 정보를 QoS 1로 발행

MQTT QoS 1(At-least-once)을 사용하므로, 네트워크 불안정 시에도 메시지가 백엔드에 최소 한 번 이상 전달됩니다. 백엔드의 RabbitMQ MQTT Plugin이 이 메시지를 수신하여 처리 파이프라인을 시작합니다.
