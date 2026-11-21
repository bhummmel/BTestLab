
# SDR and RF Experimentation

## Overview

The SDR and RF Experimentation project focused on investigating radio-frequency communications, software-defined radio (SDR) technology and the integration of radio equipment with Linux-based computing infrastructure.

The project combined amateur radio experimentation, conventional radio hardware, signal analysis and the evaluation of software-defined receivers for use in both stationary and mobile computing environments.

A central objective was to understand how radio signals are generated, transmitted, received and processed, and how software-defined radio can extend the capabilities of conventional communications equipment.

The work included investigating VHF amateur radio operation using a Baofeng UV-5R, evaluating legacy radio equipment such as the Regency RH606B and exploring SDR applications for a vehicle-mounted Linux computer.

The project also drew upon existing electronics troubleshooting experience, including the use of oscilloscopes and other bench instrumentation.

Rather than focusing on a single completed radio installation, the project documents several related technical investigations and the development of an expandable RF experimentation platform.

---

## Objectives

The primary objective was to develop practical knowledge of radio communications and software-defined signal processing.

Specific goals included:

- Understanding the fundamentals of VHF and UHF radio communication.
- Experimenting with licensed amateur radio operation.
- Evaluating traditional analog radio equipment.
- Investigating software-defined radio receivers.
- Understanding frequency selection, modulation and bandwidth.
- Exploring RF signal capture and analysis on Linux.
- Evaluating antennas and feed-line considerations.
- Investigating integration with a vehicle-mounted computer.
- Developing repeatable methods for diagnosing reception and transmission problems.
- Identifying opportunities for integrating radio telemetry into the homelab.

An additional objective was to compare the capabilities and limitations of dedicated radio hardware against general-purpose software-defined receivers.

---

## Equipment and Infrastructure

The project involved existing radio equipment and computing systems, alongside additional SDR hardware considered for future use.

| Component | Role |
|---|---|
| Baofeng UV-5R | Handheld VHF/UHF amateur transceiver |
| Regency RH606B | Legacy radio equipment evaluated |
| Dell Latitude 5424 Rugged | Linux-based portable computing platform |
| Logic Supply RT130-i7-P | Vehicle computer evaluated for radio applications |
| Linux | Operating environment for SDR software |
| Tektronix 2235 | Analog oscilloscope for electronics experimentation |
| Rigol DS1054 | Digital oscilloscope used in electronics diagnostics |
| SDR receiver | Proposed RF capture and analysis hardware |
| Antennas and feed lines | RF signal reception and transmission components |
| Unraid | Potential centralized storage and processing platform |

The environment provides a foundation for both conventional amateur radio operation and future computer-based signal processing.

Not all proposed SDR hardware or software integrations were confirmed as deployed.

---

## Radio Communications Fundamentals

Radio communication uses electromagnetic waves to transfer information between a transmitter and receiver.

The transmitted signal occupies a particular frequency range and uses a modulation technique to encode information.

Important parameters include carrier frequency, bandwidth, modulation type, transmit power and receiver sensitivity.

For example, a conventional analog FM voice transmission uses frequency modulation to encode audio information.

The receiver must be tuned to the appropriate frequency and configured to demodulate the signal.

Successful reception also depends on antenna characteristics, signal strength, interference and propagation conditions.

The project emphasized understanding these fundamentals before introducing more complex SDR software.

---

## Amateur Radio Experimentation

The project included investigating amateur radio operation using a Baofeng UV-5R handheld transceiver.

The UV-5R supports analog FM operation on commonly used VHF and UHF amateur radio bands, subject to the capabilities and regulatory status of the specific device.

The experimentation focused on understanding how to configure a radio for basic communication and how to perform an appropriate transmission test.

Relevant operating concepts included:

- Frequency selection.
- Simplex versus repeater communication.
- Transmit and receive offsets.
- CTCSS and DCS signaling.
- Squelch adjustment.
- Transmit power.
- Channel programming.
- Station identification.

A central distinction was the difference between transmitting directly to another station and communicating through a repeater.

Simplex operation uses the same frequency for transmission and reception.

Repeater operation typically requires a configured frequency offset and may require an access tone.

These concepts are foundational to reliable amateur radio operation.

---

## Amateur Radio Licensing and Operating Practices

