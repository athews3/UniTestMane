# Universal Test Manager

## Overview
Universal Test Manager is a PLC-based test automation framework built on the Mitsubishi automation stack. It integrates control logic, operator interface, and test execution into a structured and reusable system.

**Technologies:**
- GX Works3 — PLC programming
- GT Designer3 — HMI (GOT interface)

The system is designed for deterministic execution, modular expansion, and efficient operator interaction.

---

## Architecture

### PLC Layer (GX Works3)
- Implements test sequencing and control logic  
- Handles interlocks, timing, and state management  
- Interfaces with physical I/O and external hardware  

### HMI Layer (GT Designer3)
- Provides operator control and visualization  
- Displays system status, alarms, and diagnostics  
- Enables parameter input and manual overrides  

### Test Management Structure
- Abstracted sequencing model  
- Standardized logic blocks for reusability  
- Scalable design for multiple test configurations  

---

## Features

- Modular test sequence framework  
- Deterministic PLC execution model  
- Configurable parameters via HMI  
- Real-time status and diagnostics  
- Integrated pass/fail evaluation  
- Expandable architecture  

---

## Repository Structure

```text
/PLC        GX Works3 project files
/HMI        GT Designer3 project files
/Docs       Documentation (optional)
```

---

## Requirements

### Software
- GX Works3  
- GT Designer3 (part of GT Works3)

### Hardware
- Mitsubishi PLC (iQ-R, iQ-F, or compatible)  
- Mitsubishi GOT HMI  

---

## Setup

### PLC
1. Open GX Works3  
2. Load `/PLC` project  
3. Verify:
   - PLC model
   - I/O mapping
   - Communication parameters  
4. Compile and download to target PLC  

### HMI
1. Open GT Designer3  
2. Load `/HMI` project  
3. Verify communication settings  
4. Download to GOT  

---

## Operation

1. Power PLC and HMI  
2. Establish communication  
3. Initiate test sequence via HMI  
4. PLC executes:
   - Output control  
   - Input monitoring  
   - Condition evaluation  
5. Results displayed on HMI  

---

## Development

### PLC Changes
- Modify logic in GX Works3  
- Maintain structured, state-based design  
- Validate before deployment  

### HMI Changes
- Update screens in GT Designer3  
- Keep device addressing consistent with PLC  
- Preserve clear operator flow  

---

## Design Principles

- Modularity  
- Deterministic behavior  
- Clarity for operators  
- Maintainability  

---

## Notes

- Mitsubishi project files are proprietary; use compatible software versions  
- Validate hardware mappings before deployment  
- Maintain version control discipline for all changes  

---

## License

Add license as applicable.

---

## Author

Alex Thews  
Product Assurance Engineering Intern  
