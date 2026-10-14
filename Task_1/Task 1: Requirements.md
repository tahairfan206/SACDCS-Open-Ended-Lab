# Task 1. Requirement Extraction

## 1.1 Functional Requirements (10)

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| FR-01 | Only authenticated operators shall be permitted to issue robot commands. A command from an operator with no valid session, or a session idle for more than 15 min, shall be rejected with error AUTH_FAIL and the attempt logged. | FR | High | T (valid, invalid, expired session) |
| FR-02 | Command validation: before executing any command the robot controller shall check it against (a) the current robot state, (b) its parameters (destination ID must be one of the 20 registered clinics) and (c) the operator role. An invalid command shall be rejected with a reason code and the robot state left unchanged. | FR | High | T (equivalence classes, boundary values) |
| FR-03 | On a valid DeliveryRequest received in IDLE, the robot shall enter NAVIGATING and send its first status report within 2 s of local receipt. The same request received in any other state shall be rejected. | FR | High | T |
| FR-04 | Obstacle handling: when an obstacle is detected within 3 m ahead while NAVIGATING, the robot shall stop forward motion within 1 s and enter AVOIDING_OBSTACLE. It shall resume NAVIGATING only after the path has been clear for 2 s. It shall never enter DELIVERING from AVOIDING_OBSTACLE. | FR | High | T, A (model check) |
| FR-05 | Delivery and return: on DestinationReached (within 2 m of the clinic point, speed 0) the robot shall enter DELIVERING, release the batch only after the clinic handover is confirmed, then enter RETURNING and become IDLE on arrival at the warehouse. | FR | High | T, D |
| FR-06 | Safe Mode (baseline): the robot shall enter Safe Mode when a critical battery or thermal condition is detected. In Safe Mode it stops motion, gives the chamber priority on power and alerts the operator. | FR | High | T, A |
| FR-07 | Multi-robot communication (baseline): the system shall support communication with multiple robots simultaneously. | FR | High | T (load test) |
| FR-08 | Chamber correction with verification: when temperature or humidity leaves the batch profile range, the chamber shall start correction and declare recovery only after sensor readings stay in range for 60 consecutive seconds. If recovery does not occur within the 10-minute recovery period, the chamber shall enter protection mode, reduce light exposure and alert the operator within 5 s. | FR | High | T (fault injection) |
| FR-09 | Chamber activation: the chamber shall run a self-check at power-on and enter monitoring only if all essential sensors pass. It shall start active conservation only when the door is closed and the batch profile is loaded. | FR | Medium | T |
| FR-10 | Telemetry and event logging: each robot shall send telemetry every 5 s. Every command, state transition and alarm shall be logged with timestamp, robot ID, operator ID and outcome in non-volatile storage. If the link is down, the log shall be buffered for at least 24 h and uploaded in order on restoration. | FR | Medium | T, I |

## 1.2 Non-Functional Requirements (5)

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| NFR-01 (Performance) | At link delay up to 5 s, Mission Control shall receive the accept/reject acknowledgement of a command within 12 s for at least 95% of commands. Local safety reactions (Safe Mode, obstacle stop) shall not depend on the link. | NFR | High | T (network emulator) |
| NFR-02 (Security) | Operator accounts shall be locked for 15 min after 5 consecutive failed logins. Command traffic shall be encrypted and integrity-protected, and passwords stored only as salted hashes. | NFR | High | T, I |
| NFR-03 (Scalability) | With 20 robots connected, telemetry shall still be processed within 2 s of arrival, with CPU below 70%. The architecture shall be demonstrated at 50 robots in a load test (growth beyond the 20-clinic pilot). | NFR | Medium | T (load) |
| NFR-04 (Reliability) | Mission Control availability shall be at least 99.5% per month. No logged event shall be lost across a power loss or link loss (0 lost events in 100 injected faults). | NFR | High | A, T |
| NFR-05 (Usability) | After 30 min of training, a new operator shall issue a delivery command and identify an active alarm in under 60 s with fewer than 2 errors per 10 trials. Alarms shall be shown by colour and text within 2 s. | NFR | Medium | D (user study) |

## 1.3 High-Risk Requirements (4)

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| SR-01 (Railway) | The robot shall enter a level crossing only if barrier = open, no train is approaching and the ARLCCS link is healthy. On sensor failure, barrier failure, communication loss or inconsistent readings it shall stop at least 5 m before the crossing and alert the operator. | Safety | Critical | T (simulation), A (formal) |
| SR-02 (Link-dependent) | If no heartbeat is received for 15 s the robot shall enter LINK_LOST, reject all remote commands, halt at a safe position (never on the crossing), and enter Safe Mode if the link is not restored within 300 s. | Safety | Critical | T (delay and loss injection) |
| SR-03 (Chamber door) | If the chamber door opens during conservation, normal operation shall be suspended within 1 s. It shall resume only after the door is closed and temperature, humidity and sensors are verified in range. The robot shall not move while the door is open. | Safety | High | T |
| SR-04 (Real-time power) | On power loss the chamber shall switch to emergency power within 1 s if available. Otherwise it shall record the incident and shut down safely within 5 s, with the operator alerted. | Safety | Critical | T (fault injection) |
