# Test Cases (scenario flow: command received, authenticated, executed, status reported, logged)

| TC | Scenario / steps | Expected result | Req. |
|---|---|---|---|
| TC-01 | Command sent with no valid session or an expired session | Rejected AUTH_FAIL, not executed, rejection reported, logged. Also 5 failed logins lock the account. | FR-01, NFR-02 |
| TC-02 | Authenticated 'Viewer' role sends DeliveryRequest | Authentication passes, authorisation fails, command rejected, state unchanged, logged | FR-01, FR-02 |
| TC-03 | Authorised operator sends DeliveryRequest to an IDLE robot | Executed: robot NAVIGATING, first status within 2 s, ack within 12 s, log entry with operator ID | FR-02, FR-03, NFR-01 |
| TC-04 | Obstacle appears while NAVIGATING, then clears | AVOIDING_OBSTACLE within 1 s, then NAVIGATING after 2 s clear, both transitions reported and logged | FR-04 |
| TC-05 | Robot reaches destination, handover confirmed | DELIVERING, then RETURNING, then IDLE at warehouse. Each status reported and logged. | FR-05 |
| TC-06 | Inject temperature above critical (chamber, then battery) and battery below emergency level | SAFE_MODE within 3 s, operator alert, event logged. Run at threshold -1, = and +1. | FR-06, C-14 |
| TC-07 | Try IDLE to DELIVERING and AVOIDING_OBSTACLE to DELIVERING | Both rejected, state unchanged, violation logged | FR-02, FR-04, C-15 |
| TC-08 | CriticalBattery during AVOIDING_OBSTACLE and during DELIVERING | Avoidance finishes, then RETURNING. Handover finishes, then RETURNING. No undefined behaviour. | FR-05, FR-06 |
| TC-09 | Cut link for 15 s, restore, then cut for 300 s | LINK_LOST, remote commands rejected, restore gives RETURNING or IDLE, timeout gives SAFE_MODE | SR-02 |
| TC-10 | Connect 20 (and 21) robot simulators with telemetry every 5 s | All 20 connected, processing under 2 s, no dropped commands | FR-07, NFR-03 |
| TC-11 | Push chamber temperature or humidity out of range, then (a) correction works, (b) it fails | (a) recovery only after 60 s in range, (b) protection mode, light reduced, alert within 5 s | FR-08 |
| TC-12 | Open the door during conservation, close it, with and without sensor fault | Suspended within 1 s, no resume until verified, robot immobile while open | SR-03, C-09 |
| TC-13 | Train present, then comms loss, sensor failure, inconsistent readings at the crossing | Robot stops 5 m before crossing, alarm, operator alert | SR-01, C-01 to C-08 |
| TC-14 | Review the log after TC-01 to TC-13 | Every command and transition has timestamp, IDs and outcome. Nothing is lost. | FR-10, NFR-04 |
| TC-15 | Power on the chamber with one essential sensor failed, then with door open or profile missing | No monitoring, no conservation until conditions are met | FR-09 |
| TC-16 | Cut chamber power with and without emergency supply | Emergency power within 1 s, otherwise incident recorded and safe shutdown within 5 s | SR-04 |
| TC-17 | Usability study with 10 trained operators | Under 60 s per task, fewer than 2 errors per 10 trials | NFR-05 |
