# Distributed Smart Parking Management System (ATmega128)

## 1. Project Overview
This project implements a multi-node, real-time smart parking management system using three ATmega128 microcontrollers. Designed with a Master-Slave distributed architecture, the system automates vehicle entry detection, parking slot allocation, parking duration tracking, and fee calculation. Communication between nodes is handled via a custom asynchronous UART protocol, ensuring synchronized state management across the parking lot facility.

## 2. System Architecture & Key Features
* **Master MCU (Central Hub):** Manages the global parking state, records entry/exit timestamps using Timer Interrupts (RTC simulation), calculates parking fees, and broadcasts real-time available slot data to slaves.
* **Slave A MCU (Entry Node):** Utilizes ultrasonic sensors to detect incoming vehicles. Interfaces with a 4x4 matrix keypad to capture the 4-digit vehicle ID, displays inputs on a 7-segment display, and visualizes the available parking slots using an 8-LED array.
* **Slave B MCU (Exit Node):** Detects exiting vehicles, captures the vehicle ID, and requests fee calculation from the Master. Displays the parking duration and calculated fee on a 16x2 LCD, processing the final "payment" signal to free up the parking slot.
* **Custom UART Protocol:** Defined a strict communication protocol using fixed-length payloads and newline (`\n`) delimiters to ensure robust data framing between the Master and Slaves.

## 3. Critical Troubleshooting Log (Deep Dive)

### Issue: Asynchronous Event Handling in Blocking Loops (Data Synchronization Failure)
* **Symptom:** When a vehicle exited via Slave B and the Master updated the parking state, Slave A failed to immediately reflect the new available slot on its LED/LCD interface.
* **Root Cause Analysis:** Verified that the Master was transmitting the correct ASCII payload and Slave A's UART RX Interrupt Service Routine (ISR) was successfully receiving it and setting the `parking_state_updated` flag. The core issue lay in the main execution flow: Slave A's main function was trapped in nested, blocking polling loops (e.g., `while(is_full())` or waiting for keypad inputs). Although the ISR fired, the main loop could not evaluate the updated flag until the current blocking routine naturally terminated.
* **Resolution:** Refactored the rigid polling architecture into a preemptive flag-checking system. Injected `if (parking_state_updated) break;` conditions into all nested blocking loops. This forced the main execution thread to immediately exit any idle or polling state upon receiving a UART interrupt, allowing it to jump back to the top-level state machine to refresh the LCD and LED interfaces in real-time.

## 4. Future Architecture Upgrade
* **Hardware Expansion:** Integration of DC motors with PWM control to simulate physical barrier gates upon successful entry/payment.
* **Data Integrity:** Implementation of Parity bit checks for UART packets to prevent communication errors, and adding an EEPROM-based backup system to prevent data loss of parked vehicle IDs during power failures.

## 5. Result
🔗https://youtube.com/shorts/0pGOZzgGX28?feature=share
------
ATmega128 기반 분산형 스마트 주차 관제 시스템 구축

1. 프로젝트 개요
개발 환경: C, Microchip Studio, ATmega128 (Master 1, Slave 2) 
핵심 내용: 단일 칩 제어를 넘어, 3개의 독립적인 마이크로컨트롤러가 Master-Slave 구조로 협력하는 분산 제어 시스템 설계. 입차(Slave A), 출차(Slave B), 중앙 관제(Master)로 역할을 분리하고, 커스텀 UART 통신 프로토콜을 통해 실시간 주차 현황 업데이트, 타이머 인터럽트 기반 시간 계산 및 요금 정산 시스템을 구현함.

2. 핵심 엔지니어링 역량 (Troubleshooting)
① UART 인터럽트(ISR)와 메인 블로킹(Blocking) 루프 간의 충돌 해결
* 현상: 출차(Slave B)가 완료되어 Master가 빈자리 데이터를 브로드캐스팅했음에도, 입차 시스템(Slave A)의 LCD 및 LED에 잔여 주차 공간이 즉각적으로 갱신되지 않는 동기화 지연 문제 발생.
* 원인 분석: 통신 파형 및 수신 버퍼 검증 결과, Slave A의 RX ISR은 정상 동작하여 수신 플래그(parking_state_updated)를 즉시 Set 하였음. 문제는 Slave A의 Main 제어 흐름이 초음파 센서 대기(while(1))나 만차 대기(while(is_full())) 등 무한 루프에 갇혀(Blocking) 있어, 루프가 끝날 때까지 수신 플래그를 확인하지 못하는 소프트웨어 구조적 한계에 있었음.

* 해결: 시스템의 모든 Polling 대기 루프 내부에 수신 플래그 확인 로직(if (parking_state_updated) break;)을 삽입하는 Preemptive(선점형) 구조로 리팩토링 수행. 비동기적 통신 이벤트 발생 즉시 모든 대기 상태를 강제 종료하고 최상위 상태 머신으로 복귀하게 만들어, 완벽한 실시간 데이터 동기화를 달성함.

3. 시스템 통합 및 통신 아키텍처 설계
통신 프로토콜 직접 설계: 데이터 파편화를 막기 위해 [데이터 본문] + [\n] 형태의 고정/가변 프레임을 직접 정의하여 안정적인 비동기 UART 패킷 통신 구현.

하드웨어 인터페이스 제어: Timer/Counter를 활용한 초음파 센서 펄스 거리 측정 , Key Matrix 디바운싱 및 디코딩 제어 , Multi-digit 7-Segment 동적 디스플레이 구동 등 펌웨어 최하단(Register) 레벨의 직접 제어 역량 확보.

4. 향후 아키텍처 고도화 (Scalability)
UART 통신 시 노이즈로 인한 데이터 오염을 방지하기 위해 패리티 비트(Parity Bit) 에러 체크 로직 도입 및 I2C 통신 방식으로의 확장.
정전 시 데이터 보존을 위해 현재 RAM에 기록되는 차량 입차 시간과 ID 정보를 내부 EEPROM 메모리로 이관하는 비휘발성 저장 시스템 추가 계획
