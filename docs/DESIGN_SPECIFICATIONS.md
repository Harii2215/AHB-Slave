# AHB Slave - Design Specifications

## 1. Overview
This document provides detailed specifications for the AMBA AHB-Lite Slave implementation.

## 2. AHB Protocol Compliance
- **Standard:** AMBA AHB-Lite Protocol (ARM IHI 0033)
- **Slave Type:** Generic memory slave
- **Address Bus Width:** 32 bits
- **Data Bus Width:** 32 bits
- **Memory Size:** 256 bytes

## 3. Input/Output Interface

### Inputs
| Signal | Width | Description |
|--------|-------|-------------|
| `clk` | 1 | System clock |
| `hresetn` | 1 | Active-low asynchronous reset |
| `hsel` | 1 | Slave select signal |
| `haddr` | 32 | Address bus |
| `hwdata` | 32 | Write data bus |
| `hwrite` | 1 | Write signal (1=write, 0=read) |
| `hsize` | 3 | Transfer size |
| `hburst` | 3 | Burst type |
| `htrans` | 2 | Transfer type |
| `hready` | 1 | Transfer ready (from master) |

### Outputs
| Signal | Width | Description |
|--------|-------|-------------|
| `hrdata` | 32 | Read data bus |
| `hready` | 1 | Slave ready signal |
| `hresp` | 2 | Slave response |

## 4. Transfer Sizes (HSIZE)

| Value | Name | Bits |
|-------|------|------|
| 3'b000 | BYTE | 8 |
| 3'b001 | HALFWORD | 16 |
| 3'b010 | WORD | 32 |

## 5. Burst Types (HBURST)

| Value | Name | Length |
|-------|------|--------|
| 3'b000 | SINGLE | 1 |
| 3'b001 | INCR | Unspecified |
| 3'b010 | WRAP4 | 4 |
| 3'b011 | INCR4 | 4 |
| 3'b100 | WRAP8 | 8 |
| 3'b101 | INCR8 | 8 |
| 3'b110 | WRAP16 | 16 |
| 3'b111 | INCR16 | 16 |

## 6. Transfer Types (HTRANS)

| Value | Name | Description |
|-------|------|-------------|
| 2'b00 | IDLE | No transfer |
| 2'b01 | BUSY | Slave busy |
| 2'b10 | NONSEQ | Non-sequential transfer |
| 2'b11 | SEQ | Sequential transfer |

## 7. Address Calculation

### INCR Bursts
Address increments by (1 << HSIZE) on each beat
```
addr_next = addr_current + (1 << hsize)
```

### WRAP Bursts
Address wraps at burst boundary
```
boundary = burst_length * (1 << hsize)
addr_next = (addr_current + (1 << hsize)) % boundary + base_addr
```

## 8. Response Types (HRESP)

| Value | Name | Description |
|-------|------|-------------|
| 2'b00 | OKAY | Transfer successful |
| 2'b01 | ERROR | Transfer error |

**Current Implementation:** Always returns OKAY

## 9. Memory Organization

- **Depth:** 256 locations
- **Width:** 8 bits per location
- **Addressing:** Byte-addressable
- **Initialization:** All locations = 8'h0C

## 10. FSM States

```
IDLE → CHECK_MODE → ADDR_DECODE → (WRITE/READ) → IDLE
```

### State Descriptions

1. **IDLE** - Waits for `hsel` and valid transfer
2. **CHECK_MODE** - Decodes `htrans` signal
3. **ADDR_DECODE** - Calculates next address based on burst type
4. **WRITE** - Performs write operation to memory
5. **READ** - Performs read operation from memory

## 11. Timing

- **Setup Time:** Configurable via synthesis constraints
- **Hold Time:** Configurable via synthesis constraints
- **Clock-to-Q:** Standard for target technology
- **Throughput:** 1 beat per clock cycle

## 12. Constraints

- Slave does not support early termination
- No error responses (always OKAY)
- Single-master operation assumed
- No multi-beat pipeline in current implementation
