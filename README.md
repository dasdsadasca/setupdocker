# Web3 Security Testing Environment

A Docker-based environment for Web3 security testing, including fuzzing, static analysis, and symbolic execution tools.

## Quick Start

1. Create a new directory for your project
2. Create a `contracts` folder
3. Copy this `docker-compose.yml`:
```yaml
version: '3.8'

services:
  eth-security:
    image: trailofbits/eth-security-toolbox
    volumes:
      - ./contracts:/contracts
    working_dir: /contracts
    stdin_open: true
    tty: true
    command: >
      bash -c "
        solc-select install 0.8.20 &&
        solc-select use 0.8.20 &&
        /bin/bash
      "
```

4. Start the container:
```bash
docker-compose up -d
docker-compose exec eth-security bash
```

## Available Tools

### 1. Echidna (Fuzzing)

Echidna is a powerful fuzzing tool for smart contracts. It tests your contracts by trying to find inputs that violate your properties.

Basic usage:
```bash
cd /contracts
echidna YourContract.sol --test-mode assertion
```

Advanced options:
```bash
# Run with more test cases
echidna YourContract.sol --test-mode assertion --test-limit 5000

# Generate test corpus
echidna YourContract.sol --test-mode assertion --corpus-dir corpus

# Run with multiple workers
echidna YourContract.sol --test-mode assertion --workers 4

# Run with specific sequence length
echidna YourContract.sol --test-mode assertion --seq-len 100
```

Writing Echidna tests:
```solidity
// In your contract
function echidna_test_property() public view returns (bool) {
    // Your property here
    return someCondition;
}
```

### 2. Slither (Static Analysis)

Slither performs static analysis on your contracts to find vulnerabilities.

Basic usage:
```bash
cd /contracts
slither YourContract.sol
```

Advanced options:
```bash
# Generate a detailed report
slither YourContract.sol --json report.json

# Check for specific vulnerabilities
slither YourContract.sol --detect reentrancy-eth

# Generate inheritance graph
slither YourContract.sol --print inheritance-graph

# Run specific detectors
slither YourContract.sol --detect unchecked-transfer
```

### 3. Manticore (Symbolic Execution)

Manticore performs symbolic execution to find potential vulnerabilities.

Basic usage:
```bash
cd /contracts
manticore YourContract.sol
```

Advanced options:
```bash
# Run with specific timeout
manticore YourContract.sol --timeout 300

# Generate test cases
manticore YourContract.sol --testcase

# Run with specific gas limit
manticore YourContract.sol --gaslimit 1000000
```

## Example Contract with Tests

Here's an example of a contract with Echidna tests:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Example {
    uint256 public value;
    
    function setValue(uint256 _value) public {
        value = _value;
    }
    
    // Echidna test functions
    function echidna_test_value() public view returns (bool) {
        return value >= 0;
    }
    
    function echidna_test_value_bounds() public view returns (bool) {
        return value <= type(uint256).max;
    }
}
```

## Common Use Cases

### Fuzzing for Reentrancy
```solidity
// Test for reentrancy vulnerability
function echidna_test_reentrancy() public view returns (bool) {
    return address(this).balance >= balances[msg.sender];
}
```

### Testing Access Control
```solidity
// Test for access control
function echidna_test_access() public view returns (bool) {
    return owner == msg.sender || !isRestricted;
}
```

### Testing State Invariants
```solidity
// Test state invariants
function echidna_test_invariants() public view returns (bool) {
    return totalSupply == sumOfBalances();
}
```

## Tips and Best Practices

1. Always start with basic tests and gradually add more complex ones
2. Use `--test-limit` to control the number of test cases
3. Use `--corpus-dir` to save and reuse test cases
4. Combine multiple tools for better coverage:
   ```bash
   # Run all tools
   slither YourContract.sol
   echidna YourContract.sol
   manticore YourContract.sol
   ```

5. Use Echidna's test modes:
   - `assertion`: Test for assertion violations
   - `optimization`: Find optimal values
   - `dtest`: Run deterministic tests

## Troubleshooting

1. If Echidna fails to start:
   ```bash
   solc-select use 0.8.20  # Set Solidity version
   ```

2. If tests are slow:
   ```bash
   # Reduce test limit
   echidna YourContract.sol --test-limit 1000
   ```

3. If you need more detailed output:
   ```bash
   # Add verbose flag
   echidna YourContract.sol --verbose
   ```
