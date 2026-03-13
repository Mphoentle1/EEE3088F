# DOA Sensor Firmware Implementation Flow

This document outlines the high-level architecture and implementation flow for the Direction of Arrival (DOA) sensor firmware in C/C++.

## 1. Initialization Phase (Startup)

Before accessing peripheral registers, the system must be properly configured.

*   **System Clock & Power:** Enable the system clocks for specific peripherals (ADC, I2C, DMA, GPIO).
*   **GPIO Configuration:**
    *   Set ADC pins to **Analog Input**.
    *   Set I2C pins to **Alternate Function** (SDA/SCL) and configure pull-up resistors if necessary.
*   **I2C Initialization:** Configure the I2C peripheral (slave address, speed, interrupts). This must be done early so it is ready to receive read requests from the external computer at any time.

## 2. ADC Setup Phase

*   **Enable & Calibrate:** Enable the ADC peripheral. Execute a calibration routine immediately after power-on to ensure measurement accuracy.
*   **Configure ADC:** Set the resolution, sampling time, and trigger source.
*   **DMA Configuration (Highly Recommended):** For DOA calculations, high-speed sampling of multiple channels (microphones) is required. Configure a Direct Memory Access (DMA) controller to automatically transfer data from the ADC register to a buffer in memory. This frees the CPU from polling and reading every single sample.

## 3. The Main Execution Loop (State Machine)

*   **Start Sampling:** Trigger the ADC conversion to begin filling the DMA buffer.
*   **Wait for Completion:**
    *   Implement an **Interrupt Service Routine (ISR)** (e.g., `ADC_ConvCpltCallback` or a DMA transfer complete interrupt).
    *   Inside the interrupt, set a volatile flag (`dataReady = true`).
*   **Signal Processing:**
    *   In the main loop, check if `dataReady` is true.
    *   If true, reset the flag and perform the DOA mathematics (FFT, Phase difference calculation, etc.) on the sampled data buffer.
    *   Update a global structure or array with the calculated Angle, Phase, and the Student Number.
    *   *Note: Ensure this structure is updated atomically or double-buffered so the I2C interrupt doesn't read partially updated data.*

## 4. Communication (Asynchronous)

*   **I2C Handling:** Communication with the external computer runs asynchronously in the background via interrupts.
*   **I2C Interrupt Handler:** When the external computer requests a read:
    *   The I2C ISR accesses the global result buffer (Angle, Phase, Student Number).
    *   It transmits this data back to the master.
    *   If a calculation is currently modifying the data, ensure proper data locking/buffering logic is in place.
