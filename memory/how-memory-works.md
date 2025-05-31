# How EVM Memory Works During External Calls (Under the Hood)

## Memory Basics
- EVM memory is a linear, byte-addressable space that starts empty for each contract call
- It behaves like a temporary scratchpad that's erased between external calls
- Expands automatically when accessed (costs gas for expansion)

## What Happens During an External Call (`CALL`, `STATICCALL`, `DELEGATECALL`)

1. **Argument Preparation**:
   - The calling contract writes call data (function selector + arguments) to memory
   - Memory locations are specified for where to find:
     * Recipient address
     * Ether value to send
     * Call data start position and length

2. **Memory Isolation**:
   - When execution jumps to the called contract:
     * A completely fresh memory space is created (all zeros)
     * The called contract's memory is isolated from the caller's
     * The called contract gets its own memory "sandbox"

3. **During Execution**:
   - The called contract uses its own clean memory space
   - Any memory writes don't affect the caller's memory
   - Memory expansion costs gas based on highest accessed offset

4. **After Call Completion**:
   - The called contract's memory is discarded
   - Execution returns to caller with:
     * Success/failure status
     * Return data (if any) copied to caller's memory
   - Caller's original memory remains intact (except where return data is written)

## Key Implications
- Memory can't be used for persistent storage between calls
- Each external call gets a fresh memory space (security feature)
- Call data must be explicitly copied between memory spaces if needed
- Memory is cheaper than storage but must be re-initialized after calls
