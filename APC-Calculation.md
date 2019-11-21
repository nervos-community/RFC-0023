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

> Question to Solve: from block `i`, where total issuance up to and including block `i` is `C_i`, what is the prospective compensation rate in the next month / year / years?

We use the data above to calculate compensation rate. It is noticed that, the secondary issuance velocity is always uniform, and the primary issuance velocity may change.

### Assume primary issuance is uniform during compensation calculation period

Let's say secondary issuance in single time interval is `dx`. The ratio of primary issuance to secondary issuance in every block is `a`. There will have been `S_n` secondary issuance since block `i` in the estimation period. And `t` means the treasury portion of secondary issuance. The compensation rate in this period is

![](http://latex.codecogs.com/gif.latex?rate=\\prod^{S_n}\\left(1+\\frac{dx}{C_i+(\\alpha+1)x-t}\\right)-1\\approx\\int_{0}^{S_n}\\frac{dx}{C_i+(\\alpha+1)x-t})

![](http://latex.codecogs.com/gif.latex?rate_{lowbound}=\\int_{0}^{S_n}\\frac{dx}{C_i+(\\alpha+1)x}=\\frac{\\ln{((\\alpha+1)S_n+C_i)}-\\ln{C_i}}{(\\alpha+1)})

> **Notice**: treasury portion `t` is never issued, and it's varying according to Nervos DAO locked portion and CKBytes used portion, which are unpredictable. We simply ignore the `t` part to get the lower bound of compensation rate.

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
APC = (1+0.0124)*(1+0.0119) - 1 = 2.44%
```

## Simple Calculation Formula
> If I deposit CKBytes at epoch `e`, block `i`, how much compensation will I get after `n` epochs?

**requirement**
- `n` must greater than `180`
- we assume that user send withdraw request at perfect time

**pre-calculation**
- `C_i`, the total issuance up to and including block `i`
- `S_n`, the prospective secondary issuance in future `n` epochs. `S_n = n * s`, where `s` is constant.
- `P_n`, the prospective primary issuance in future `n` epochs. `P_n = n * p`, where `p` is constant in one 4-year-half period, if not, we calculate the result separately.
- `a`, the ratio of primary issuance to secondary issuance in every block, `a = p/s` if `p` is constant.

**formula**

![](http://latex.codecogs.com/gif.latex?rate=\\frac{1}{\\alpha+1}\\ln{\\left(1+\\frac{(\\alpha+1)s}{C_i}\\cdot%20n\\right)})

**useful constants**
- epochs in one natural year: `365*24/4 = 2190`
- `C_i`: data field embeded in block header.
- `s`: secondary issuance per epoch, always equals to `1.344B / 2190`
- `p_1`: primary issuance per epoch in first 4-year-half period, equals to `4.2B / 2190`
- `p_2`: primary issuance per epoch in second 4-year-half period, equals to `2.1B / 2190`
- `a_1`: the ratio of primary issuance to secondary issuance in first 4-year-half period, `4.2/1.344 = 3.125`
- `a_2`: the ratio of primary issuance to secondary issuance in first 4-year-half period, `2.1/1.344 = 1.5625`