The project was conducted in the context of Technician-class amateur radio privileges.

In the United States, amateur radio operation is regulated by the Federal Communications Commission under Part 97.

Authorized operating frequencies, emission types and power limits depend on the operator's license privileges and the applicable rules.

Radio experimentation must therefore distinguish between receiving signals and transmitting them.

Reception is generally less restrictive, although some communications and uses of received information are subject to legal limitations.

Transmission requires appropriate authorization.

The project emphasized the importance of verifying operating privileges and ensuring that radio equipment is configured correctly before transmitting.

Proper station identification and avoidance of harmful interference are also essential operating practices.

---

## VHF and UHF Propagation

VHF and UHF radio signals are influenced by terrain, antenna placement, obstructions and propagation conditions.

For many terrestrial communications applications, these frequencies behave predominantly as line-of-sight signals.

However, reflections, diffraction and atmospheric effects can extend or complicate reception.

Antenna height can have a substantial effect on usable communication range.

Urban structures, vegetation and terrain may attenuate or reflect signals.

The project considered these factors when evaluating handheld radio performance and future mobile radio installations.

It reinforced the importance of distinguishing equipment limitations from environmental reception problems.

---

## Software-Defined Radio Architecture

Software-defined radio replaces portions of a conventional radio receiver's fixed-function signal processing with software.

A typical SDR receiver captures radio-frequency signals, converts them into digital samples and processes those samples using software.

The architecture can be represented as follows:

```text
                RF Signal
                    |
                    v
                 Antenna
                    |
                    v
             RF Front End
                    |
                    v
          Analog-to-Digital
              Conversion
                    |
                    v
             Digital Samples
                    |
                    v
             SDR Software
                    |
          +---------+---------+
          |         |         |
          v         v         v
      Spectrum   Demodulated  Recorded
      Display      Audio       IQ Data
```

This architecture provides flexibility because multiple signal processing operations can be performed on the same captured data.

Depending on the receiver and software, an SDR can support spectrum visualization, demodulation, signal recording and protocol decoding.

The project evaluated SDR as a general-purpose platform for RF experimentation rather than as a replacement for every dedicated radio.

---

## IQ Sampling and Digital Signal Processing

A fundamental SDR concept is the use of in-phase and quadrature samples.

IQ samples represent the amplitude and phase characteristics of a received signal.

They allow software to process a portion of the radio spectrum around a selected center frequency.

The sample rate determines the amount of spectrum that can be represented, subject to the receiver's hardware limitations and signal processing design.

Higher sample rates increase the amount of data generated.

For example, a receiver capturing complex 8-bit IQ samples at 2.4 million samples per second produces approximately 4.8 megabytes of raw data per second.

This corresponds to roughly 17.3 gigabytes per hour before accounting for recording overhead or compression.

The evaluation highlighted the importance of considering storage throughput and capacity when designing an SDR recording system.

---

## SDR Hardware Evaluation

Selecting an SDR receiver requires balancing frequency coverage, dynamic range, sample rate, resolution and cost.

Several hardware categories were considered suitable for different experimentation goals.

### Entry-Level USB Receivers

RTL-SDR-compatible receivers provide an accessible introduction to software-defined radio.

They are useful for monitoring supported frequencies, displaying spectrum activity and decoding many receive-only signal formats.

Their limitations include relatively modest analog-to-digital converter resolution and front-end performance compared with higher-end receivers.

### Wideband SDR Receivers

More advanced receivers provide broader frequency coverage, improved dynamic range or higher sample rates.

These capabilities can be valuable when examining crowded spectrum environments or signals with demanding bandwidth requirements.

### Transmit-Capable SDR Platforms

Some SDR devices support both transmission and reception.

These platforms introduce additional complexity involving RF filtering, output power, signal quality and regulatory compliance.

Transmit-capable hardware was considered a separate category requiring additional safeguards and testing.

The evaluation emphasized choosing hardware according to specific technical requirements rather than maximum advertised frequency coverage alone.

---

## Linux SDR Software

Linux was identified as a suitable operating environment for SDR experimentation.

The existing computing infrastructure already includes Linux-based desktop, server and portable systems.

Potential SDR applications include:

