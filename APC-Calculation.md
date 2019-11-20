# Compensation Rate Quick Calculation

reference: [NERVOS-DAO-RFC](https://github.com/nervosnetwork/rfcs/blob/2aa14e142397570778f300468de2bb427e485507/rfcs/0000-dao-deposit-withdraw/0000-dao-deposit-withdraw.md)

## DAO Compensation Data

### Theoretical Data

- Annual secondary issuance is constantly 1.344B CKBytes
- Annual primary issuance is half at every 4 years, first 4 years issuance is 33.6B/2 in total, 4.2B CKBytes per year
- Genesis issuance is 33.6B CKBytes

### Runtime Data

CKB’s block header has a special field named dao containing auxiliary information for Nervos DAO’s use. Specifically, the following data are packed in a 32-byte dao field in the following order:

- `C_i` : the total issuance up to and including block `i`.
- `AR_i`: the current `accumulated rate` at block `i`. `AR_j / AR_i` reflects the CKByte amount if one deposit 1 CKB to Nervos DAO at block `i`, and withdraw at block `j`.
- `S_i`: the total secondary issuance up to and including block `i`.
- `U_i` : the total `occupied capacities` currently in the blockchain up to and including block `i`. Occupied capacity is the sum of capacities used to store all cells.

## Compensation Rate

We use the data above to calculate compensation rate.

### APC (Annual Percentage Compensation) at Specific Block

Let's say `C_i` is the the total issuance up to and including block `i`. `P_i` is the total primary issuance in future 365 days since block `i`. The APC of deposit begin at this block should be:

```python
APC_y = 1.344B / (C_i + P_i) * 100%
```

### APC in Period Calculation

If user only deposit for one month or limited period like `D` days. In future `D` days, there will be `C_d` primary and secondary issuance. The APC equation is slightly different from that above.

```python
APC_d = 1.344B / (C_i + C_d) * (365 / D) * 100% 
```

The APC_d is a linear approximate value, APC_d should be greater than `APC_y` if `D` < 365, and should be less than `APC_y` if `D` > 365.
