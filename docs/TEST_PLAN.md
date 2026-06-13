# AHB Slave - Test Plan

## Overview

This document outlines the verification strategy and test cases for the AHB-Lite Slave design.

## 1. Verification Strategy

### Coverage Goals
- **Functional Coverage:** 100% of protocol features
- **Code Coverage:** >90% statement, branch, and toggle coverage
- **Scenarios:** All burst types, transfer sizes, and edge cases

### Verification Levels
1. **Unit Level** - Individual components
2. **Integration Level** - Complete slave with testbench
3. **Protocol Level** - AHB protocol compliance

## 2. Test Categories

### 2.1 Single Transfer Tests

| Test ID | Description | Transfer Type | Expected Result |
|---------|-------------|----------------|------------------|
| TEST_001 | Single byte write | SINGLE, BYTE | Data written to memory |
| TEST_002 | Single halfword write | SINGLE, HALFWORD | Data written to memory |
| TEST_003 | Single word write | SINGLE, WORD | Data written to memory |
| TEST_004 | Single byte read | SINGLE, BYTE | Correct data read |
| TEST_005 | Single halfword read | SINGLE, HALFWORD | Correct data read |
| TEST_006 | Single word read | SINGLE, WORD | Correct data read |

### 2.2 INCR Burst Tests

| Test ID | Description | Burst Type | Length | Expected Result |
|---------|-------------|------------|--------|------------------|
| TEST_010 | INCR4 byte writes | INCR4, BYTE | 4 | All data written correctly |
| TEST_011 | INCR8 word writes | INCR8, WORD | 8 | All data written correctly |
| TEST_012 | INCR16 halfword writes | INCR16, HALFWORD | 16 | All data written correctly |
| TEST_013 | INCR4 byte reads | INCR4, BYTE | 4 | All data read correctly |
| TEST_014 | INCR8 word reads | INCR8, WORD | 8 | All data read correctly |

### 2.3 WRAP Burst Tests

| Test ID | Description | Burst Type | Expected Behavior |
|---------|-------------|------------|--------------------|
| TEST_020 | WRAP4 write | WRAP4, BYTE | Address wraps at 4-byte boundary |
| TEST_021 | WRAP8 write | WRAP8, HALFWORD | Address wraps at 8 location boundary |
| TEST_022 | WRAP16 write | WRAP16, WORD | Address wraps at 16 location boundary |
| TEST_023 | WRAP4 read | WRAP4, BYTE | Correct wrapping behavior |
| TEST_024 | WRAP8 read | WRAP8, WORD | Correct wrapping behavior |

### 2.4 Mixed Scenarios

| Test ID | Description | Scenario |
|---------|-------------|----------|
| TEST_030 | Back-to-back transfers | Multiple bursts without gaps |
| TEST_031 | Interleaved read/write | Alternating read and write bursts |
| TEST_032 | Different sizes | Variable transfer sizes in sequence |
| TEST_033 | Boundary crossing | Transfers crossing memory boundaries |
| TEST_034 | Reset during transfer | System reset in middle of burst |

### 2.5 Protocol Compliance Tests

| Test ID | Description | Protocol Check |
|---------|-------------|----------------|
| TEST_040 | HREADY assertion | Slave asserts HREADY correctly |
| TEST_041 | HRESP response | Slave returns correct HRESP value |
| TEST_042 | Data stability | Data remains stable on bus |
| TEST_043 | Address decode | Addresses decoded correctly |
| TEST_044 | Timing compliance | All timing paths met |

## 3. Test Execution

### 3.1 Simulation Environment

```bash
# Compile design and testbench
vlog -sv src/ahb_slave.v tb/ahb_tb_pkg.sv

# Run simulation
vsim -c work.tb -do "run -all"

# Generate coverage report
vcover report -detail
```

### 3.2 Coverage Collection

Enable coverage collection:
```bash
vsim -c work.tb -coverage -do "run -all; coverage save work.ucdb"
```

## 4. Expected Results

### 4.1 Functional Verification
- All write operations store data correctly
- All read operations retrieve correct data
- Address calculations for INCR bursts are correct
- Address wrapping for WRAP bursts is correct
- Protocol signals behave according to AHB specification

### 4.2 Edge Cases
- Reset during active transfer → Returns to IDLE state
- Maximum burst length → Handles correctly
- Minimum transfer size (byte) → Works correctly
- Memory boundary crossing → Wraps or handles gracefully

## 5. Regression Testing

Run full regression suite before each release:
```bash
# Regression script
./scripts/regression.sh
```

Expected: All tests pass with 0 failures

## 6. Documentation

- Document any failures with root cause analysis
- Update design specifications if behavior changes
- Maintain test case documentation
- Update this test plan as new tests are added

## 7. Performance Metrics

| Metric | Target | Method |
|--------|--------|--------|
| Code Coverage | >90% | UCDB reports |
| Test Duration | <60s | Benchmark |
| Setup Time | <5s | Benchmark |
| Memory Usage | <500MB | Tool report |

## 8. Continuous Integration

Tests should be automated to run:
- On every commit to main branch
- On every pull request
- Nightly full regression
- Before each release

## 9. Sign-Off Criteria

Verification is complete when:
- [ ] All test cases pass
- [ ] Code coverage >90%
- [ ] No critical issues
- [ ] Documentation updated
- [ ] Peer review completed
