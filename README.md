# F411_LAB

> **현재 main.c = DHT11 온습도 측정** (A2 = PA4) → 정리 문서 : [STM32F411_Project / 03. DHT11](https://github.com/yeooooly/STM32F411_Project/tree/main/03_DHT11_DOC)
>
> 아래 HC-SR04 내용은 이전 단계 (`ultrasonic.c` 는 그대로 남아 있음)

NUCLEO-F411RE 로 HC-SR04 초음파 센서 거리를 측정하는 STM32CubeIDE 프로젝트

- 수업 자료 : [gotree94/STM32 STEP02_거리측정2 (NUCLEO-F411RE)](https://github.com/gotree94/STM32/blob/main/NUCLEO_F411RE/00.F411_TASK_LAB/STEP02_%EA%B1%B0%EB%A6%AC%EC%B8%A1%EC%A0%952.md)
- 정리 문서 : [STM32F411_Project / 02. HC-SR04 / 9. STEP02 거리측정2](https://github.com/yeooooly/STM32F411_Project/tree/main/02_HC-SR04_DOC#9-실습--step02-거리측정2-tim1--중앙값--디버그-출력)

## 기능

- `HAL_Delay()` 없이 **비차단 상태 머신**으로 측정
- 200 ms 마다 측정 (5 Hz), 1초마다 최근 5개 **중앙값** 출력 (1 Hz)
- ECHO 펄스 폭은 **EXTI 양쪽 에지** + **TIM1 카운터** (1 tick = 1 µs) 로 측정
- B1 버튼 EXTI → LD2 토글
- SWV 트레이스 변수 (`g_trace_*`) 로 디버거에서 파형 확인

## 핀 배치

| 핀 | 기능 | 연결 |
|---|---|---|
| PA0 | ECHO (EXTI0, Rising/Falling) | HC-SR04 Echo → 1 kΩ / 2 kΩ 분압 후 |
| PA1 | TRIG (GPIO Output) | HC-SR04 Trig |
| PA2 / PA3 | USART2 TX / RX (115200) | ST-LINK 가상 COM |
| PA5 | LD2 | 보드 LED |
| PC13 | B1 (EXTI13, Falling) | 보드 파란 버튼 |
| PB3 | SWO | SWV 트레이스 |
| 5 V / GND | 센서 전원 | HC-SR04 VCC / GND |

## 설정

| 항목 | 값 |
|---|---|
| SYSCLK | 84 MHz (HSI → PLL) |
| TIM1 | Prescaler 84-1, Period 65535 → 1 MHz |
| 측정 타임아웃 | 35 ms |
| 유효 펄스 / 거리 | 100 ~ 30000 µs / 20 ~ 4000 mm |
| 음속 | 343 m/s |

거리(mm) = 펄스 폭(µs) × 3430 / 20000

## 시리얼 출력

```text
=== F411_TASK_LAB Step 2: Ultrasonic ===
HC-SR04: TRIG=PA1, ECHO=PA0, 5 Hz sample, 1 Hz median
TIM1: PCLK2=84000000Hz divider=84 ARR=65535 tick=1000000Hz
D,1785000,1782,1,7
C,10401
```

| 행 | 의미 |
|---|---|
| `D` | 시각(ms), 중앙값 거리(mm), 유효 여부, 누적 오류 수 |
| `C` | 마지막 ECHO 펄스 폭 (TIM1 count = µs) |

## 파일 구조

```
F411_LAB/
├── Core/Inc/ultrasonic.h     초음파 API
├── Core/Src/ultrasonic.c     상태 머신, 중앙값 필터, 거리 계산
├── Core/Src/main.c           UART printf, Sensor_Task, EXTI 콜백
├── Core/Src/stm32f4xx_it.c   EXTI0 (ECHO), EXTI15_10 (B1) 핸들러
└── F411_LAB.ioc              CubeMX 설정
```