| Software | Primary Function |
|---|---|
| GNU Radio | Digital signal processing and SDR flowgraphs |
| GQRX | SDR receiver and spectrum visualization |
| SDR++ | Cross-platform SDR receiver |
| SoapySDR | Hardware abstraction interface |
| rtl-sdr utilities | Device testing and sample capture |
| Dire Wolf | Software packet radio modem and decoding |
| multimon-ng | Decoding selected digital radio signaling formats |

These applications provide different levels of complexity.

A graphical SDR receiver is useful for basic spectrum observation and tuning.

GNU Radio provides a more flexible environment for constructing custom digital signal processing pipelines.

The project identified Linux as a practical foundation for future SDR development and experimentation.

---

## Spectrum Analysis

One of the most useful capabilities of SDR is visualizing radio-frequency activity.

A spectrum display shows signal power across a range of frequencies.

A waterfall display adds a time dimension, making it possible to observe intermittent transmissions and changing signal conditions.

These tools can assist with identifying occupied channels, evaluating interference and comparing signal characteristics.

However, an SDR spectrum display is not automatically equivalent to a calibrated laboratory spectrum analyzer.

Measurements depend on receiver gain, front-end linearity, calibration and other hardware characteristics.

The project emphasized distinguishing qualitative spectrum observation from calibrated RF power measurement.

---

## Antenna Evaluation

Antenna selection is one of the most important aspects of radio system performance.

An SDR receiver or transceiver can only process signals delivered to its RF input.

Antenna characteristics affect the strength and quality of those signals.

Important considerations include:

- Operating frequency.
- Antenna polarization.
- Impedance matching.
- Feed-line attenuation.
- Connector compatibility.
- Antenna placement.
- Ground plane requirements.
- Environmental exposure.

Antenna design must be appropriate for the intended frequency range.

Antenna systems optimized for one band may perform poorly on another.

The project highlighted the importance of evaluating the complete RF signal path rather than attributing reception problems exclusively to the receiver.

---

## Feed Lines and RF Loss

Coaxial cable introduces attenuation between an antenna and radio equipment.

The amount of loss depends on cable type, frequency and length.

At higher frequencies, feed-line losses can become particularly significant.

Poor connectors, damaged cables and impedance mismatches can introduce additional performance problems.

For a mobile installation, cable routing must also account for vibration, weather exposure and mechanical strain.

The evaluation emphasized selecting appropriate coaxial cable and minimizing unnecessary feed-line length.

For transmission systems, impedance matching and reflected power require additional consideration.

---

## Legacy Radio Hardware Evaluation

The project also included investigating the Regency RH606B.

Legacy radio equipment provides an opportunity to study receiver architecture and the evolution of radio technology.

Unlike modern SDR receivers, traditional radios generally implement most of their tuning, filtering and demodulation functions through dedicated electronic circuits.

Evaluating older equipment requires attention to power supply condition, frequency coverage, component aging and interface compatibility.

The investigation provided a useful contrast between fixed-function radio hardware and software-defined signal processing.

It also connected the SDR project with broader experience troubleshooting vintage electronic equipment.

---

## Mobile SDR Infrastructure

A significant area of interest was integrating SDR functionality into a vehicle-mounted computer.

The proposed platform involved a Logic Supply RT130-i7-P industrial computer installed in a first-generation Toyota Sequoia.

The computer was evaluated for several functions, including offline navigation, radio monitoring and signal capture.

A proposed architecture is illustrated below.

```text
              Vehicle Antenna
                     |
                     v
                SDR Receiver
                     |
                    USB
                     |
                     v
             +----------------+
             | Industrial PC  |
             | Linux OS       |
             +----------------+
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Spectrum    Audio     IQ Recording
       Display     Output      Storage
          |
          v
     Touchscreen
      Interface
```

The proposed installation would provide a dedicated computing environment for RF experimentation while traveling.

However, the evaluation did not establish that the complete SDR system had been installed and operational in the vehicle.

---

## Vehicle Installation Considerations

A mobile SDR system introduces several challenges that do not occur in a stationary installation.

Vehicle electrical systems can produce voltage fluctuations and electrical noise.

The computer and SDR receiver require stable power.

A suitable automotive power supply must tolerate the operating conditions of the vehicle.

Additional considerations include:

