# EEE3088F
DoA project

This project involves the design and implementation of a high-precision acoustic sensor capable of determining the Direction of Arrival (DoA) of a specific 2000 Hz beacon. The system integrates custom analog front-end hardware with a high-performance STM32F429ZGT6 microcontroller to process signals in real-time under strict latency constraints.

Technical Specifications

Target Frequency: 2000 Hz 
Sampling Rate: 20,000 Hz (5x oversampling for phase accuracy) 
Microphone Spacing: 50 mm
Supply Voltage: 3.3V DC (Strictly compliant with Automated Testing Rig) 
Communication: I2C Slave (Address: 0x42) 
Latency Ceiling: < 50ms from trigger to DoA result 

System Architecture

Analog Front-End
The signal chain uses a Zilltek ZTS6618 top-ported MEMS microphone for superior assembly reliability. The analog path includes:
AC Coupling: Blocks microphone DC offset.Active 
Bandpass Filter: A Multiple Feedback (MFB) topology using the AD8605 rail-to-rail op-amp, providing ~40dB of gain.
DC Biasing: Centering the signal at 1.65V for unipolar ADC sampling.
Digital ProcessingADC Acquisition: High-speed sampling triggered by the Rig state-machine.
DSP Algorithm: Cross-correlation (CCF) performed on the STM32's hardware Floating Point Unit (FPU) to calculate the time-of-flight difference between the microphone pair.
Register Interface: Results are reported via 32-bit IEEE 754 float registers.

📂 Repository Structure

/Hardware: KiCad PCB files, LTspice simulations, and Bill of Materials (BOM).
/Firmware: STM32CubeIDE project (C code), including CMSIS-DSP implementation.
/instruction_sheets: M1 Technical Contract and Physics Baseline proofs.
