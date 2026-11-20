
# Self-Hosted SMS Gateway Evaluation

## Overview

The Self-Hosted SMS Gateway Evaluation project focused on investigating methods of integrating Short Message Service (SMS) functionality into a privately managed homelab environment.

The objective was to evaluate whether SMS messaging could be hosted and administered locally rather than relying entirely on commercial messaging APIs or cloud-based notification platforms.

The project examined cellular modem hardware, GSM/LTE gateways, SMS gateway software, SIP-based messaging integration and potential interoperability with existing VoIP and automation infrastructure.

Particular emphasis was placed on integrating SMS functionality with Home Assistant, Linux servers and the existing private branch exchange (PBX) environment.

The evaluation provided practical experience with telecommunications architecture, cellular networking, messaging protocols and infrastructure integration.

---

## Objectives

The primary objective was to determine the feasibility of operating a self-hosted SMS gateway capable of sending and receiving text messages through locally managed infrastructure.

The evaluation focused on the following goals:

- Investigating cellular hardware capable of SMS transmission and reception.
- Comparing software-based and dedicated hardware SMS gateways.
- Evaluating integration with Linux and Docker environments.
- Exploring interoperability with SIP and PBX infrastructure.
- Investigating Home Assistant notification and automation capabilities.
- Assessing ongoing operating costs and carrier requirements.
- Identifying security and reliability considerations.
- Determining whether a self-hosted gateway could reduce dependence on commercial SMS APIs.

The project also aimed to distinguish between hosting the messaging application locally and operating the underlying telecommunications network.

Even with a self-hosted gateway, SMS delivery normally requires connectivity to a mobile carrier or another external messaging provider.

---

## Infrastructure Environment

The evaluation was conducted in the context of an existing self-hosted infrastructure environment.

Relevant systems included:

| Component | Role |
|---|---|
| Unraid | Primary Linux-based container hosting platform |
| Docker | Deployment environment for gateway software |
| Home Assistant | Automation and notification platform |
| OPNsense | Firewall, routing and network access control |
| FusionPBX | Existing SIP and telephony infrastructure |
| FreePBX | Alternative PBX platform evaluated within the homelab |
| VoIP.ms | Existing SIP trunking provider |
| GSM/LTE modem | Potential cellular network interface |
| SMS gateway software | Message processing, routing and application integration |

The existing environment provided a foundation for investigating how messaging services could be integrated into established infrastructure.

The project was an architectural and technology evaluation rather than a confirmed production SMS gateway deployment.

---

## SMS Gateway Architecture

A self-hosted SMS gateway acts as an intermediary between local applications and an external messaging transport.

For a cellular modem-based implementation, the gateway would communicate directly with a modem connected to a mobile carrier.

Applications would submit outbound messages to the gateway, which would instruct the modem to transmit them over the cellular network.

Incoming messages would be retrieved from the modem and delivered to authorized applications or users.

A proposed architecture is illustrated below.

```text
             Local Applications
                     |
          +----------+----------+
          |                     |
          v                     v
    Home Assistant         PBX / Services
          |                     |
          +----------+----------+
                     |
                     v
              SMS Gateway
              Linux Server
                     |
                     v
               GSM/LTE Modem
                     |
                     v
              Mobile Carrier
                     |
                     v
              SMS Recipient
```

This architecture provides local control over the application layer while using the carrier network for message delivery.

The design also allows messaging functionality to be exposed to multiple internal services through a common gateway.

---

## SMS Transport Technologies

Several transport methods were considered during the evaluation.

### Cellular Modem-Based SMS

A GSM/LTE modem equipped with a compatible SIM card can provide direct access to carrier SMS services.

The gateway communicates with the modem using supported control interfaces, which may include AT commands.

This approach provides substantial local control over message submission, reception and routing.

However, compatibility depends on the modem, carrier, network technology and subscription.

### Dedicated Cellular Gateways

Dedicated cellular gateways combine modem hardware with an embedded management platform.

Some devices expose SMS functionality through HTTP APIs, web interfaces or other integration methods.

These appliances may simplify deployment but can introduce proprietary management interfaces and additional hardware costs.

### Commercial SMS APIs

Cloud-based messaging providers offer programmable SMS functionality through hosted APIs.

These services generally simplify carrier integration and scaling but introduce recurring costs and external service dependencies.

They were considered as a comparison point rather than the preferred self-hosted architecture.

### SIP-Based Messaging

SIP supports instant messaging through methods such as SIP MESSAGE.

However, SIP messaging and carrier SMS are separate technologies.

