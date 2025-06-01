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
  
# Solidity Functions That Use `MLOAD`, `MSTORE`, and `MSTORE8` Under the Hood

Many common Solidity operations compile down to these low-level memory opcodes. Here are practical examples showing high-level Solidity code and what happens at the EVM level:

## 1. Basic Value Assignment (Uses `MSTORE`/`MLOAD`)
```solidity
function valueAssignment() public pure returns (uint256) {
    uint256 x = 42; // Compiles to MSTORE operation
    
    uint256 y = x;  // Compiles to:
                    // 1. MLOAD to get x's value
                    // 2. MSTORE to store in y's slot
    
    return y;       // MLOAD to get y's value
}
```

## 2. Bytes/String Manipulation (Uses Memory Operations)
```solidity
function stringOperations() public pure returns (bytes memory) {
    string memory hello = "Hello"; // MSTORE for each character
    
    bytes memory world = "World";  // Uses MSTORE8 for individual bytes
    
    bytes memory combined = abi.encodePacked(hello, world);
    // Under the hood:
    // 1. MLOAD to get lengths
    // 2. MSTORE to write new length
    // 3. Series of MSTORE8 to copy bytes
    
    return combined;
}
```

## 3. Array Operations
```solidity
function arrayOps() public pure returns (uint256) {
    uint256[] memory arr = new uint256[](3); // MSTORE for length
    
    arr[0] = 1; // MSTORE at calculated offset
    arr[1] = 2; // MSTORE at calculated offset
    arr[2] = arr[0] + arr[1]; // MLOAD both, then MSTORE result
    
    return arr.length; // MLOAD of array's length slot
}
```

## 4. Struct Operations
```solidity
function structOps() public pure returns (uint256) {
    struct Point {
        uint256 x;
        uint256 y;
    }
    
    Point memory p = Point(10, 20); // Two MSTORE operations
    
    p.x = p.y * 2; // MLOAD p.y, then MSTORE p.x
    
    return p.x; // MLOAD p.x
}
```

## 5. ABI Encoding (Heavy Memory Usage)
```solidity
function abiEncoding(address recipient, uint256 amount) public pure returns (bytes memory) {
    // Under the hood uses multiple MLOAD/MSTORE operations:
    return abi.encodeWithSignature("transfer(address,uint256)", recipient, amount);
    
    // Equivalent to manual encoding:
    // bytes memory data = new bytes(4 + 32 + 32);
    // mstore(data, 0xa9059cbb) // function selector
    // mstore(add(data, 0x04), recipient)
    // mstore(add(data, 0x24), amount)
}
```

## Key Observations:
1. **Memory variables** (declared with `memory` keyword) always use these opcodes
2. **Value assignments** between memory variables use `MLOAD`/`MSTORE`
3. **Array/struct operations** compile to sequential memory accesses
4. **String/bytes operations** often use `MSTORE8` for individual bytes
5. **ABI encoding** functions are essentially memory manipulation wrappers

The Solidity compiler automatically generates these low-level operations whenever you work with memory variables, making memory management transparent to developers in most cases.
