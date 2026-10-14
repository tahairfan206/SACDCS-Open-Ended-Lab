# Task 2. Change Requests and Traceability

## 2.1 Version-controlled change log (v1.0 to v1.1)

| CR | Req. ID | Status | Old text (v1.0) | New text (v1.1) | Reason |
|---|---|---|---|---|---|
| CR-01 | FR-06 | **Modified** | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | The robot shall enter Safe Mode within 3 s when chamber or battery temperature exceeds the critical threshold, or battery capacity falls below the emergency level. | Mission Control needs a bounded reaction time and named triggers for the thermal and battery limits. |
| CR-02 | FR-07 | **Modified** | The system shall support communication with multiple robots simultaneously. | The system shall support at least 20 simultaneously connected robots. | The 'multiple' target was undefined. The pilot serves 20 clinics. |
| CR-03 | FR-01 | **Modified** | Only authenticated operators shall be permitted to issue robot commands. | The system shall require authenticated and role-authorised operators before accepting any robot command. A command from an unauthenticated operator, or from one whose role lacks permission for that command type, shall be rejected, not executed and logged. | Authentication alone does not prevent a valid but unprivileged user (for example a viewer) from commanding a robot. |
| (impact) | FR-02, NFR-03, SR-04 | Reviewed, no text change | n/a | n/a | FR-02 already checks the role (consistent with CR-03). NFR-03 already covers 20 robots, so FR-07 (connectivity) and NFR-03 (performance at that load) are complementary. SR-04 is unaffected. |

## 2.2 Why each new wording is more measurable or testable

- **CR-01.** The old text said 'critical' without a time limit, so any delay would pass. The new text has a pass/fail bound (3 s) and three observable triggers: chamber temperature above threshold, battery temperature above threshold and battery capacity below the emergency level. A tester injects each trigger, timestamps it, and checks that state = SAFE_MODE within 3 s. Boundary tests sit at the threshold, just below it and just above it.
- **CR-02.** 'Multiple' could mean 2. 'At least 20' gives a fixed number, so a test connects 20 robot simulators (and 21 as a boundary) and verifies that all report telemetry and accept commands at once.
- **CR-03.** The old rule had one condition (authenticated). The new one has two independent conditions, giving a 2 x 2 decision table (auth yes/no, role yes/no). Only one combination may be accepted, and the other three are explicit negative tests, each with a defined outcome (rejected, not executed, logged).

## 2.4 Test cases to re-run per change request

| CR | Must re-run | Reason |
|---|---|---|
| CR-01 (Safe Mode within 3 s) | TC-06, TC-08, TC-09, TC-12, TC-14 | These cover every path into Safe Mode, the new time limit and the logging of the event |
| CR-02 (at least 20 robots) | TC-10, TC-03, TC-14 | These cover the connection count, command and ack timing under load, and the log volume |
| CR-03 (authenticated and role-authorised) | TC-01, TC-02, TC-03, TC-07, TC-14 | These cover the new role check, plus every command-issuing test, since a role is now a precondition |

(The test cases themselves are in [tests/test-cases.md](../tests/test-cases.md).)

## 2.5 Multi-layer Requirement Traceability Matrix

IDs refer to the models in Tasks 3 to 5.

| Requirement | State / Transition | Operation | Constraint | Test case | Risk |
|---|---|---|---|---|---|
| FR-01 (CR-03) | Guard on every command, including T01 | OP-20 | C-13 | TC-01, TC-02, TC-03, TC-14 | High |
| FR-02 | T08, T09, T29, T30 (rejections) | OP-20, OP-21 | C-13, C-15 | TC-02, TC-07 | High |
| FR-03 | T01 | OP-21, OP-22 | C-15 | TC-03 | Medium |
| FR-04 | T02, T03 | OP-21 | C-15 | TC-04, TC-07 | High |
| FR-05 | T04, T05, T06, T10, T11, T12 | OP-18, OP-21 | C-10, C-11, C-15 | TC-05, TC-08 | High |
| FR-06 (CR-01) | T07, T13 to T19, T25, T26 | OP-09, OP-11, OP-21 | C-14 | TC-06, TC-08, TC-09 | Critical |
| FR-07 (CR-02) | All states (connectivity) | OP-22 | none (see NFR-03) | TC-10 | Medium |
| FR-08 | CH-CONSERVING, CH-PROTECTION | OP-05 to OP-11 | C-11 | TC-11 | High |
| FR-09 | CH-SELFCHECK, CH-MONITORING, CH-CONSERVING | OP-01 to OP-04 | C-09 | TC-15 | Medium |
| FR-10 | All transitions | OP-19, OP-22 | C-12 | TC-14 | Medium |
| SR-01 | S2 (crossing segment) | OP-21 | C-01 to C-08 | TC-13 | Critical |
| SR-02 | T20 to T28 (S7) | OP-22 | C-12 | TC-09 | Critical |
| SR-03 | CH-SUSPENDED | OP-14, OP-15 | C-09, C-10 | TC-12 | High |
| SR-04 | CH-EMERGENCY_POWER, CH-SHUTDOWN | OP-16, OP-17 | C-16 | TC-16 | Critical |
| NFR-01 | T01 (ack path) | OP-20, OP-22 | none | TC-03 | Medium |
| NFR-02 | n/a | OP-20 | C-13 | TC-01 | High |
| NFR-03 | n/a | OP-22 | none | TC-10 | Medium |
| NFR-04 | All | OP-19 | C-12 | TC-14, TC-16 | High |
| NFR-05 | n/a | OP-11 | none | TC-17 | Low |
