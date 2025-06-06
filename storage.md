Here is your file converted to storage.md:

---

# Ethereum Storage and EVM Data Locations

In Ethereum, each smart contract at a specific address has its own storage, consisting of a key-value store that maps 256-bit words to 256-bit words. Data in the storage persists between function calls.

You can read and write from/to the contract storage. At the low level, the EVM opcodes used to do so are SSTORE and SLOAD.

In Ethereum smart contracts, SSTORE and SLOAD are low-level EVM opcodes used to **write** and **read** storage variables, respectively. They directly interact with the contract's persistent storage.

---

## 1. `SSTORE` (Storage Write)

- **Opcode:** `SSTORE(key, value)`
- **Solidity Equivalent:** Writing to a state variable.
- **Gas Cost:** Very expensive (up to 20,000 gas for new values, 5,000 for modifications).

### Use Cases

- Storing critical contract data (e.g., user balances, ownership).
- Updating contract state (e.g., voting results, DAO proposals).
- Implementing upgradeable storage patterns (like proxy contracts).

### Example in Solidity

```solidity
// Storing a value in contract storage
contract StorageExample {
    uint256 storedData; // This uses SSTORE behind the scenes

    function set(uint256 x) public {
        storedData = x; // SSTORE operation
    }
}
```

### Example in Inline Assembly (Yul)

```solidity
function setWithAssembly(uint256 key, uint256 value) public {
    assembly {
        sstore(key, value) // Direct EVM storage write
    }
}
```

---

## 2. `SLOAD` (Storage Read)

- **Opcode:** `SLOAD(key)`
- **Solidity Equivalent:** Reading a state variable.
- **Gas Cost:** ~800 gas (cheaper than SSTORE).

### Use Cases

- Retrieving stored data (e.g., checking a user’s token balance).
- Verifying contract state (e.g., checking if an address is an owner).
- Reading values for computations (e.g., fetching a price from storage).

### Example in Solidity

```solidity
contract StorageExample {
    uint256 storedData;

    function get() public view returns (uint256) {
        return storedData; // SLOAD operation
    }
}
```

### Example in Inline Assembly (Yul)

```solidity
function getWithAssembly(uint256 key) public view returns (uint256 result) {
    assembly {
        result := sload(key) // Direct EVM storage read
    }
}
```

---

## 3. Advanced Use Case: Efficient Storage Packing

EVM storage slots are **32 bytes** each, but you can pack smaller variables (e.g., `uint8`, `bool`) into a single slot to save gas.

### Example: Packing Multiple Variables

```solidity
contract StoragePacking {
    uint256 public a; // Slot 0
    uint128 public b; // Slot 1 (first half)
    uint128 public c; // Slot 1 (second half)

    function updateVars(uint128 newB, uint128 newC) public {
        // Read entire slot (SLOAD)
        uint256 slot1 = (uint256(c) << 128) | uint256(b);
        
        // Update packed values (SSTORE)
        b = newB;
        c = newC;
    }
}
```

**Gas Savings:**  
Instead of 2 SSTORE ops (40k gas), this uses **1 SSTORE (20k gas)**.

---

## 4. Security Considerations

1. **Reentrancy Risks**  
   Avoid reading storage after external calls (like in the classic DAO hack).

   ```solidity
   // UNSAFE: Vulnerable to reentrancy
   function unsafeWithdraw() public {
       uint256 balance = balances[msg.sender]; // SLOAD
       (bool success, ) = msg.sender.call{value: balance}("");
       require(success);
       balances[msg.sender] = 0; // SSTORE (too late!)
   }
   ```
   **Fix:** Use Checks-Effects-Interactions pattern.

2. **Gas Optimization**  
   Cache storage variables in memory if used multiple times:

   ```solidity
   function gasOptimized() public {
       uint256 cachedVar = storedData; // 1 SLOAD
       if (cachedVar > 10) {
           cachedVar *= 2;
           storedData = cachedVar; // 1 SSTORE
       }
   }
   ```

---

## 5. When to Use `SSTORE`/`SLOAD` in Assembly

- **Gas Optimization:** When you need fine-grained control over storage (e.g., packing variables).
- **Proxy Contracts:** Storage manipulation in upgradeable contracts.
- **Low-Level Hacks:** Building custom data structures (e.g., bitmaps, compact arrays).

### Example: Bitmask Storage

```solidity
contract Bitmask {
    uint256 public packedFlags;

    function setFlag(uint8 position) public {
        uint256 mask = 1 << position;
        packedFlags |= mask; // SSTORE
    }

    function getFlag(uint8 position) public view returns (bool) {
        uint256 mask = 1 << position;
        return (packedFlags & mask) != 0; // SLOAD
    }
}
```

---

## Summary

| Opcode   | Purpose            | Gas Cost        | Use Case                        |
|----------|--------------------|-----------------|----------------------------------|
| `SSTORE` | Write to storage   | High (5k–20k)   | Updating state, storing data     |
| `SLOAD`  | Read from storage  | Medium (~800)   | Reading state, checking conditions|

**Key Takeaways:**

- Use `SSTORE` sparingly (it’s expensive).
- Optimize by packing data into slots.
- Prefer memory caching for repeated reads.
- Avoid storage ops after external calls.

These opcodes are fundamental to Ethereum smart contracts and are used in almost every DeFi protocol, NFT, or DAO.

---
