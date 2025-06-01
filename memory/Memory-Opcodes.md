# Examples of `MLOAD`, `MSTORE`, and `MSTORE8` Opcodes in Solidity

These are low-level EVM opcodes that interact with memory. Here are examples demonstrating their usage:

## 1. Basic MLOAD and MSTORE
```solidity
function memoryOps() public pure returns (uint256) {
    // Allocate memory pointer
    uint256 freeMemPtr;
    
    assembly {
        // Get current free memory pointer
        freeMemPtr := mload(0x40)
        
        // Store value 0x123 at freeMemPtr
        mstore(freeMemPtr, 0x123)
        
        // Load the value back
        let loadedValue := mload(freeMemPtr)
        
        // Update free memory pointer (0x40 is special slot for free mem ptr)
        mstore(0x40, add(freeMemPtr, 0x20))
        
        // Return the loaded value
        return(freeMemPtr, 0x20)
    }
}
```

## 2. MSTORE8 (stores a single byte)
```solidity
function storeBytes() public pure returns (bytes32) {
    bytes32 result;
    
    assembly {
        // Store individual bytes
        mstore8(0x00, 0x41) // 'A'
        mstore8(0x01, 0x42) // 'B'
        mstore8(0x02, 0x43) // 'C'
        
        // Load back as bytes32
        result := mload(0x00)
    }
    return result; // Will return 0x414243000... (ABC followed by zeros)
}
```

## 3. Practical Use Case: Bytes Manipulation
```solidity
function concatBytes(bytes memory a, bytes memory b) public pure returns (bytes memory) {
    bytes memory result;
    
    assembly {
        // Get length of a
        let aLen := mload(a)
        // Get length of b
        let bLen := mload(b)
        
        // Calculate total length
        let totalLen := add(aLen, bLen)
        
        // Allocate memory for result (length + data)
        result := mload(0x40)
        mstore(result, totalLen)
        
        // Copy a's data
        let dest := add(result, 0x20)
        let src := add(a, 0x20)
        for { let i := 0 } lt(i, aLen) { i := add(i, 0x20) } {
            mstore(add(dest, i), mload(add(src, i))
        }
        
        // Copy b's data
        dest := add(dest, aLen)
        src := add(b, 0x20)
        for { let i := 0 } lt(i, bLen) { i := add(i, 0x20) } {
            mstore(add(dest, i), mload(add(src, i))
        }
        
        // Update free memory pointer
        mstore(0x40, add(result, add(0x20, totalLen)))
    }
    
    return result;
}
```

## Key Points:
- `MLOAD(addr)`: Loads 32 bytes from memory address `addr`
- `MSTORE(addr, value)`: Stores 32-byte `value` at memory address `addr`
- `MSTORE8(addr, value)`: Stores 1 byte `value` at memory address `addr`
- Memory is linear and can be addressed at byte level
- Solidity reserves 0x00-0x3f for scratch space and special purposes
- 0x40 stores the current free memory pointer

These low-level operations are typically used for:
- Efficient bytes manipulation
- Custom data serialization
- Memory-optimized algorithms
- Gas optimization in critical functions
  
