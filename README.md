# 4-bit Conditional Arithmetic Unit

## Overview
This circuit implements an arithmetic unit that performs different operations based on the relationship between two 4-bit binary numbers (A and B), where A ≥ B is always true. The design optimizes component usage while implementing complex conditional arithmetic logic.

## Functionality
The circuit performs the following conditional operations:

1. **When A = B**:  
   `Z = A + 1`

2. **When A > B**:  
   `Z = (A - B) - 1`

[![Circuit Overview](media/circuit-overview.png)](media/circuit-overview.png)

## Design Architecture

### 1. Equality Detection Circuit
- **Components**: 7486 (Quad XOR) + 7432 (Triple OR)
- **Logic**: Bitwise XOR between A and B determines equality
- **Output**: `ANEQB` (A Not EQual to B) = 1 when A ≠ B
- **Implementation**: 4 XOR gates feed into a 3-input OR gate

[![Equality Detection](media/equality-detection.png)](media/equality-detection.png)

### 2. Input Processing Stage
- **Components**: 7404 (Hex NOT) + 7408 (Quad AND)
- **Logic**: `B_processed = ANEQB * B'`
  - When A = B: All bits of B are forced to 0
  - When A > B: Each bit of B is inverted
- **Efficiency**: Utilizes 4 NOT gates and 4 AND gates

[![Input Processing](media/input-processing.png)](media/input-processing.png)

### 3. Core Arithmetic Unit
- **Component**: 7483 4-bit Binary Adder
- **Input A**: Direct connection (A₃A₂A₁A₀)
- **Input B**: Connected to processed B values
- **Carry-in (C₀)**: Controlled by `ANEQB'` signal
  - When A = B: `C₀ = 1` → Implements `A + 1`
  - When A > B: `C₀ = 0` → Implements `A + B'`

[![Core Arithmetic](media/core-arithmetic.png)](media/core-arithmetic.png)

### 4. Carry-Out Management
During testing, the circuit exhibited an unexpected carry-out (C₄) when A ≠ B. The ideal solution would use an additional AND gate to conditionally suppress C₄. However, to optimize component usage and reduce IC count, an alternative implementation was developed.

**Implementation**: 
- Utilizes the remaining NOT gate from the 7404 to generate `ANEQB'`
- Implements `C₄_corrected = C₄ · ANEQB'` using discrete components
- Employs diode-resistor logic with a carefully selected 1kΩ pull-up resistor
- Ensures sufficient current for downstream 7448 7-segment decoders

[![Carry Management](media/carry-management.png)](media/carry-management.png)

## Mathematical Formulation

### Case A = B:
```
Z = A + 1
  = A + 0 + 1 (B_processed = 0, C₀ = 1)
```

### Case A > B:
```
Z = (A - B) - 1
  = A + (~B + 1) - 1  (two's complement representation)
  = A + ~B            (simplification)
  = A + B_processed   (B_processed = ~B, C₀ = 0)
```

Both cases elegantly reduce to the same hardware operation:
```
Z = A + B_processed + C₀
```

## Truth Table (Selected Test Cases)

| Condition | A (Hex) | B (Hex) | Operation | Result (Hex) |
|-----------|---------|---------|-----------|--------------|
| A = B     | 0x0     | 0x0     | 0 + 1     | 0x1          |
| A = B     | 0x5     | 0x5     | 5 + 1     | 0x6          |
| A = B     | 0xF     | 0xF     | F + 1     | 0x0*         |
| A > B     | 0x5     | 0x3     | (5-3)-1   | 0x1          |
| A > B     | 0x8     | 0x6     | (8-6)-1   | 0x1          |
| A > B     | 0xF     | 0x0     | (F-0)-1   | 0xE          |

*Note: Result with C₄ = 1 (16 in decimal)*

[![Truth Table](media/truth-table.png)](media/truth-table.png)

## Simulation Results

[![Timing Simulation](media/timing-simulation.png)](media/timing-simulation.png)

[![Waveform Analysis](media/waveform-analysis.png)](media/waveform-analysis.png)

## Component Utilization

| IC | Type | Usage | Purpose |
|----|------|-------|---------|
| 7486 | Quad XOR | 4/4 gates | Equality comparison |
| 7432 | Quad OR | 3/4 gates | Combine comparison results |
| 7404 | Hex NOT | 5/6 gates | Signal inversion |
| 7408 | Quad AND | 4/4 gates | Conditional B processing |
| 7483 | 4-bit Adder | Full | Arithmetic core |
| Discrete | Diode + Resistor | Custom | Carry-out conditioning |

## Design Principles

This implementation demonstrates several key engineering principles:
- **Component Efficiency**: Maximizing gate utilization across ICs
- **Functional Integration**: Combining multiple operations in single stages
- **Resource Optimization**: Using discrete components where appropriate to reduce IC count
- **Signal Integrity**: Careful selection of resistor values for proper current drive

The circuit successfully implements complex conditional arithmetic using minimal TTL components while maintaining reliable operation across all input conditions.