A SIP provider must explicitly support SMS functionality and provide a compatible integration mechanism before messages can be exchanged with mobile telephone numbers.

This distinction was particularly important when evaluating integration with the existing PBX infrastructure.

---

## GSM/LTE Modem Evaluation

Cellular modem selection was identified as one of the most important technical decisions.

A suitable modem must support the cellular bands and technologies required by the intended mobile carrier.

It must also expose SMS functionality through a software-accessible interface.

Hardware evaluation criteria included:

- Carrier compatibility.
- LTE band support.
- SIM card compatibility.
- Linux driver availability.
- USB or Ethernet connectivity.
- SMS command support.
- Device stability.
- Power consumption.
- Antenna requirements.
- Long-term operating cost.

Legacy GSM-only hardware presents significant compatibility concerns because many mobile operators have retired older cellular technologies.

For a deployment in the United States, LTE compatibility and carrier provisioning would be particularly important.

A modem supporting LTE data connectivity does not necessarily guarantee that SMS functionality will operate with every carrier or subscription.

Carrier-specific validation would therefore be required before purchasing hardware for production use.

---

## Linux SMS Gateway Software

The evaluation considered software capable of interfacing with cellular modems and exposing messaging functionality to other applications.

Potential software platforms included Gammu, Gammu SMSD and Kannel.

### Gammu

Gammu provides tools for communicating with supported mobile phones and cellular modems.

It can perform operations such as reading messages, sending SMS and interacting with modem functionality.

The software provides a relatively direct method of testing modem compatibility before introducing additional application infrastructure.

### Gammu SMSD

Gammu SMSD extends Gammu by providing a background service for processing incoming and outgoing messages.

Messages can be managed through supported storage backends, allowing applications to interact with a message queue.

This approach was considered suitable for a small-scale, self-hosted gateway.

### Kannel

Kannel is an open-source gateway platform supporting SMS-related messaging interfaces.

Its architecture is more oriented toward messaging gateway operations and integration with external systems.

Kannel was considered as a potential option for environments requiring more advanced message routing or gateway functionality.

The evaluation highlighted the importance of matching software complexity to the intended deployment scale.

For a small homelab notification system, a lightweight modem interface may be preferable to a more complex messaging gateway.

---

## Docker Deployment Considerations

Containerization was considered as a deployment method for SMS gateway software.

A Docker-based gateway could operate alongside other self-hosted applications on the Unraid server.

Containerization would simplify application dependency management and allow gateway software to be updated independently of the host operating system.

However, modem-based deployments introduce hardware access requirements.

A USB cellular modem may expose serial devices such as `/dev/ttyUSB0` or control interfaces such as `/dev/cdc-wdm0`, depending on its implementation.

The gateway container would require access to the appropriate device nodes.

Device permissions, modem initialization and host-level modem management services would need to be considered.

USB device numbering may also change following reboots or reconnection.

Stable device identification through persistent paths or appropriate udev rules would improve reliability.

The project demonstrated that containerized hardware integrations require coordination between the host operating system and the application runtime.

---

## Home Assistant Integration

Home Assistant was identified as a primary consumer of SMS gateway functionality.

The proposed integration would allow automations to send text messages when predefined conditions are met.

Potential applications included:

- Critical infrastructure failure notifications.
- UPS power events.
- Security system alerts.
- Water leak detection.
- Network service outages.
- Equipment temperature warnings.
- Backup failure notifications.

A locally managed gateway could provide an alternative notification path when conventional internet-based notification services are unavailable.

However, this benefit depends on the gateway remaining powered and having a functioning cellular connection.

For example, an SMS gateway connected to a UPS-backed server and cellular modem could potentially deliver alerts during certain internet outages.

This would not provide complete independence from external infrastructure because the cellular carrier network would still be required.

The integration architecture would need to include appropriate authentication and message delivery status reporting.

---

## PBX & VoIP Integration

The existing homelab includes experience with FusionPBX, FreePBX and VoIP.ms.

This infrastructure provided a useful foundation for evaluating whether SMS functionality could be integrated into the broader telephony environment.

Potential integration methods included:

- Provider-supported SMS APIs.
- PBX application hooks.
- HTTP-based gateway interfaces.
- SIP MESSAGE support where available.
- Custom message routing services.

An important consideration was whether an existing VoIP telephone number could send and receive SMS.

Voice service availability does not automatically imply SMS support.

SMS capabilities depend on the telephone number, provider, registration requirements and available messaging interfaces.

The evaluation also considered whether a cellular gateway could operate independently of the SIP trunk while exposing messaging functionality to PBX users.

