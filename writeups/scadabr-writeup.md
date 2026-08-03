# ScadaBR in Practice — Homelab Writeup Series

## What is SCADA and ScadaBR?

SCADA stands for Supervisory Control and Data Acquisition. SCADA systems serve as an interface between processes from the manufacturing floor and an operator. SCADA systems can be simple sensing applications, or can control panels in entire power plants.

A typical SCADA system needs to have communication drivers with connected equipment, a system for data recording/logging, and a GUI known as an HMI, Human Machine Interface. Common SCADA functions include the following:

- Generation of charts and reports
- Detection of alarms
- Automated event logging
- Process control
- Drive and control equipment
- Use scripting to develop logical automation

So what is ScadaBR? ScadaBR is an open source, free software that provides an HMI to monitor and control industrial processes, equipment, and sensors remotely via Modbus/DNP3 protocols. ScadaBR is just one software implementation of SCADA, and big corporations often use more industry-dominant platforms like AVEVA System Platform and Ignition by Inductive Automation.

---

## How will ScadaBR be used in my homelab?

ScadaBR will operate on a VM and provide an HMI for data tracking and control. It will communicate over port 502, Modbus TCP, with the OpenPLC VM to collect data. The HMI is web-based and will need to be configured alongside Modbus to assign registers to poll and how often.

---

## What is SCADA cybersecurity?

SCADA systems provide organizations with advantages in cost reduction, flexibility, and performance efficiency. However, threats have grown against these SCADA systems due to increased remote access and internet connectivity. Hacks to these systems can result in adversaries gaining control of a city water supply system, shutting down electricity, or causing malfunctions in nuclear reactors.

These are some of the current challenges to SCADA cybersecurity:

- Legacy Systems
- IT/OT convergence
- Remote Access
- Regulatory Requirements

Here are some of the best practices that can address these challenges:

- Gain visibility into your ICS environment
- Integrate existing IT tools and workflow with your ICS
- Extend IT security controls and governance to ICS environment
