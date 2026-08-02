# Modbus in Practice — Homelab Writeup Series

## What is Modbus?

Modbus is a protocol designed for exchanging process data between industrial control systems. It primarily functions on L7 of OSI (the Application Layer). The protocol consists of a set of digital codes intended to read/write data to and from industrial devices. An industrial device can be made Modbus-compliant by being programmed to understand and respond to these codes appropriately.

### Without Modbus

A PLC-controlled motor system that does not employ Modbus has the following functionality. The PLC sends individually-wired Forward, Reverse, Stop, and speed-control command signals to a variable-frequency drive which then sends power of different frequencies to an electric motor.

### With Modbus

By using appropriate Modbus commands transmitted to the VFD, the PLC can issue the same four commands with fewer wires. Not only can it send the same commands, it can also read data from the VFD that it couldn't do before. An example is if the VFD provides a memory location for the storage of fault codes. The PLC can be programmed to read a single register from that memory location, and thus monitor the status of the VFD.

Another advantage of Modbus is that it is designed to address multiple devices on the same network. The PLC is not limited then to controlling one motor, but up to 247 separate Modbus devices on the same communication cable.

Each VFD in that long chain can be given its own Modbus network 'slave address', so that the PLC can distinguish between them when communicating on the same wire pair.

The disadvantage Modbus has is a decrease in speed and reliability for sensing and control functions. Modbus is slower than dedicated wire control because the PLC cannot issue different commands on the network at the same time. For example, if the PLC needed to tell a VFD to turn its motor in one direction at a certain RPM, the Modbus-based system would have to issue two separate Modbus codes, whereas the wired system could issue these commands at once.

There are two different Modbus formats, ASCII and RTU, and they each have their own message frame formats. In Modbus ASCII mode, all slave device addresses, function codes, and data are represented with ASCII characters which can be read directly by terminal programs. The advantage of ASCII is the readability for troubleshooting. RTU mode expresses all data in raw binary form. RTU is faster than ASCII because of the raw binary frame, only requiring about half the bits as ASCII frames.

### Modbus Function Codes

| Code | Function |
|------|----------|
| 01 | Read one or more PLC output "coils" (1 bit each) |
| 02 | Read one or more PLC input "contacts" (1 bit each) |
| 03 | Read one or more PLC "holding" registers (16 bits each) |
| 04 | Read one or more PLC analog input registers (16 bits each) |
| 05 | Write (force) a single PLC output "coil" (1 bit) |
| 06 | Write (preset) a single PLC "holding" register (16 bits) |
| 15 | Write (force) multiple PLC output "coils" (1 bit each) |
| 16 | Write (preset) multiple PLC "holding" registers (16 bits each) |

### Modbus 984 Address Ranges

| Codes | Address Range | Purpose |
|-------|--------------|---------|
| 01, 05, 15 | 00001–09999 | Discrete outputs ("coils"), read/write |
| 02 | 10001–19999 | Discrete inputs ("contacts"), read only |
| 03, 06, 16 | 40001–49999 | Holding registers, read/write |
| 04 | 30001–39999 | Analog input registers, read only |

---

## Modbus in Practice

Okay, Modbus exists, it allows for more scalable communication with industrial control systems. On paper it has function codes, functional address ranges, modes, and applications. Yet, how is it seen in day-to-day operations in the real-world? How is it implemented? And what tools often use it to communicate?

In practice, Modbus is used in building automation for networking field devices and sensors. But, that is a broad definition for its application. To better visualize its utility, here are four specific examples of where it is used:

1. **Manufacturing floors:** PLCs controlling assembly lines, conveyor belts, and CNC machines communicate with SCADA/HMI systems over Modbus to report status and receive commands.
2. **Building automation:** HVAC controllers, lighting panels, and energy meters use Modbus to network with building management systems.
3. **Energy and utilities:** Power substations, water treatment plants, and oil pipelines use Modbus for sensor polling and actuator control.
4. **Process industries:** Chemical plants, refineries, and pharmaceutical manufacturing use Modbus to read temperature, pressure, and flow sensors in real-time.

The typical Modbus conversation occurs between a SCADA/HMI system (the client) polling a PLC or RTU (the server) on a fixed interval. These intervals could be every 500ms, every second, every 5 seconds. The HMI polls "what's the value of this specific register" and the PLC responds. That function code, represented by 03 in this case, could occur thousands of times a day and comprise the majority of the traffic.

In my homelab, this is simulated by ScadaBR (the client) and OpenPLC (the server). ScadaBR polls registers over Modbus TCP on port 502, and then displays the value on the HMI dashboard. This deterministic and repeatable process is a defining characteristic of ICS, and it makes anomaly detection possible, where looking for deviations from that deterministic process is the whole picture.
