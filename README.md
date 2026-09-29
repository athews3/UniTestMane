# Universal Test Manager

## Overview

Universal Test Manager (UniTestMan) is a reusable PLC/HMI framework for automated door cycling and product validation testing built on the Mitsubishi automation platform.

The system centralizes test execution, state management, operator interaction, diagnostics, and test analytics into a single architecture that supports multiple test types while maintaining a consistent technician experience.

This system is currently in use at Allegion HRTC.

**Technologies**

- GX Works3 — PLC programming and control logic
- GT Designer3 — GOT HMI development
- Mitsubishi FX5 Platform
- Future Ignition Integration (SCADA / Historian / Database)

The architecture is specifically designed to:

- Support 25+ independent test stations
- Minimize code duplication
- Standardize test execution
- Simplify HMI operation
- Enable future database and cloud connectivity
- Maintain deterministic PLC scan behavior

---

# System Architecture

## PLC Layer (GX Works3)

The PLC is the authoritative source of system state.

Responsibilities include:

- Test execution
- State management
- Safety and interlock handling
- Hold management
- Failure tracking
- Completion forecasting
- Cycles-per-minute calculation
- Next Pause management
- HMI data generation

The PLC owns all control decisions.

The HMI never determines machine state.

---

## HMI Layer (GT Designer3)

The GOT HMI provides the operator interface.

Responsibilities:

- Test selection
- Start / Stop control
- Hold entry and release
- Hold reason selection
- Parameter editing
- Status indication
- Completion forecast viewing
- Diagnostics and fault visibility

The HMI reads PLC-owned states rather than inferring behavior from button states.

---

# Universal Test Manager (UniTestMan)

UniTestMan serves as the orchestration layer for all test execution.

Responsibilities:

- Test selection management
- Run state management
- Hold state management
- Safety interlocks
- E-stop handling
- Failure aggregation
- HMI synchronization
- Utility function block execution

Only one test executes at a time.

---

# Test Execution Architecture

The active test is selected through `HMI_SelectedTest`.

UniTestMan dispatches execution using a structured `CASE` statement.

Current test families:

| ID | Test |
|----|--------|
| 1 | Trim |
| 2 | Exit |
| 4 | QEL |
| 5 | Mag Lock |
| 6 | Chexit |

Execution model:

```pascal
CASE HMI_SelectedTest OF
    1: TrimInst();
    2: ExitInst();
    4: QELInst();
    5: MagLockInst();
    6: ChexitInst();
END_CASE;
```

Each test FB is responsible for:

- Physical sequencing
- Sensor validation
- Cycle completion determination
- Pass/fail evaluation
- Test-specific logic

UniTestMan is responsible for all higher-level orchestration.

---

# Utility Function Blocks

## CPM

Calculates current cycles-per-minute.

Inputs:

```text
CycleCount
Manager_Executing
```

Outputs:

```text
CPM
```

Used by:

- HMI display
- Completion forecasting
- Future reporting systems

---

## Completion

Calculates projected completion date and time.

Inputs:

```text
CycleCount
CycleTarget
CPM
```

Outputs:

```text
Comp_Year
Comp_Month
Comp_Day
Comp_Hour
Comp_Min
```

The completion forecast is RTC-based and performs calendar rollover logic for:

- Minutes
- Hours
- Days
- Months
- Years

The displayed completion time is an estimate and varies with live CPM performance.

---

## NextPause

Allows technicians to stop testing at predetermined cycle intervals.

Example:

```text
Current Cycles = 950

PauseSet = 1000
```

When:

```text
CycleCount >= PauseSet
```

the system:

1. Stops execution
2. Clears PauseSet
3. Returns to Idle
4. Waits for technician action

Typical use:

```text
Run 250k cycles
Inspect product
Enter next checkpoint
Resume testing
```

---

# State Management

The Universal Test Manager uses a state-driven architecture.

Primary states:

| State | Purpose |
|---------|----------|
| Running | Test actively executing |
| Idle | Ready for execution |
| Held | Technician-imposed hold |
| Paused | User stopped execution |
| Complete | Cycle goal reached |
| Faulted | Failure condition active |
| E-Stop | Safety event latched |

State derivation is PLC-owned.

No HMI screen directly drives these states.

---

# Hold System

The hold system replaced the earlier reservation architecture.

Purpose:

- Pause testing for documented reasons
- Provide traceability
- Improve reporting and future historian integration

Workflow:

1. Technician selects Hold Reason
2. Hold reason writes a numeric code
3. Hold latches
4. Test stops
5. Hold lamp illuminates
6. Technician resolves issue
7. Technician clears hold

Examples:

- Sample Broken
- Fixture Adjustment
- Hardware Service
- Investigation Required

Hold reasons are designed for future Ignition historian logging.

---

# Emergency Stop Handling

The system uses a latched E-stop architecture.

Safety Input:

```text
X17
```

Behavior:

