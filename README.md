# clone-lin-geth-call2929

LIN clone of [ethereum/go-ethereum](https://github.com/ethereum/go-ethereum) tag `v1.16.9` `gasCallEIP2929` (Berlin CALL: cold-warm 2500 charged before EIP-150 63/64, LGPL-3.0).

This repository holds **only the LIN copy**. Auditable proofs live in [kbelludoo/lin-open](https://github.com/kbelludoo/lin-open):

- Kernel: `src/lin_geth_call2929.lin`
- Proofs: `examples/geth_call2929/`
- Event: `docs/events/EVENT_GETH_CALL2929_CLONE_LIN.rulel`
- Harness: `python3 test/prove_geth_call2929_external.py`

## Class

**EXPERIMENTAL** i64-scale. Not a Geth node. Geth `makeCallVariantGasCallEIP2929` reads `StateDB` + stack, `UseGas(2500)` if cold, then calls `gasCall` (new-account 25000 + value 9000 + Cmem + EIP-150 63/64) and adds 2500 back. LIN lifts that to integers `available`, `requested`, `memory_gas`, `transfers`, `new_acct`, `is_cold`. Quadratic `memoryGasCost` (Cmem) and `CallStipend` 2300 are out of scope. Jump-table `constantGas` (`WarmStorageReadCostEIP2929=100`) is not returned by the dynamic function.

## Goldens

When the 63/64 cap binds, Berlin parent delta is **not** a flat 2500: the child receives less gas because cold is charged first.

| available | requested | extras | warm dyn | cold dyn | delta | warm child | cold child |
|---|---:|---|---:|---:|---:|---:|---:|
| 64000 | 1e6 | none | 63000 | 63040 | **40** | 63000 | 60540 |
| 64000 | 1000 | none | 1000 | 3500 | **2500** | 1000 | 1000 |
| 100000 | 0 | value 9000 | 9000 | 11500 | 2500 | 0 | 0 |
| 100000 | 1e6 | value+new 34000 | 98969 | 99008 | **39** | 64969 | 62508 |

Opcode total adds the 100 warm constant. Negative inputs or flags outside `{0,1}` fail-close to 0. `available=2499` cold fail-closes (cannot `UseGas(2500)`).

## License

Algorithm: LGPL-3.0 (go-ethereum). This LIN transcription is published so auditors can hash the clone independently of lin-open proofs.
