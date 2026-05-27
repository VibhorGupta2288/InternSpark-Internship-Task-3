# InternSpark-Internship-Task-3
 Network Packet Analysis and Traffic Inspection


## Table of Contents

1. Overview
2. Goal
3. Requirements
4. Traffic Capture and Analysis
5. Unusual and Suspicious Packet Behavior
6. Technical Findings
7. Risk Classification
8. Recommendations
9. Conclusion


## Overview

This project demonstrates a network traffic capture and analysis exercise performed to identify communication protocols, inspect packet behavior, and detect suspicious or abnormal network activity.

The investigation focused on analyzing TCP handshake behavior, HTTP and HTTPS communication, DNS activity, QUIC protocol traffic, packet retransmissions, connection failures, and TCP flags and packet structures. The analysis identified failed local service connections, retransmission activity, port reuse behavior, and successful encrypted external communications using TLS and QUIC protocols.


## Goal

Capture and analyze network packets to identify active protocols and detect suspicious or abnormal traffic behavior across local and external communications.


## Requirements

The assessment included the following objectives:

Capturing HTTP, DNS, and TCP handshake traffic. Analyzing TCP flags and packet structures. Identifying retransmissions and malformed packets. Detecting failed connection attempts. Reviewing encrypted TLS and QUIC communication behavior.


## Traffic Capture and Analysis


### TCP Handshake Traffic

**Filter Used**

```
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

**Observations**

Multiple SYN packets were observed originating from 127.0.0.1 and ::1. Destination ports included Port 1521 for Oracle Database, Port 8080 for a web server, and Port 3000 for a development server.

Some connections received no SYN-ACK response while others immediately returned RST, ACK responses, indicating that the target services were not available or not actively listening.


### HTTP and HTTPS Traffic

**Ports Observed**

Port 443 was observed carrying HTTPS traffic.

**Protocols Identified**

TLSv1.2, TLSv1.3, and QUIC (HTTP/3) were all identified during the capture session.

**TLS Handshake Observations**

Successful encrypted sessions were observed containing Client Hello, Server Hello, and Encrypted Application Data exchanges. Server Name Indication fields revealed the following domains being accessed: www.google.com, fonts.gstatic.com, and www.gstatic.com.


### DNS Traffic

DNS activity was observed during browsing sessions, occurring before TLS and QUIC connections were established. Domain resolution activity was primarily associated with Google services.


### Packet Structure and Flag Analysis

**TCP Flags Observed**

| Flag | Meaning | Observed Behavior |
|------|---------|------------------|
| SYN | Start connection | Frequently observed |
| ACK | Acknowledgment | Present in responses |
| RST | Reset connection | Frequent in local services |
| PSH | Push data | Observed in HTTPS traffic |
| FIN | Close connection | Seen in retransmissions |

**Key Packet Patterns**

A normal TCP handshake follows the sequence SYN, SYN-ACK, ACK and was observed during all successful external communications.

A failed TCP handshake follows the sequence SYN, RST, ACK and was consistently observed during failed local service connection attempts.

Retransmission activity was identified through Wireshark indicators labeled as TCP Retransmission and TCP Spurious Retransmission.

**QUIC Packet Structure**

QUIC frame types observed included CRYPTO, PING, and PADDING. All QUIC traffic was carried over UDP.

**TLS Packet Structure**

The observed TLS sequence consisted of Client Hello, Server Hello, Change Cipher Spec, and Encrypted Application Data.


## Unusual and Suspicious Packet Behavior


### Broken TCP Handshakes

Continuous SYN packets were observed with no successful handshake completion and immediate RST, ACK responses returned by the server. This behavior indicates that services were not running or not actively listening on Port 8080 and Port 1521.


### TCP Port Reuse

Wireshark reported a TCP Port numbers reused indicator during the capture. This behavior may indicate rapid retry attempts by the application, application level retry loops, or poor connection handling logic within the client software.


### TCP Retransmissions

Both standard retransmissions and spurious retransmissions were observed. Possible causes include packet loss, network congestion, Wi-Fi instability, and delayed packet delivery.

**Capture Warnings**

Wireshark reported the warnings TCP Previous segment not captured and Ignored Unknown Record during the session. These warnings indicate packet capture limitations, missing captured frames, and non-malicious recording issues. No confirmed malicious activity was associated with these warnings.


## Technical Findings


### Failed Local Service Connections

Continuous SYN packets followed by RST, ACK responses indicated closed ports, misconfigured services, or stopped applications. Affected services included Oracle Database on Port 1521 and a web server on Port 8080.


### Application Retry Behavior

Rapid retry attempts and repeated port reuse suggested aggressive reconnection logic, possible infinite retry loops, and poor application level error handling.


### Healthy External Communications

Successful encrypted communication was observed with Google services and Cloudflare infrastructure. Protocols used included TLS and QUIC, confirming proper and secure internet connectivity was functioning normally.


## Risk Classification

| Issue | Risk Level |
|-------|-----------|
| Local service connection failures | Medium |
| TCP retransmissions | Low |
| External encrypted traffic | No Risk |


## Recommendations


### Service Availability

Start and verify all required local services including Oracle Database on Port 1521 and the web server on Port 8080. Confirm that each service is running and actively listening on its expected port before conducting further tests.


### Network Configuration

Verify port bindings, service configurations, firewall rules, and that all expected services are in a listening state. Misconfigurations in any of these areas can result in connection failures similar to those observed.


### Application Improvements

Improve application level retry logic to prevent excessive and repeated connection attempts. Implement proper connection timeout handling and error responses to avoid infinite retry loops.


### Network Monitoring

Monitor retransmission frequency over time. Investigate network stability if retransmissions increase beyond normal thresholds. Review abnormal TCP behaviors periodically as part of routine network health checks.


## Conclusion

The packet analysis identified two primary behaviors during the capture session. The first was failed local TCP connections to ports 8080 and 1521, and the second was successful encrypted external communication using TLS and QUIC protocols.

The investigation showed that local services were either unavailable or misconfigured, resulting in repeated failed handshake attempts and TCP port reuse behavior. External internet communication functioned normally using secure and properly negotiated encrypted protocols.

The assessment demonstrates how packet analysis can be used to identify service failures, connection anomalies, retransmission activity, network instability, protocol behavior, and secure communication patterns within a network environment.


## Disclaimer

This analysis was conducted in a controlled lab environment for educational purposes only. All packet captures were performed on networks and systems owned and operated by the tester. Never capture or inspect network traffic on systems or networks without proper authorization.
