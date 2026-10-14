# Task 3. State Modelling and Transition Verification

## 3.1 States

| ID | State | Meaning | Entry condition | Exit condition |
|---|---|---|---|---|
| S1 | IDLE | Robot on, stationary, waiting for a request | Power-on, WarehouseReached, or authorised recovery at the warehouse | DeliveryRequest accepted, SafeModeTrigger or LinkLost |
| S2 | NAVIGATING | Driving to the clinic and checking for obstacles | Valid DeliveryRequest in S1, or ObstacleCleared in S3 | ObstacleDetected, DestinationReached, CriticalBattery, SafeModeTrigger, LinkLost |
| S3 | AVOIDING_OBSTACLE | Navigation suspended, robot manoeuvring around an obstacle | ObstacleDetected in S2 | ObstacleCleared, SafeModeTrigger, LinkLost |
| S4 | DELIVERING | Stationary at the clinic, handing over the batch | DestinationReached in S2 (only source) | PackageDelivered, SafeModeTrigger |
| S5 | RETURNING | Driving back to the warehouse (also for an abandoned delivery) | PackageDelivered, CriticalBattery in S2, ObstacleCleared with abort flag, or recovery away from the warehouse | WarehouseReached, SafeModeTrigger, LinkLost |
| S6 | SAFE_MODE (added, CR-01) | Motion stopped, chamber power prioritised, operator alerted | SafeModeTrigger from any state, or link-loss timeout | Authorised OperatorRecover with conditions cleared |
| S7 | LINK_LOST (added) | No heartbeat for 15 s, robot halted at a safe point, remote commands rejected | LinkLost from S1, S2, S3 or S5 (S4 defers) | LinkRestored, link timeout of 300 s, or SafeModeTrigger |

## 3.2 Events and the difference between a state and an event

A **state** is a condition the robot stays in for a period of time (for example NAVIGATING lasts until something happens). An **event** is an instantaneous occurrence or detected condition that may cause a change of state, and it takes no time (for example ObstacleDetected). A state answers 'what is the robot doing now?', while an event answers 'what just happened?'. The same event can have different effects depending on the state: DeliveryRequest starts navigation in IDLE but must be rejected everywhere else.

| ID | Event | Meaning |
|---|---|---|
| E1 | DeliveryRequest | Valid command from an authenticated and role-authorised operator |
| E2 | ObstacleDetected | Obstacle within 3 m on the path |
| E3 | ObstacleCleared | Path clear for 2 s |
| E4 | DestinationReached | Within 2 m of the clinic point and speed 0 |
| E5 | PackageDelivered | Handover confirmed by the clinic |
| E6 | CriticalBattery | Battery at or below the return threshold (for example 20%) |
| E7 | WarehouseReached | Robot docked at the warehouse |
| E8 | SafeModeTrigger | Temperature above critical, battery below emergency level (for example 10%), or critical fault (CR-01) |
| E9 | LinkLost | No heartbeat for 15 s |
| E10 | LinkRestored | Heartbeat received and session re-validated |
| E11 | OperatorRecover | Authorised recovery command with trigger conditions cleared |
| E12 | LinkTimeout | 300 s in LINK_LOST |

## 3.3 State Transition Table (Current State + Event gives Next State)

TC references are the evidence from the test cases. 'Added' marks transitions that were missing in the narrative and found during verification.

| ID | Current state | Event | Next state | Req. | Valid? | Verification evidence |
|---|---|---|---|---|---|---|
| T01 | IDLE | E1 | NAVIGATING | FR-03 | Valid | TC-03 |
| T02 | NAVIGATING | E2 | AVOIDING_OBSTACLE | FR-04 | Valid | TC-04 |
| T03 | AVOIDING_OBSTACLE | E3 | NAVIGATING | FR-04 | Valid | TC-04 |
| T04 | NAVIGATING | E4 | DELIVERING | FR-05 | Valid | TC-05 |
| T05 | DELIVERING | E5 | RETURNING | FR-05 | Valid | TC-05 |
| T06 | RETURNING | E7 | IDLE | FR-05 | Valid | TC-05 |
| T07 | NAVIGATING | E6 | RETURNING (delivery abandoned) | FR-06 | Valid | TC-08 |
| T08 | IDLE | start delivering directly | DELIVERING | FR-02, FR-03 | **Invalid** | TC-07, model check: no such event exists |
| T09 | AVOIDING_OBSTACLE | E4 | DELIVERING | FR-04 | **Invalid** | TC-07 |
| T10 | AVOIDING_OBSTACLE | E3 with abort flag | RETURNING | FR-05, FR-06 | Valid (added) | TC-08 |
| T11 | AVOIDING_OBSTACLE | E6 | AVOIDING_OBSTACLE (abort flag set) | FR-06 | Valid (added) | TC-08 |
| T12 | DELIVERING | E6 | DELIVERING (abort flag set, handover completes) | FR-05, FR-06 | Valid (added) | TC-08 |
| T13 | IDLE | E8 | SAFE_MODE | FR-06 | Valid | TC-06 |
| T14 | NAVIGATING | E8 | SAFE_MODE | FR-06 | Valid | TC-06 |
| T15 | AVOIDING_OBSTACLE | E8 | SAFE_MODE | FR-06 | Valid | TC-06 |
| T16 | DELIVERING | E8 | SAFE_MODE | FR-06 | Valid | TC-06 |
| T17 | RETURNING | E8 | SAFE_MODE | FR-06 | Valid | TC-06 |
| T18 | SAFE_MODE | E11 (at warehouse) | IDLE | FR-06 | Valid | TC-06 |
| T19 | SAFE_MODE | E11 (away from warehouse, battery above emergency level) | RETURNING | FR-06 | Valid | TC-06 |
| T20 | IDLE | E9 | LINK_LOST | SR-02 | Valid | TC-09 |
| T21 | NAVIGATING | E9 | LINK_LOST | SR-02 | Valid | TC-09 |
| T22 | AVOIDING_OBSTACLE | E9 | LINK_LOST | SR-02 | Valid | TC-09 |
| T23 | RETURNING | E9 | LINK_LOST | SR-02 | Valid | TC-09 |
| T24 | LINK_LOST | E10 | IDLE (if previously idle) or RETURNING | SR-02 | Valid | TC-09 |
| T25 | LINK_LOST | E12 | SAFE_MODE | SR-02 | Valid | TC-09 |
| T26 | LINK_LOST | E8 | SAFE_MODE | FR-06 | Valid | TC-06 |
| T27 | DELIVERING | E9 | DELIVERING (link flag set, handover is local) | SR-02 | Valid (added) | TC-09 |
| T28 | DELIVERING | E5 with link flag | LINK_LOST | SR-02 | Valid (added) | TC-09 |
| T29 | SAFE_MODE | E1 | (no change) rejected | FR-02 | Valid (rejection) | TC-07 |
| T30 | LINK_LOST | E1 | (no change) rejected | FR-02, SR-02 | Valid (rejection) | TC-09 |