This architecture would provide a separate cellular messaging path but would generally use the telephone number associated with the cellular SIM.

It would not automatically enable SMS functionality for existing VoIP numbers.

---

## Inbound Message Processing

Receiving SMS introduces additional architectural requirements beyond outbound notifications.

An inbound messaging service must identify new messages, retrieve their contents and determine the appropriate destination.

Potential destinations include a database, application webhook, administrative dashboard or automation service.

A proposed inbound processing workflow would involve:

1. The cellular modem receives an SMS.
2. Gateway software retrieves the message.
3. The message is stored or queued.
4. A processing service validates the sender and message contents.
5. The message is delivered to an authorized internal application.
6. The gateway records the processing result.

Inbound message handling also requires safeguards against unauthorized commands.

Messages originating from the public telephone network should not automatically be trusted simply because they appear to come from a recognized telephone number.

Sender identification can be unreliable, and SMS is not an appropriate authentication mechanism for sensitive administrative operations.

The architecture therefore favors inbound SMS as a notification or communication channel rather than a privileged remote-control interface.

---

## Message Queuing & Delivery Reliability

Reliable SMS delivery requires more than submitting a message to a modem.

A gateway must distinguish between a message being accepted by the local application, submitted to the modem and successfully delivered to the recipient.

Carrier delivery receipts may be available depending on modem, network and software support.

The proposed design considered message queues to prevent temporary modem unavailability from immediately discarding outbound notifications.

Queue management requirements include:

- Message timestamps.
- Delivery attempts.
- Retry limits.
- Failure reporting.
- Duplicate prevention.
- Message retention policies.

Retry logic must be implemented carefully because resubmitting a message after an ambiguous failure can result in duplicate delivery.

The evaluation reinforced the importance of designing notification infrastructure around observable delivery states rather than assuming that successful API submission guarantees receipt.

---

## Security Considerations

Security was a major consideration throughout the evaluation.

An SMS gateway creates a connection between internal applications and an external telecommunications network.

The gateway must therefore be treated as a security-sensitive infrastructure service.

Important controls include restricting access to messaging APIs, protecting credentials and limiting which applications can submit outbound messages.

Additional safeguards should include rate limiting, message logging and monitoring for unexpected traffic.

Unauthorized use could result in financial charges, unwanted messages or abuse of the associated telephone number.

Sensitive information should not be transmitted through SMS unless the risks are understood and acceptable.

SMS is not end-to-end encrypted, and message contents may be accessible to telecommunications intermediaries.

The project emphasized that self-hosting the gateway application does not eliminate the security limitations of the underlying SMS protocol.

---

## Carrier Requirements & Operating Costs

Operating a self-hosted SMS gateway generally requires an active messaging service.

For a modem-based deployment, this would typically involve a compatible SIM card and a cellular subscription that permits the intended usage.

Costs may include:

- Cellular modem hardware.
- SIM card activation.
- Monthly service charges.
- Per-message charges.
- Antennas and accessories.
- Electrical power consumption.

Carrier terms may restrict automated messaging or the use of certain modem devices.

Commercial SMS API providers may also require sender registration or compliance with applicable application-to-person messaging requirements.

The evaluation considered these factors when comparing local cellular gateways against hosted messaging services.

A self-hosted gateway may reduce application hosting dependencies, but it does not necessarily eliminate recurring telecommunications expenses.

---

## Monitoring & Telemetry

Operational monitoring would be necessary for a production SMS gateway.

A gateway may appear operational while being unable to send messages because of carrier registration failures, modem errors or subscription problems.

Monitoring requirements include:

- Gateway process availability.
- Modem detection.
- Cellular registration status.
- Signal quality.
- Outbound queue depth.
- Message submission failures.
- Delivery status where available.
- Storage utilization.

Periodic test messages could provide additional end-to-end validation.

However, such testing would need to account for carrier charges and message frequency limits.

Monitoring integration with Home Assistant or another centralized platform could provide visibility into gateway health.

The evaluation highlighted the importance of distinguishing application availability from actual telecommunications service availability.

---

## Troubleshooting Methodology

Troubleshooting a cellular SMS gateway requires separating software, hardware and carrier-related problems.

The initial diagnostic stage involves confirming that the Linux host recognizes the modem.

The next stage involves validating device permissions and determining whether the gateway software can communicate with the hardware.

Modem status should then be inspected to confirm SIM recognition and cellular network registration.

Once the modem is functioning, outbound messaging can be tested independently of application integrations.

Inbound message reception should be validated separately.

Only after basic messaging functionality is confirmed should integrations with Home Assistant or PBX services be introduced.

