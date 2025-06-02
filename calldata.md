The calldata is where the data of a transaction or the parameters of an external function call reside. It is a read-only data location. You cannot write to it.

At the low level, the EVM opcodes that can be used to read from the calldata are CALLDATALOAD, CALLDATASIZE and CALLDATACOPY.


