# Task 4. Operation Identification and Operation Schema (Conservation Chamber)

## 4.1 Chamber states (used for the links to Task 3)

CH-OFF, CH-SELFCHECK, CH-MONITORING, CH-CONSERVING, CH-SUSPENDED (door open or vibration), CH-PROTECTION, CH-EMERGENCY_POWER, CH-SHUTDOWN.

## 4.2 Operations (22)

| ID | Operation | What it does |
|---|---|---|
| OP-01 | RunSelfCheck | Tests all essential sensors at power-on |
| OP-02 | LoadBatchProfile | Reads batch ID and temperature/humidity limits |
| OP-03 | CheckDoorClosed | Reads the door sensor |
| OP-04 | StartActiveConservation | Starts regulation when door is closed and profile is loaded |
| OP-05 | SampleEnvironment | Reads temperature, humidity, vibration and power |
| OP-06 | CorrectTemperature | Drives heating or cooling toward the profile range |
| OP-07 | CorrectHumidity | Drives humidifier or dehumidifier toward the range |
| OP-08 | VerifyRecovery | Confirms from sensors that values returned to range and stayed there |
| OP-09 | EnterProtectionMode | Isolates the batch after correction fails |
| OP-10 | ReduceLightExposure | Dims or closes light sources |
| OP-11 | AlertOperator | Sends an alarm to Mission Control and the local display |
| OP-12 | SuspendRiskyActivities | Halts door unlock, loading and correction when vibration is high |
| OP-13 | ResumeAfterStabilisation | Resumes after vibration stays below threshold for the stabilisation period |
| OP-14 | SuspendOnDoorOpen | Suspends normal operation when the door opens during conservation |
| OP-15 | ResumeAfterVerification | Resumes after door is closed and conditions and sensors are verified |
| OP-16 | SwitchToEmergencyPower | Switches to the backup supply on power loss |
| OP-17 | RecordIncidentAndShutdown | Logs the incident and shuts down safely when no backup exists |
| OP-18 | AuthoriseBatchRemoval | Allows removal or handover only when the chamber is safe and no protection response is active |
| OP-19 | LogEvent | Writes a timestamped record to non-volatile storage |
| OP-20 | AuthenticateAndAuthoriseCommand | Checks session and role for each command (CR-03) |
| OP-21 | ExecuteCommand | Runs a validated command against the robot state model |
| OP-22 | ReportStatus | Sends telemetry and command results to Mission Control |

## 4.3 Operation schemas

**OP-04 StartActiveConservation**
- Description: begins regulating temperature and humidity for the loaded batch.
- Inputs: batch profile (ID, T min/max, H min/max), door status.
- Outputs: conservation-started flag.
- Reads: Door_Status, Profile_Loaded, Essential_Sensors_OK, current temperature and humidity.
- Changes: Chamber_State to CH-CONSERVING, regulator set-points.
- Sends/Alerts: status 'Conserving' to Mission Control; event logged.
- Assumes: Door_Closed, Profile_Loaded and Essential_Sensors_OK are all TRUE; chamber in CH-MONITORING.
- Result: Chamber_State = CH-CONSERVING, set-points equal to the profile.

**OP-06 CorrectTemperature**
- Description: drives temperature back into the profile range.
- Inputs: current temperature, T min/max.
- Outputs: actuator command (heat or cool), correction-started time.
- Reads: temperature sensor, profile, recovery timer.
- Changes: heater/cooler output, Recovery_Timer.
- Sends/Alerts: 'Correcting' event; alert if the timer expires.
- Assumes: Chamber in CH-CONSERVING, temperature outside range, sensor valid.
- Result: the actuator is commanded, and recovery is only claimed after OP-08 confirms it.

**OP-08 VerifyRecovery**
- Description: confirms from sensor readings that correction worked.
- Inputs: sensor readings, range, 60 s stability period.
- Outputs: Recovered = TRUE/FALSE.
- Reads: temperature and humidity sensors, Recovery_Timer.
- Changes: Recovery_Timer reset, Chamber_State.
- Sends/Alerts: success event, or timeout to OP-09.
- Assumes: correction (OP-06 or OP-07) has started.
- Result: Recovered = TRUE only if readings stayed in range for 60 s, else timeout leads to protection mode after 10 min.

