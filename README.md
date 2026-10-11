# Embedded Portfolio

MCU와 Zynq SoC 위에서 **센서 입력부터 연산, 구동 출력까지** 연결한 임베디드 프로젝트 모음입니다.
각 프로젝트는 **문제와 원인 → 해결 과정 → 결과** 순서로 정리했습니다.

| 프로젝트 | 한 줄 요약 | 주요 기술 |
|---|---|---|
| [**STM32 3-MCU 센서·구동 분산 제어**](./stm32-distributed-control) | STM32 세 개에 센싱·판단·통신을 나누고 SPI로 연결한 제어 시스템 | C · STM32 레지스터·HAL · ADC/DMA · SPI · TIM PWM |
| [**Zynq 엣지 AI 영상 분류**](./zynq-edge-ai) | 센서가 움직임을 감지하면 PS–PL이 CNN을 나눠 추론하고 부저·서보를 구동하는 시스템 | C · FreeRTOS · AXI DMA · Vivado/Vitis · Arduino |

---

## 1. STM32 3-MCU 센서·구동 분산 제어

![STM32 architecture](stm32-distributed-control/docs/architecture.png)

| 핵심 결과 | 시연 |
|---|---|
| • ADC Polling → Circular DMA<br>센싱·필터 구간 **2,264 → 1,171 cycles (약 48% 감소)**<br><br>• 강수확률 60% 진입·40% 해제 히스테리시스로 모드 반복 전환 방지<br><br>• SPI Timeout 시 NSS 해제·버퍼 정리·SPI 재설정 후 제어 루프로 복귀<br><br>→ [자세히 보기](./stm32-distributed-control) | <img src="stm32-distributed-control/docs/demo.jpg" alt="STM32 demo" width="240"> |

## 2. Zynq 엣지 AI 영상 분류

![Zynq architecture](zynq-edge-ai/docs/architecture.png)

| 핵심 결과 | 시연 |
|---|---|
| • 학습 구조를 HW 합성곱 규칙에 맞추고 Q12로 변환, SW 재현 경로로 HW 결과 대조<br><br>• GAP 적용으로 가중치 헤더 **17.1MB → 1,513KB (−91%)**, 정확도 97% → 89% 감수<br><br>• FreeRTOS 수신·추론 Task 분리, 이중 버퍼, DMA 완료는 인터럽트·세마포어<br><br>→ [자세히 보기](./zynq-edge-ai) | <img src="zynq-edge-ai/docs/demo.jpg" alt="Zynq demo" width="300"> |

---

## 두 프로젝트에서 공통으로 다룬 문제

| 주제 | STM32 분산 제어 | Zynq 엣지 AI |
|---|---|---|
| **CPU 대기 줄이기** | ADC Polling → Circular DMA, DWT로 같은 구간 측정 | DMA 완료를 인터럽트·세마포어로 처리 |
| **전송 방식 선택** | 자주 반복되는 ADC는 DMA, 짧고 드문 SPI는 Polling + Timeout | 제어는 AXI4-Lite, 영상·Feature Map은 AXI DMA |
| **통신 오류 대응** | SPI Timeout 시 NSS 해제·버퍼 정리·SPI 재설정 | UART 프레임 CRC 검사, 재동기화, DMA Timeout·Reset |
| **자원 제약 안의 판단** | 임계값을 둘로 나눈 히스테리시스로 잦은 모드 전환 방지 | GAP로 가중치 91% 축소, 정확도 8%p 감수 |

## 저장소 구조

```
Embedded_Portfolio/
├── stm32-distributed-control/
│   ├── README.md
│   └── docs/            # 아키텍처, 시연 사진
└── zynq-edge-ai/
    ├── README.md
    └── docs/
```
