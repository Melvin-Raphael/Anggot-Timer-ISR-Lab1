

## Laboratory Activity 1: Timers, Interrupts, and Task Scheduling

**Course:** BCA143 Firmware Programming

**Student:** Melvin Raphael R. Anggot

**Date:** 06/14/2026

### Project Description

This project implements timer-based interrupts and a cyclic executive scheduler on the STM32F407ZGT6 microcontroller (RT-Thread RT-Spark board). It utilizes the STM32F4 HAL drivers.

### Hardware

* **Board:** RT-Thread RT-Spark Development Board (STM32F407ZGT6)
* **LEDs:** PF11 (Red), PF12 (Blue)
* **User Button:** PA1 (Configured for EXTI1 rising edge interrupt)
* **Debug Pin:** PE0

### Features

* **TIM2:** 1 Hz periodic interrupt (high priority), configured with a prescaler of 8399 and a period of 9999.
* **TIM3:** 2 Hz periodic interrupt (medium priority), configured with a prescaler of 8399 and a period of 4999.
* **External Interrupt:** Triggered on button press via PA1 (EXTI1), immediately toggling both the Red and Blue LEDs.
* **Scheduler:** Foreground/background system architecture using flag-based polling (`task_adc_ready` and `task_display_ready`).
* **Timing Measurement:** Uses the DWT (Data Watchpoint and Trace) Cycle Counter to accurately measure execution latency (`DWT->CYCCNT`).

### Timing Measurements

| Task | Simulated Delay | Trigger | Variable Flag |
| --- | --- | --- | --- |
| ADC Processing | 50 ms | TIM2 | `task_adc_ready` |
| Display Update | 20 ms | TIM3 | `task_display_ready` |

*(Note: Maximum latency for the ADC task is actively tracked in the `maxlatency` variable. Insert your live timing measurement table here.)*

### Sequence Diagram

HandUML Sequence Diagram:
@startuml
participant "Main_Loop\n(Foreground)" as Main
participant "TIM2\n(Hardware)" as TIM2
participant "TIM3\n(Hardware)" as TIM3
participant "ISR_Handler\n(Background)" as ISR
participant "LEDs\n(GPIO)" as LEDs

Main -> TIM2 : Start Timer (IT)
Main -> TIM3 : Start Timer (IT)

loop [Infinite Loop (Cyclic Executive)]
    note over Main
        Polling flags...
    end note

    opt [Timer 2 Trigger]
        TIM2 --> ISR : Interrupt
        activate ISR
        ISR -> ISR : Set task1_flag = 1
        ISR --> Main : Return
        deactivate ISR
    end

    opt [Task 1 Execution]
        Main -> Main : Clear task1_flag
        Main -> LEDs : Toggle LED1
    end

    opt [Timer 3 Trigger]
        TIM3 --> ISR : Interrupt
        activate ISR
        ISR -> ISR : Set task2_flag = 1
        ISR --> Main : Return
        deactivate ISR
    end

    opt [Task 2 Execution]
        Main -> Main : Clear task2_flag
        Main -> LEDs : Toggle LED2
    end
end
@enduml

### Build Instructions

1. Open project in STM32CubeIDE.
2. Build: **Project → Build All**.
3. Connect RT-Spark via USB.
4. Upload: **Run → Debug** or **Run**.

### Analysis

Analysis Questions: 
1. What is the maximum latency for Task A in your system? 
Answer: Task A’s Maximum Latency is 0.0165 ms. This is the delay from the interrupt up to the actual execution of Task A.
2. If Task B is running when TIM2 interrupt occurs, how does it affect TLatency(Task A)? 
Answer: TLatency for Task A increases when that happens on this kind of a system
3. Calculate worst-case TResponse for Task A if all other tasks are running 
Answer: TResponse = TLatency + TTask = 0.0165+50 = 50.0165 ms 
        TResponse(When other tasks are 30ms total) = 30+50 = 80ms
4. How would response time change with a preemptive scheduler? 
Answer: TResponse = TISR + TTask = 0.0093+50 = 50.0093 ms 

### References

1. Castor, P. R. P. (2025). Software Design Basics [Lecture 2]. BCA143 Firmware Programming, MSU-IIT.
2. STMicroelectronics. (2024). STM32F4 HAL Driver User Manual.
