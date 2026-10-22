# EVM Average program with pure opcodes

An EVM program written directly in **mnemonic opcodes** that computes the average of N integers in calldata.

Built on the [evm.codes playground](https://www.evm.codes/playground)

## Task

1. Read from calldata an `int N` and `N` words (signed integers)
2. Validate input: `N > 0` and all `N` words are present
3. Compute the average
4. Clear the stack
5. Return the result

## Input format

Calldata is a sequence of 32-byte words:

```
[ N ][ x1 ][ x2 ] ... [ xN ]
 0    32    64        N*32     ← byte offsets
```

## Output

One 32-byte word: `sum / N` as a signed integer.
On invalid input the program executes `REVERT` with no data.

## How it works

The program keeps all values on the stack.

### 1. Read N

```asm
PUSH0 CALLDATALOAD            // [N]
```

offset = 0.

### 2. Validate input

**N > 0**:

```asm
PUSH0 DUP2
SGT ISZERO
PUSH1 0x37 JUMPI
```

**Exactly N words follow N**:

```asm
CALLDATASIZE PUSH1 5 SHR
DUP2 PUSH1 1 ADD
EQ ISZERO
PUSH1 0x37 JUMPI
```

Each check inverts a flag with `ISZERO`, because `JUMPI` only jumps when its condition is non-zero.

### 3. Sum and average

The loop keeps `[i, sum, N]` on the stack, with the counter `i` going from N down to 1, so the exit condition is just `i == 0` (easier than incrementing `i`):

```asm
PUSH0 DUP2

JUMPDEST
DUP1 ISZERO PUSH1 0x2e JUMPI
DUP1 PUSH1 5 SHL CALLDATALOAD
SWAP1 SWAP2 ADD SWAP1
PUSH1 1 SWAP1 SUB
PUSH1 0x18 JUMP

JUMPDEST POP
SDIV
```

### 4. Clear the stack and Return

```asm
JUMPDEST
POP SDIV
PUSH0 MSTORE
PUSH1 0x20 PUSH0
RETURN
```

The result goes: stack → memory → output.

### Error processing

```asm
JUMPDEST POP
PUSH0 PUSH0 REVERT
```

### Jump targets

| Label  | Byte offset |
|--------|-------------|
| `loop` | `0x18` (24) |
| `end`  | `0x2e` (46) |
| `fail` | `0x37` (55) |

## Test it yourself

[project on evm.codes](https://www.evm.codes/playground?fork=osaka&unit=Wei&callData=0x0000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000a0000000000000000000000000000000000000000000000000000000000000014&codeType=Mnemonic&code=%27vk%D1%82%D0%B0%D0%B5%D0%BC%20NsXyN%3E0sm2aGTFy%D0%BDhNg%D0%BBjoSIZE4Rm2z19~EQFf%D1%86%D0%B8%D0%BA%D0%BBsm2~dm1p2eqI~m14LXl1l29l1z1l1aUBY18qc%D0%BE%D0%B4%D1%81%D1%87%D0%B5%D1%82g%D1%80%D0%B5%D0%B4%D0%BD%D0%B5%D0%B3%D0%BEKaDIVbkiwi%D0%B5w%D0%B8%20returns~MSTOREY206TURNb%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82w%D0%BE%D1%88%D0%B8%D0%B1%D0%BA%D0%B8Ks6VERT%27~%5Cnz_1%20yc%D1%80j%D0%B5%D1%80ww%D0%BAhv%2F%2F%20s_0q~JUMPp~ISZEROYo~CALLDATAm~DUPlaWAPk%D1%87%D0%B8j%D0%BE%D0%B2i%D1%81%D1%82h%D0%B0%20g%20%D1%81f~~vdqDESTcf%D0%BFbf%D0%BEa~S_~PUSHYz0xXoLOADKd~POPFp37qI9~ADD6s~RE4z5aH%01469FKXY_abcdfghijklmopqsvwyz~_)

## License

MIT
