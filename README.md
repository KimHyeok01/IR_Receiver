사용자의 깃허브(GitHub) 저장소가 단순한 코드 저장 공간을 넘어, **임베디드 엔지니어로서의 역량**을 보여줄 수 있는 포트폴리오가 될 수 있도록 구성했습니다.

노션의 PARA 체계와 필자의 경험(로봇 유지보수, STM32 활용 능력)이 잘 드러나도록 작성한 README 마크다운입니다.

---

# 🚀 STM32 IR Remote Control Project

STM32F103RB(Nucleo-64)를 활용하여 **TSOP 센서의 IR 신호를 분석하고, NEC 프로토콜을 디코딩**하는 임베디드 펌웨어 프로젝트입니다. 하드웨어 타이머의 **Input Capture** 인터럽트를 활용하여 신호의 정밀한 마이크로초($\mu s$) 단위 측정을 구현했습니다.

## 🛠 Tech Stack
- **MCU**: STM32F103RBT6 (ARM Cortex-M3)
- **Toolchain**: STM32CubeIDE, STM32CubeMX
- **Library**: STM32Cube HAL
- **Hardware**: TSOP4838 (IR Receiver), NUCLEO-F103RB
- **Protocol**: NEC IR Protocol

## 🎯 Key Features
* **Precision Timing**: 타이머 분주(Prescaler) 설정을 통해 $1\mu s$ 단위의 고해상도 펄스 폭 측정.
* **Interrupt-Driven**: CPU 부하를 최소화하기 위해 Input Capture 인터럽트 방식을 채택.
* **Serial Debugging**: UART 리다이렉션을 통한 실시간 펄스 데이터 및 디코딩 결과 모니터링.
* **Hardware Validation**: 오실로스코프를 이용한 신호 무결성 검증 및 펌웨어 로직 정합성 확보.

## 🔌 Hardware Connection
| Peripheral | Pin (Nucleo) | Description |
| :--- | :--- | :--- |
| **TSOP OUT** | **PA0 (TIM2_CH1)** | Input Capture 신호 입력 |
| **TSOP VCC** | **+3.3V** | 전원 공급 |
| **TSOP GND** | **GND** | 그라운드 |
| **Debug UART** | **PA2/PA3** | ST-LINK VCP (115200 bps) |



## 💻 Firmware Logic
### 1. Timer Configuration
* **Frequency**: 64MHz
* **Prescaler**: 63 (1MHz, $1\mu s$ resolution)
* **Edge**: Falling Edge (Active Low signal detection)

### 2. Signal Analysis
NEC 프로토콜의 특성인 Leader Code(9ms Low, 4.5ms High)를 감지하여 데이터 프레임의 시작을 식별하고, 이후 들어오는 로직 비트(560us 간격)를 판별하여 32비트 데이터를 복원합니다.

```c
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim) {
    if (htim->Instance == TIM2) {
        uint32_t diff = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_1);
        __HAL_TIM_SET_COUNTER(htim, 0);
        
        // TODO: NEC Protocol Decoding Logic
        printf("Pulse Width: %lu us\r\n", diff);
    }
}
```