## 3.4 Extension with Safe Mode (CR-01) and Link Lost

- **Safe Mode** may be entered from **all** states (IDLE, NAVIGATING, AVOIDING_OBSTACLE, DELIVERING, RETURNING, LINK_LOST). A thermal or battery emergency is independent of what the robot is doing, so the trigger must be handled everywhere. It has the highest priority and must complete within 3 s. Exit is only by an authorised OperatorRecover, and never directly to NAVIGATING or DELIVERING.
- **Link Lost** may be entered from IDLE, NAVIGATING, AVOIDING_OBSTACLE and RETURNING. It is **not** entered directly from DELIVERING, because stopping in the middle of a handover could leave a batch half-released. The robot completes the local handover, then enters LINK_LOST (T27, T28). It cannot be entered from SAFE_MODE, because Safe Mode is already stricter.
- The robot never leaves LINK_LOST into NAVIGATING or DELIVERING. A deliberately abandoned delivery must be re-issued by an operator.

## 3.5 Verifier investigation

**(a) Can IDLE to DELIVERING happen?** Not in the corrected model. In the narrative there is no event that moves a waiting robot into DELIVERING, and T08 is marked Invalid. If a defect or a spurious DestinationReached event allowed it, the robot would try to hand over a batch at the warehouse (or an unknown place) without travelling. That violates FR-03 (a request must start navigation) and FR-05 (DELIVERING only from NAVIGATING at the destination). It also violates the narrative rule that a waiting robot must never begin delivering directly, and constraint C-15. The guard is that DestinationReached is only accepted in NAVIGATING.

**(b) AVOIDING_OBSTACLE with no way back.** The robot would be stuck: it could not resume navigation, so it could never reach DELIVERING. It would also be unable to return, because RETURNING is only reachable from NAVIGATING, DELIVERING and the abort path. The batch would sit in a moving-then-stalled chamber with no delivery completed, a liveness failure (deadlock) and a risk to the medicine. The fix is T03 and T10, plus a 120 s obstacle timeout that triggers an operator alert.

**(c) Can AVOIDING_OBSTACLE to DELIVERING occur?** Only if a DestinationReached event were processed while the robot is manoeuvring. That is unsafe because the robot would hand over a batch while still moving or positioned off the delivery point. T09 is Invalid. DestinationReached must be ignored in S3, and the robot may only reach DELIVERING after returning to NAVIGATING and then arriving (T03, T04). Testing: TC-07.

**(d) CriticalBattery in AVOIDING_OBSTACLE or DELIVERING (missing transition).** The original model only handles CriticalBattery in NAVIGATING (T07). In the other two states the event is undefined, so it would be silently ignored and the robot could run flat in the field. The fix: in AVOIDING_OBSTACLE set an abort flag, finish the manoeuvre, then go to RETURNING (T10, T11). In DELIVERING the handover is short, so it completes first and the robot then goes to RETURNING (T12, T05). If the battery then falls below the emergency level, SAFE_MODE (E8) pre-empts everything (T15, T16).

**Other findings.** (1) Two battery thresholds exist (return threshold and emergency level), so the model keeps them separate (E6 and E8) to avoid a race. (2) An unspecified timeout in AVOIDING_OBSTACLE was found. (3) A DeliveryRequest in SAFE_MODE, LINK_LOST or RETURNING must be rejected, so T29 and T30 were added.
