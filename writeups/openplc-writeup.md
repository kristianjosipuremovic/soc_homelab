# OpenPLC in Practice — Homelab Writeup Series

The writeup will consist of three parts: what is OpenPLC, what is it simulating, and how is it implemented in my homelab?

---

## What is OpenPLC?

OpenPLC is an open-source Programmable Logic Controller based on easy-to-use software and is mainly used in industrial and home automation, IoT, and SCADA research.

OpenPLC consists of two services: Runtime and Editor. Runtime is a portable software designed to run from small microcontrollers to powerful servers in the cloud, and is responsible for executing PLC programs created with Editor. Editor is the software that runs on your computer and creates the PLC programs.

---

## What is a PLC, and how does it fit in with the field of OT security as a whole?

PLCs are industrial computers with inputs and outputs used to control and monitor industrial equipment with custom programming.

The operation of a PLC is broken down into three stages: inputs, program execution, and outputs. PLCs take in data from the plant floor by monitoring inputs from connected machines. The inputs are checked against the program logic which changes the outputs to connected output devices. For example, a valve position sensor can be connected to the inputs with the control of the valve position connected to the outputs. A custom program could read the valve position, check where it needs to be, and then move the valve position with the output.

PLC programs work in cycles. The PLC detects the state of connected input devices, executes the custom program, then uses the inputs to determine output state. After completing the cycle, the PLC then does diagnostic safety checks and restarts the cycle once completed, starting again with checking inputs.

There are two main categories of PLCs, Fixed and Modular. Fixed PLCs are the most common, and happen to be smaller and cheaper than modular PLCs. This makes them useful for smaller control systems or simpler tasks. One downside is they have less memory than modular PLCs, making them harder to repair and modify. Modular PLCs are more scalable and customizable, yet larger and more expensive.

PLCs are beneficial to ICS because of their flexibility. They allow the changing of custom programs without manual rewriting. They don't have moving parts and are built to keep working in harsh environments. They provide more control and monitoring to admins and can manage multiple I/Os at once.

PLCs are used in SCADA and HMI systems. SCADA/HMI systems are used for the consolidation of data from the manufacturing floor. PLCs act as physical interfaces between devices on the plant and a SCADA/HMI system. They can communicate, monitor, and control processes.

PLCs participate in IIoT with the traditional poll-response method. Yet there are also solutions such as MQTT which employ a publish-subscribe protocol that can improve communications from the network edge.

---

## How is OpenPLC simulated in my homelab?

Just as described above, OpenPLC will be simulated in my homelab using VMs simulating PLCs and SCADA/HMI systems. OpenPLC on one VM will communicate through Modbus to the ScadaBR VM. This simulates a mini manufacturing floor, with PLCs sending data to an organized SCADA dashboard.
