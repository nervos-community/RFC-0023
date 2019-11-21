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

> Question to Solve: from block `i`, where total total issuance up to and including block `i` is `C_i`, what is the prospective compensation rate in the next month / year / years?

We use the data above to calculate compensation rate. It is noticed that, the secondary issuance velocity is always uniform, and the primary issuance velocity may change.

### Assume primary issuance is uniform during compensation calculation period

Let secondary issuance in single time interval is `dx`. The ratio of primary issuance to secondary issuance in every block is `a`. There will have been `S_n` secondary issuance since block `i` in the estimation period. The compensation rate in this period is

![](http://latex.codecogs.com/gif.latex?\\prod^{S_n}\\left(1+\\frac{dx}{C_i+(\\alpha+1)x}\\right)-1\\approx\\int_{0}^{S_n}\\frac{dx}{C_i+(\\alpha+1)x}=\\frac{\\ln{((\\alpha+1)S_n+C_i)}-\\ln{C_i}}{(\\alpha+1)})

**Example: annual percentage compensation from genesis to the end of first year**
```yml
C_0 = 33.6B
S_n = 1.344B
a = 4.2/1.344 = 3.125
APC = 0.037 = 3.7%
```

**Example: average annual percentage compensation from genesis to half of the first year**
```yml
C_0 = 33.6B
S_n = 1.344B / 2
a = 4.2/1.344 = 3.125
APC = 0.0192 * 2 = 3.84%
```

### Assume primary issuance is not uniform during compensation calculation period

We should separately calculate the compensation rate, and combine them.

**Example: average annual percentage compensation from year 3.5 to year 4.5**
```yml
// first half year:
C_3_5 = 33.6 + 4.2*3.5 + 1.344*3.5 = 53.004
S_n = 1.344 / 2 = 0.672
a = 4.2/1.344 = 3.125
r_1 = 0.0124
// sencond half year:
C_4 = 33.6 + 4.2*4 + 1.344*4 = 55.776
S_n = 1.344 / 2 = 0.672
a = 2.1/1.344 = 3.125 = 1.5625
r_2 = 0.0119
// combine
APC = (1+0.0124)*(1+0.0119) - 1 = 2.4%
```
