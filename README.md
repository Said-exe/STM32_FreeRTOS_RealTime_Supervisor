# STM32 FreeRTOS Real-Time Supervisor

## Overview

Real-time multi-tasking supervision system developed on an STM32F446RE using FreeRTOS and STM32CubeIDE.

The project implements a small embedded architecture where one task performs a mathematical computation, another task processes and displays the result, and a third task provides system monitoring triggered by a hardware button interrupt.

The objective is to practice and demonstrate FreeRTOS concepts in a concrete embedded system.

## Hardware

- NUCLEO-F446RE
- STM32F446RE
- User push button on PC13
- No external sensors or peripherals required

## Software

- STM32CubeIDE 1.19.0
- STM32 HAL
- FreeRTOS
- CMSIS-RTOS2
- C

## Architecture

```text
                    ┌─────────────────────┐
                    │       Task 1        │
                    │  sqrt(n) calculation│
                    └──────────┬──────────┘
                               │
                         Message Queue
                               │
                               ▼
                    ┌─────────────────────┐
                    │       Task 2        │
                    │ Result processing   │
                    │      + printf       │
                    └─────────────────────┘


User Button PC13
       │
       ▼
     EXTI
       │
       ▼
   Semaphore
       │
       ▼
┌─────────────────────┐
│       Task 3        │
│ System Monitoring   │
└─────────────────────┘
```

## FreeRTOS Concepts Implemented

### Tasks

Three independent tasks with different priorities:

- **Task1:** performs the square-root calculation.
- **Task2:** receives and displays the calculation result.
- **Task3:** performs system monitoring when the user button is pressed.

### Message Queue

Task1 sends the calculation result to Task2 using a FreeRTOS message queue.

```text
Task1 → Message Queue → Task2
```

The queue transfers a structure containing:

- input value
- calculated square root

### Binary Semaphore

The user button on PC13 generates an EXTI interrupt.

The interrupt releases a binary semaphore, allowing Task3 to process the event.

```text
PC13 → EXTI → Semaphore → Task3
```

### Mutex

A mutex protects the shared `printf()` resource and prevents concurrent access from multiple tasks.

### Interrupt Handling

The STM32 EXTI interrupt mechanism is used to detect the user button event.

The interrupt callback remains short and only signals the corresponding FreeRTOS synchronization object.

### System Monitoring

Task3 reports:

- task states
- message queue occupancy
- remaining stack margin of each task

Example:

```text
========== SYSTEM MONITOR ==========
Task1 state : ...
Task2 state : ...
Task3 state : ...
Queue : 0 / 5
Task1 stack free : ...
Task2 stack free : ...
Task3 stack free : ...
====================================
```

## Real-Time Architecture

The tasks use blocking RTOS primitives rather than continuously polling shared resources.

For example, Task2 waits for a new queue message:

```c
osMessageQueueGet(..., osWaitForever);
```

and Task3 waits for a button event:

```c
osSemaphoreAcquire(..., osWaitForever);
```

This avoids unnecessary CPU usage while tasks are waiting for events.

## Key Learning Outcomes

- FreeRTOS task creation and scheduling
- Task priorities
- Inter-task communication
- Message queues
- Binary semaphores
- Mutexes
- Interrupt-driven event handling
- CMSIS-RTOS2 API
- Stack usage monitoring
- Real-time embedded software architecture
- Debugging with SWV/ITM

## Development Environment

Developed with:

- STM32CubeIDE 1.19.0
- STM32CubeMX
- STM32F446RE
- FreeRTOS / CMSIS-RTOS2
- SWV / ITM for runtime debugging

## Author

Said Toumi  
Embedded Systems / Firmware Engineer