```text
X17 drops
↓
EStop_Latch sets
↓
Run_Latch resets
↓
Internal_Stop sets
```

The system cannot restart automatically.

Operator action required:

```text
Restore safety condition
↓
Press Reset
↓
Restart test
```

The HMI displays the latched condition through:

```text
HMI_EStop
```

---

# Failure Tracking

The system tracks two independent metrics.

## Total Failures

```text
FailToRet
```

Tracks all failed cycles during the current test run.

---

## Consecutive Failures

```text
ConsecutiveFailCnt
```

Tracks uninterrupted failure streaks.

Logic:

```text
Pass
→ counter resets

Fail
→ counter increments
```

When:

```text
ConsecutiveFailCnt >= 5
```

the system:

- Trips failure lockout
- Stops execution
- Requires intervention

---

# Pass / Fail Pulse Architecture

Every test FB emits standardized results.

Outputs:

```text
Pass_Pulse
Fail_Pulse
```

Requirements:

- One scan only
- Mutually exclusive
- Generated only at cycle completion

Benefits:

- Unified failure handling
- Reusable manager logic
- Consistent reporting

---

# Stall Detection

A cycle watchdog protects against stalled sequences.

Purpose:

Detect situations where:

- PLC still believes a test is running
- Physical hardware has stopped progressing

Example:

```text
Cylinder disconnected
Sensor never changes
Sequence waits forever
```

Behavior:

```text
No completed cycle for 5 seconds
↓
Watchdog expires
↓
Run_Latch resets
↓
System returns to Idle
```

This prevents false "Running" indications.

---

# HMI Features

Technicians can:

- Select tests
- Start / stop testing
- Enter hold reasons
- Clear holds
- Configure pause checkpoints
- Edit approved timing values
- View cycle counts
- View CPM
- View projected completion date
- View active test selection
- Observe system state
- View E-stop status
- Review fault conditions

Technicians are not expected to modify PLC logic.

---

# Completion Forecasting

The system estimates completion using:

```text
Current Cycle Count
Cycle Target
Cycles Per Minute
Current RTC Time
```

Displayed on HMI:

```text
Month
Day
Year
Hour
Minute
```

Example:

```text
07/22/2026 14:30
```

This value is predictive and can change as CPM changes.

It should be interpreted as:

```text
Projected Completion
```

rather than guaranteed completion.

---

# Database / Ignition Readiness

The project is being prepared for future Ignition integration.

Design goals:

- Stable global labels
- Clean device assignments
- Meaningful naming conventions
- Minimal device-level dependencies

Planned historian/database data:

- Operator ID
- Test ID
- Cycle Count
- Cycle Target
- CPM
- Completion Forecast
- Hold Reason
- Failure Count
- Consecutive Failure Count
- E-stop Events
- Test Start Time
- Test Stop Time
- Test Completion Time

Ignition should consume named labels rather than raw device addresses wherever possible.

---

# Repository Structure

```text
/PLC
    GX Works3 project

/HMI
    GT Designer3 project

/Docs
    Design documentation
    Device maps
    Architecture specifications
```

---

# Requirements

## Software

- GX Works3
- GT Designer3

## Hardware

- Mitsubishi FX5 PLC
- Mitsubishi GOT HMI
- Associated test station hardware

---

# Setup

## PLC

1. Open GX Works3
2. Load project
3. Verify:
   - PLC model
   - Device/memory assignments
   - Communication settings
4. Compile (rebuild)
5. Write to PLC

---

## HMI

1. Open GT Designer3
2. Load project
3. Verify communication settings
4. Download to GOT

---

# Design Principles

## Deterministic

All behavior must remain predictable under scan execution.

## Modular

Individual tests should be isolated inside their own FBs.

## Reusable

Manager logic should remain test-independent.

## Observable

System state should be visible to technicians and future historians.

## Maintainable

Labels and logic should be understandable without requiring deep PLC expertise.

## Scalable

Architecture should support additional tests and stations without requiring redesign.

---

# Development Guidelines

When implementing changes:

1. Preserve state-machine architecture.
2. Maintain consistent label naming.
3. Keep HMI-facing variables global and documented.
4. Keep scratch variables local.
5. Preserve deterministic execution order.
6. Avoid unsupported IEC datetime assumptions.
7. Validate all device assignments against GT Designer references.
8. Design all future data with Ignition consumption in mind.

---

# Current Roadmap

### Completed

- Universal test dispatch architecture
- Hold system
- E-stop latching
- Consecutive failure tracking
- Completion forecasting
- Active test indicators
- NextPause checkpoint system
- Cycle watchdog stall detection
- Standardized pass/fail pulse interface

### Planned

- Ignition integration
- Historian logging
- Operator accountability reporting
- Enhanced diagnostics
- Unified test parameter management
- Station-to-station analytics
- Statistical performance dashboards

---

# Author

Alex Thews  
Product Assurance Engineering Intern