- Ignition-controlled startup and shutdown.
- Protection from voltage transients.
- Antenna mounting.
- RF interference from vehicle electronics.
- USB device stability.
- Heat management.
- Vibration resistance.
- Safe touchscreen placement.
- Storage durability.

The project emphasized the importance of treating a vehicle installation as an integrated electrical and computing system.

---

## RF Interference and Noise

Radio reception can be degraded by electrical interference from nearby equipment.

Potential sources include switching power supplies, computers, USB interfaces, displays and vehicle electronics.

A noisy power supply may introduce unwanted signals into the receiver.

USB cables and poorly shielded equipment can also radiate interference.

The proposed troubleshooting methodology involves comparing reception under different operating conditions.

For example, a receiver could be tested with the vehicle engine off and then with the engine running.

Changes in observed interference would help identify whether the noise originates from vehicle systems.

The evaluation reinforced the importance of controlled testing when investigating RF noise.

---

## RF Signal Recording

Software-defined radio enables the recording of raw IQ samples for later analysis.

This capability is valuable because recorded signals can be replayed through compatible software without requiring the original transmission to remain active.

Potential applications include:

- Comparing receiver settings.
- Testing demodulation algorithms.
- Investigating intermittent signals.
- Developing digital signal processing workflows.
- Preserving examples of authorized transmissions for analysis.

IQ recordings can consume substantial storage capacity.

A practical recording workflow would need to define sample rate, sample format, recording duration and retention policy.

The existing Unraid storage infrastructure could provide a centralized destination for recordings generated by portable or stationary SDR systems.

This was considered a potential integration rather than a confirmed automated workflow.

---

## Homelab Integration

The broader homelab provides opportunities for centralized SDR processing and storage.

A network-connected SDR receiver or a dedicated Linux host could potentially expose authorized radio data to other internal services.

Possible integrations include:

- Centralized IQ recording storage.
- Remote spectrum monitoring.
- Scheduled signal capture.
- Signal processing containers.
- Environmental or telemetry decoding.
- Monitoring dashboards.
- Archival analysis.

A server-based architecture would also allow more computationally demanding processing to be performed on hardware separate from the receiver.

However, reliable network-based SDR operation depends on bandwidth, latency and the selected sample format.

The project highlighted the need to account for data throughput when moving raw radio samples across a network.

---

## Bench Instrumentation

Existing electronics test equipment provides additional opportunities for radio-related experimentation.

The project environment includes oscilloscopes such as the Tektronix 2235 and Rigol DS1054.

Oscilloscopes are useful for examining low-frequency waveforms, control signals, audio circuits and selected intermediate-frequency stages.

However, a general-purpose oscilloscope is not a substitute for a calibrated RF spectrum analyzer or vector network analyzer.

RF measurements require instruments with suitable bandwidth, input characteristics and measurement capabilities.

The project reinforced the importance of selecting appropriate test equipment for the signal being investigated.

---

## Troubleshooting Methodology

RF troubleshooting benefits from a layered approach similar to network troubleshooting.

### Stage 1: Verify Hardware Detection

Confirm that the Linux host recognizes the SDR device.

For USB receivers, inspect device enumeration and driver availability.

### Stage 2: Validate Software Access

Confirm that the selected SDR application can communicate with the receiver.

### Stage 3: Verify Receiver Configuration

Check frequency, sample rate, gain and demodulation settings.

### Stage 4: Inspect Antenna Connections

Verify that the antenna, connectors and feed line are appropriate for the operating frequency.

### Stage 5: Establish a Known Signal

Use an authorized, known transmission or suitable laboratory signal source to validate reception.

### Stage 6: Investigate Interference

Compare reception under controlled conditions to identify environmental noise sources.

### Stage 7: Validate Signal Processing

Confirm that demodulation or decoding settings match the received signal.

This methodology helps distinguish hardware problems from software configuration and environmental limitations.

---

## Technical Challenges Identified

### Hardware Compatibility

SDR software support varies between receiver models and operating systems.

### Signal Quality

Low-cost receivers may experience limitations involving dynamic range, overload and frequency accuracy.

### Antenna Performance

Poor antenna selection or placement can significantly reduce reception quality.

### RF Interference

Computers, power supplies and vehicle electronics can introduce unwanted signals.