This staged methodology helps distinguish modem compatibility problems from application configuration and networking issues.

---

## Technical Challenges Identified

### Carrier Compatibility

Not all cellular modems support the bands and network services required by modern carriers.

Device compatibility must be confirmed before deployment.

### SMS Support on Data Plans

A cellular data connection does not guarantee that SMS transmission and reception are enabled.

Subscription capabilities must be validated independently.

### USB Device Management

USB modem interfaces can change device numbering or become unavailable following disconnection.

Stable device mapping and recovery procedures are important for unattended operation.

### Message Delivery Confirmation

A successfully submitted message may still fail to reach its recipient.

Reliable operation requires appropriate status tracking and failure handling.

### VoIP Number Integration

Existing SIP telephone numbers cannot automatically be used through a separate cellular SMS gateway.

Provider support and number ownership determine available messaging options.

### Operational Cost

A locally hosted gateway still depends on carrier services and may incur ongoing charges.

### Security & Abuse Prevention

Messaging interfaces require access restrictions to prevent unauthorized transmission and unexpected costs.

These considerations influenced the feasibility assessment and potential deployment strategy.

---

## Reliability & Recovery

A production deployment would require recovery procedures for both the software gateway and the cellular modem.

Container restart policies could assist with recovering from application failures.

However, some hardware failures may require modem reinitialization or USB device recovery.

Persistent message queues would help preserve pending messages across application restarts.

Configuration backups would simplify rebuilding the gateway following host or storage failures.

UPS-backed infrastructure could improve availability during short power interruptions.

A reliable deployment would also require periodic verification that messages can actually be transmitted through the cellular network.

The evaluation demonstrated that telecommunications reliability depends on multiple independent systems, including local software, hardware and external carrier infrastructure.

---

## Evaluation Outcome

The evaluation identified a cellular modem connected to a Linux-based SMS gateway as a viable architectural approach for locally managed SMS functionality, subject to hardware and carrier compatibility.

Gammu SMSD emerged as a reasonable candidate for a lightweight gateway, while more specialized platforms could be considered if message routing requirements expanded.

The existing Unraid and Home Assistant infrastructure provides a suitable environment for hosting and integrating gateway software.

However, the evaluation did not establish that a particular modem, carrier subscription or software configuration had been successfully deployed as a production messaging service.

Further testing would be necessary to confirm hardware compatibility, delivery reliability and operational costs.

The project therefore remains an infrastructure evaluation and deployment planning exercise.

---

## Lessons Learned

Self-hosting SMS gateway software does not eliminate dependence on mobile carrier infrastructure.

Cellular hardware compatibility is a critical requirement and must be validated against the intended carrier.

SIP messaging and carrier SMS are distinct technologies that require explicit interoperability support.

Containerized deployments involving cellular modems introduce hardware access and device management challenges.

Reliable messaging requires queue management, delivery tracking and monitoring beyond basic application availability.

SMS should not be treated as a secure transport for privileged administrative commands.

A cellular messaging gateway can provide a useful alternative notification path, but its availability depends on local power, modem functionality and carrier connectivity.

The evaluation strengthened understanding of how telecommunications services can be integrated into self-hosted infrastructure.

---

## Future Improvements

Future work will focus on validating the proposed architecture through controlled hardware testing.

Potential next steps include:

- Selecting a carrier-compatible LTE modem.
- Confirming SMS functionality with a supported subscription.
- Testing Gammu and Gammu SMSD on Linux.
- Evaluating USB device passthrough to Docker.
- Implementing persistent outbound message queues.
- Creating a restricted messaging API for internal applications.
- Integrating outbound notifications with Home Assistant.
- Testing inbound message processing.
- Evaluating compatibility with existing PBX services.
- Implementing gateway health monitoring.
- Measuring message delivery reliability and operating costs.

A successful proof of concept would provide the basis for determining whether the gateway should become a permanent infrastructure service.

---

## Summary

The Self-Hosted SMS Gateway Evaluation project investigated the feasibility of integrating cellular text messaging into a privately managed homelab environment.

The evaluation covered cellular modem hardware, Linux SMS gateway software, Docker deployment, carrier requirements, PBX integration and Home Assistant automation.

The project demonstrated that local SMS gateway infrastructure is technically feasible while identifying important limitations involving carrier dependency, hardware compatibility, message delivery and security.

Although the architecture was not confirmed as a production deployment, the evaluation established a practical direction for future implementation.

The project strengthened skills in telecommunications infrastructure, Linux systems administration, containerized service design, network integration, security assessment and technical feasibility analysis.
