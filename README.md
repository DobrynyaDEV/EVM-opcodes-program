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

Calldata is a sequence of 32-byte words (big-endian, two's complement for negatives):

```
[ N ][ x1 ][ x2 ] ... [ xN ]
 0    32    64        N*32     ← byte offsets
```

## Output

One 32-byte word: `sum / N` as a signed integer (`SDIV`, rounds toward zero).
On invalid input the program executes `REVERT` with no data.

## How it works

| Step | Opcodes | Notes |
|---|---|---|
| Read N | `PUSH0 CALLDATALOAD` | N is the first word |
| Check `N > 0` | `SGT ISZERO JUMPI` | signed comparison, so negative N is rejected |
| Check word count | `CALLDATASIZE SHR LT ISZERO JUMPI` | `size >> 5` gives the word count, and the check is `N < words`. Dividing instead of multiplying avoids overflow |
| Sum | loop with `JUMPDEST` / `JUMPI` / `JUMP` | stack layout `[i, sum, N]`, `i` counts down from N to 1 |
| Average | `SDIV` | signed division |
| Clear stack | `POP`, `SDIV`, `MSTORE` consume every value | stack is empty before `RETURN` |
| Return | `MSTORE` then `RETURN(0, 32)` | |

## Test if it works
[project on evm.codes](https://www.evm.codes/playground?fork=osaka&unit=Wei&callData=0x0000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000a0000000000000000000000000000000000000000000000000000000000000014&codeType=Mnemonic&code=%27vk%D1%82%D0%B0%D0%B5%D0%BC%20NsXyN%3E0sm2aGTFy%D0%BDhNg%D0%BBjoSIZE4Rm2z19~EQFf%D1%86%D0%B8%D0%BA%D0%BBsm2~dm1p2eqI~m14LXl1l29l1z1l1aUBY18qc%D0%BE%D0%B4%D1%81%D1%87%D0%B5%D1%82g%D1%80%D0%B5%D0%B4%D0%BD%D0%B5%D0%B3%D0%BEKaDIVbkiwi%D0%B5w%D0%B8%20returns~MSTOREY206TURNb%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82w%D0%BE%D1%88%D0%B8%D0%B1%D0%BA%D0%B8Ks6VERT%27~%5Cnz_1%20yc%D1%80j%D0%B5%D1%80ww%D0%BAhv%2F%2F%20s_0q~JUMPp~ISZEROYo~CALLDATAm~DUPlaWAPk%D1%87%D0%B8j%D0%BE%D0%B2i%D1%81%D1%82h%D0%B0%20g%20%D1%81f~~vdqDESTcf%D0%BFbf%D0%BEa~S_~PUSHYz0xXoLOADKd~POPFp37qI9~ADD6s~RE4z5aH%01469FKXY_abcdfghijklmopqsvwyz~_)

## License

MIT