### Data Throughput

High sample rates can generate large amounts of data requiring substantial storage and processing resources.

### Mobile Power Management

Vehicle installations require protection from electrical transients and unstable power conditions.

### Regulatory Compliance

Transmit-capable experimentation requires appropriate licensing, equipment configuration and operating procedures.

These challenges informed the proposed expansion of the RF experimentation environment.

---

## Security and Operational Considerations

Software-defined radio systems introduce several operational considerations.

Network-accessible receivers should be restricted to authorized users.

Remote control interfaces should not be exposed unnecessarily to public networks.

Recorded signal data should be managed according to applicable legal and privacy requirements.

Transmit-capable SDR hardware requires additional safeguards to prevent unintended transmissions.

The project emphasized separating receive-only experimentation from transmission-capable systems.

Any future automated transmission functionality would require explicit authorization, frequency restrictions and appropriate operational controls.

---

## Reliability and Recovery

A reliable SDR environment depends on stable hardware, software configuration and power delivery.

For portable systems, USB device reconnection and driver behavior can affect application stability.

For mobile systems, startup and shutdown procedures must account for vehicle power conditions.

Configuration files should document receiver parameters and device-specific settings.

Important recordings should be backed up when they are needed for later analysis.

The project also considered the value of maintaining known-working receiver configurations for troubleshooting.

This would allow software or hardware changes to be evaluated against a consistent baseline.

---

## Evaluation Outcome

The project established a technical foundation for experimenting with conventional radio equipment and software-defined receivers.

Hands-on amateur radio investigation provided experience with frequency configuration, operating procedures and the fundamentals of radio communication.

Legacy radio equipment evaluation provided additional context for understanding conventional receiver architecture.

The SDR investigation identified Linux-based signal processing and mobile computing integration as promising areas for future development.

However, the available project information does not establish that a complete SDR capture and analysis platform was deployed.

The SDR components should therefore be understood as an architectural evaluation and planned expansion of the existing radio experimentation environment.

---

## Lessons Learned

Software-defined radio provides significant flexibility by moving signal processing functions into software.

Receiver performance depends on both the SDR hardware and the quality of the antenna system.

IQ sampling introduces storage and processing requirements that must be considered when designing recording infrastructure.

Conventional radio equipment provides a useful foundation for understanding the behavior of software-defined receivers.

RF troubleshooting requires separating receiver configuration, antenna performance and environmental interference.

Vehicle-mounted SDR systems introduce additional electrical, mechanical and operational requirements.

Radio experimentation must account for licensing, authorized transmissions and the limitations of the selected equipment.

The project strengthened understanding of radio communications, Linux-based signal processing and hardware integration.

---

## Future Improvements

Future development may include:

- Selecting and testing an SDR receiver compatible with Linux.
- Establishing a repeatable SDR software configuration.
- Experimenting with spectrum visualization and waterfall analysis.
- Recording and replaying IQ samples.
- Evaluating antenna performance across different frequency bands.
- Testing reception in stationary and mobile environments.
- Investigating digital amateur radio modes.
- Integrating SDR recording storage with Unraid.
- Developing remote receiver access through the private network.
- Evaluating RF filtering and interference mitigation.
- Investigating GPS-assisted frequency and timing references.
- Implementing a vehicle-mounted SDR workstation.
- Documenting controlled RF measurement procedures.

These improvements would support progression from exploratory radio experimentation to a more structured software-defined radio laboratory.

---

## Summary

The SDR and RF Experimentation project investigated conventional radio communications, software-defined receivers and their potential integration with Linux-based computing infrastructure.

The work included amateur radio experimentation using a Baofeng UV-5R, evaluation of legacy Regency radio equipment and exploration of SDR applications for stationary and vehicle-mounted systems.

The project examined RF fundamentals, antenna systems, modulation, IQ sampling, spectrum analysis, signal recording and hardware troubleshooting.

It also identified opportunities to integrate radio signal processing with existing Unraid storage and Linux computing resources.

Although several SDR capabilities remain in the evaluation and planning stages, the project established a foundation for future RF experimentation.

The work demonstrates practical knowledge of radio communications, Linux hardware integration, digital signal processing concepts, electronics diagnostics, systems architecture and technical troubleshooting.
