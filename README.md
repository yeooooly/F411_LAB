# F411_LAB

NUCLEO-F411RE 실습 프로젝트 (STM32CubeIDE) 와 실습 정리 문서

- 수업 자료 : [gotree94/STM32 / NUCLEO_F411RE](https://github.com/gotree94/STM32/tree/main/NUCLEO_F411RE)
- 이전 정리 (개발 환경, GPIO, HC-SR04 1 ~ 8장) : [yeooooly/STM32F411_Project](https://github.com/yeooooly/STM32F411_Project)

## 목차

- [01. HC-SR04 STEP02 거리측정2](./01_HC-SR04_STEP02_2) : TIM1 + EXTI 비차단 상태 머신, 5 Hz 측정 / 1 Hz 중앙값, UART `D,` / `C,` 출력, SWV 트레이스 (Timeline Graph, TRIG ↔ ECHO 타이밍)
- [02. DHT11](./02_DHT11) : 온습도 센서 1-Wire 통신, 시작 신호 / 40비트 데이터, 체크섬, PA4 (A2) 연결, UART 출력
- [03. FreeRTOS](./03_FreeRTOS) : CMSIS_V2, 초음파 / DHT 태스크 분리, UART mutex, 온습도 음속 보정, `D` / `C` / `TH` / `ENV` / `S` 출력, ECHO 파형

## 프로젝트 구조

```
F411_LAB/
├── 01_HC-SR04_STEP02_2/   HC-SR04 STEP02 거리측정2 문서
├── 02_DHT11/              DHT11 온습도 센서 문서
├── 03_FreeRTOS/           FreeRTOS 태스크 분리 문서
├── Core/                  사용자 코드 (main.c, ultrasonic.c 등)
├── Drivers/               HAL / CMSIS
├── Middlewares/           FreeRTOS (CubeMX 생성)
└── F411_LAB.ioc           CubeMX 설정
```

---

## HC-SR04 STEP02 거리측정2 요약

### 기능

- `HAL_Delay()` 없이 **비차단 상태 머신**으로 측정
- 200 ms 마다 측정 (5 Hz), 1초마다 최근 5개 **중앙값** 출력 (1 Hz)
- ECHO 펄스 폭은 **EXTI 양쪽 에지** + **TIM1 카운터** (1 tick = 1 µs) 로 측정
- B1 버튼 EXTI → LD2 토글
- SWV 트레이스 변수 (`g_trace_*`) 로 디버거에서 파형 확인

### 핀 배치

| 핀 | 기능 | 연결 |
|---|---|---|
| PA0 | ECHO (EXTI0, Rising/Falling) | HC-SR04 Echo → 1 kΩ / 2 kΩ 분압 후 |
| PA1 | TRIG (GPIO Output) | HC-SR04 Trig |
| PA2 / PA3 | USART2 TX / RX (115200) | ST-LINK 가상 COM |
| PA5 | LD2 | 보드 LED |
| PC13 | B1 (EXTI13, Falling) | 보드 파란 버튼 |
| PB3 | SWO | SWV 트레이스 |
| 5 V / GND | 센서 전원 | HC-SR04 VCC / GND |

### 설정

| 항목 | 값 |
|---|---|
| SYSCLK | 84 MHz (HSI → PLL) |
| TIM1 | Prescaler 84-1, Period 65535 → 1 MHz |
| 측정 타임아웃 | 35 ms |
| 유효 펄스 / 거리 | 100 ~ 30000 µs / 20 ~ 4000 mm |
| 음속 | 343 m/s |

거리(mm) = 펄스 폭(µs) × 3430 / 20000

### 시리얼 출력

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

### 파일 구조

```
F411_LAB/
├── Core/Inc/ultrasonic.h     초음파 API
├── Core/Src/ultrasonic.c     상태 머신, 중앙값 필터, 거리 계산
├── Core/Src/main.c           UART printf, Sensor_Task, EXTI 콜백
├── Core/Src/stm32f4xx_it.c   EXTI0 (ECHO), EXTI15_10 (B1) 핸들러
└── F411_LAB.ioc              CubeMX 설정
```