**OP-09 EnterProtectionMode**
- Description: protects the batch when correction fails.
- Inputs: Recovery timeout signal.
- Outputs: Protection_Active = TRUE.
- Reads: Recovery_Timer, door state.
- Changes: Chamber_State to CH-PROTECTION, light level.
- Sends/Alerts: calls OP-10 and OP-11 (alert within 5 s); logs event; may raise SafeModeTrigger for the robot if temperature is critical.
- Assumes: recovery period elapsed with value still out of range.
- Result: Chamber_State = CH-PROTECTION, light reduced, operator alerted, batch removal blocked.

**OP-14 SuspendOnDoorOpen**
- Description: stops normal operation when the door opens during conservation.
- Inputs: door-open signal.
- Outputs: Operation_Suspended = TRUE.
- Reads: Door_Status, Chamber_State.
- Changes: Chamber_State to CH-SUSPENDED, regulator mode.
- Sends/Alerts: alert to the operator; event logged; robot must remain stopped.
- Assumes: Chamber_State = CH-CONSERVING and Door_Status = OPEN.
- Result: Chamber_State = CH-SUSPENDED within 1 s, no resume until OP-15.

**OP-16 SwitchToEmergencyPower**
- Description: keeps the chamber powered when the main supply fails.
- Inputs: main power status, backup availability.
- Outputs: Power_Source.
- Reads: Main_Power, Backup_Available, backup voltage.
- Changes: Power_Source to BACKUP, Chamber_State to CH-EMERGENCY_POWER.
- Sends/Alerts: alarm 'Power loss'; event logged; if no backup, calls OP-17.
- Assumes: Main_Power = LOST and Backup_Available = TRUE.
- Result: chamber powered from backup within 1 s with sensors still readable.

**OP-18 AuthoriseBatchRemoval**
- Description: permits taking the batch out.
- Inputs: removal request, authenticated operator or clinic confirmation.
- Outputs: Removal_Allowed.
- Reads: Chamber_Safe, Protection_Active, Door_Status.
- Changes: Door lock released.
- Sends/Alerts: refusal message or confirmation; event logged.
- Assumes: robot stationary (DELIVERING or IDLE at the warehouse).
- Result: lock released only if Chamber_Safe and no protection response is active.

## 4.4 State changes and links to the Task 3 model

| Operation | Chamber change | Link to robot model (Task 3) |
|---|---|---|
| OP-14 (door opened during conservation) | CH-CONSERVING to CH-SUSPENDED | Robot must not move: constraint C-09 means NAVIGATING, AVOIDING_OBSTACLE and RETURNING cannot continue with the door open. The robot stops and the alert is sent. If the door opens at a legitimate handover, the robot is already in DELIVERING and stationary (C-10). Resume (OP-15) happens only after closing and verification. |
| OP-09 (correction fails) | CH-CONSERVING to CH-PROTECTION | If chamber temperature exceeds the critical threshold, the chamber raises E8 SafeModeTrigger, and the robot goes from any state to SAFE_MODE (T13 to T17) within 3 s. Batch removal (OP-18) stays blocked (C-11). |
| OP-16 (power loss) | CH-CONSERVING to CH-EMERGENCY_POWER | If battery falls below the emergency level while on backup, E8 fires and the robot enters SAFE_MODE. |

## 4.5 Operations whose postcondition must be verified by sensors

1. **OP-06/OP-08 correction of temperature or humidity.** Switching on the heater or cooler only means a command was issued. A faulty actuator, a stuck valve, a dead sensor or a thermal lag can leave the value out of range, and a medicine batch could be spoiled while the system believes it is fine. So the postcondition is checked by readings in range for 60 consecutive seconds, and a timeout forces protection mode.
2. **OP-15 resume after door open.** A closed-door command or a door-closed signal does not prove the interior is back to range, or that the sensors work. A door seal fault, an ajar door or an interior warm-up could go unnoticed. The chamber therefore resumes only after the door sensor reads closed and temperature, humidity and sensor self-checks are verified from live readings.

The same applies to OP-16, where backup voltage should be read rather than assumed after the switching command.
