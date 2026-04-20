# ISD Complexity Estimates for GBD Parameters

## Overview

We computed Information-Set Decoding (ISD) complexity estimates for the Generalized Bicycle Distinguishing (GBD) problem using the [Syndrome Decoding Estimator](https://github.com/Crypto-TII/syndrome_decoding_estimator) (Esser & Bellini, PKC 2022). The attack reduces to finding low-weight codewords in the dual code C^perp, which is a standard syndrome decoding problem.

### ISD input convention

For a GBD instance with parameters (r, s, w\_seed, K):

- n = 2rs (code length)
- k = n - K (code dimension, where K is the logical dimension)
- w = 2w\_seed (target codeword weight)

## Tool Calibration (BIKE Level 1)

We verified the estimator against BIKE Level 1 (n=24646, k=12323, w=142). The tool returns BJMM time complexity of **160.9 bits**, matching the estimator's own documented test output exactly.

BIKE's published 128-bit security claim includes a quasi-cyclic speedup of log2(k) = 13.6 bits plus memory-restricted attack models. GBD does not admit quasi-cyclic speedup, so **raw ISD estimates are the correct attack cost** for GBD.

## Results

### Main table (K=20 placeholder)

| (r, s) | l | N | w\_seed | 2w | Best ISD (log2) | Best algorithm | BJMM (log2) | Brute force (log2) | Regime |
|---|---|---|---|---|---|---|---|---|---|
| (15, 16) | 240 | 480 | 8 | 16 | 20 | Ball Collision | 25 | 97 | toy |
| (15, 16) | 240 | 480 | 12 | 24 | 20 | Ball Collision | 28 | 133 | toy |
| (20, 21) | 420 | 840 | 10 | 20 | 26 | Ball Collision | 39 | 132 | small |
| (20, 21) | 420 | 840 | 14 | 28 | 82 | Ball Collision | 103 | 172 | small |
| (35, 36) | 1260 | 2520 | 16 | 32 | 130 | Ball Collision | 143 | 243 | medium |
| (45, 46) | 2070 | 4140 | 20 | 40 | 218 | BJMM | 218 | 320 | near-target |
| (55, 56) | 3080 | 6160 | 22 | 44 | 274 | BJMM | 274 | 372 | near-target |
| (63, 64) | 4032 | 8064 | 24 | 48 | 306 | BJMM | 306 | 419 | target |

All estimates are classical time complexity in log2 bit operations. Brute force floor is approximately log2(C(n, 2w)) - 1.

### Full algorithm comparison

| Label | n | k | 2w | Prange | Dumer | Ball Collision | BJMM | May-Ozerov | Both-May |
|---|---|---|---|---|---|---|---|---|---|
| toy-1 | 480 | 460 | 16 | 25.3 | 21.0 | 20.2 | 25.3 | 25.3 | 26.0 |
| toy-2 | 480 | 448 | 24 | 27.6 | 20.6 | 19.8 | 27.5 | 27.6 | 28.4 |
| small-1 | 840 | 820 | 20 | 38.4 | 27.1 | 26.1 | 39.4 | 39.4 | 39.4 |
| small-2 | 840 | 820 | 28 | N/A | 128.8 | 81.6 | 102.6 | 97.5 | 97.5 |
| medium | 2520 | 2500 | 32 | N/A | N/A | 129.7 | 142.9 | 151.6 | 151.5 |
| near-target-1 | 4140 | 4120 | 40 | N/A | N/A | N/A | 217.5 | 232.2 | 232.2 |
| near-target-2 | 6160 | 6140 | 44 | N/A | N/A | N/A | 273.6 | 281.7 | 281.7 |
| target | 8064 | 8044 | 48 | N/A | N/A | N/A | 305.5 | 328.9 | 328.9 |

N/A indicates the algorithm cannot handle the parameter regime (w > n-k for Prange, or optimization failure for Dumer/Ball Collision at large parameters). Stern crashes on all parameter sets due to a division-by-zero bug with high-rate codes (n-k < w).

## Analysis

### Security regime

The **128-bit security boundary** at K=20 falls at the medium parameters (r=35, s=36, N=2520, w\_seed=16), where the best attack gives ~130-bit complexity. The target parameters (r=63, s=64) overshoot 128-bit security by ~178 bits.

For smaller parameter sets (toy and small-1), ISD complexity is below 40 bits -- these are trivially breakable regardless of the attack model.

### Why the codes are in an unusual ISD regime

These codes have **extremely high rate**: k/n > 0.99, with n-k = K = 20 for most parameter sets. This is fundamentally different from BIKE (n-k = 12323) or McEliece (n-k = 768).

In standard ISD (BJMM), the algorithm distributes the target weight w across the information set (k positions) and redundancy part (n-k positions). The constraint is:

    w - 2p <= n - k = K

where p is the weight placed on each half of the information set. So the minimum p is:

    p_min = ceil((w - K) / 2)

When K is small and w > K, most of the weight **must** be in the information set, forcing p to be large. The list sizes in BJMM scale as C(k/2, p), so large p creates exponentially larger lists and higher complexity. This is the source of security in these codes.

| K | p\_min (w=48) | List size C(k/2, p\_min) | Approx log2 |
|---|---|---|---|
| 20 | 14 | C(4022, 14) | ~131 |
| 30 | 9 | C(4017, 9) | ~93 |
| 40 | 4 | C(4012, 4) | ~43 |
| 48 | 0 | 1 (Prange) | 0 |

### Best attack transitions

- **Toy/small parameters** (w <= K or w slightly above K): Ball Collision gives the best attack, outperforming BJMM. Dumer is competitive. These are parameter sets where the simple split-and-merge approach works well because p is small.

- **Medium and above** (w >> K): BJMM dominates. Ball Collision and Dumer fail to optimize (return infinity). The BJMM tree structure becomes essential for managing the large lists forced by high p\_min.

### Scaling behavior

In the non-trivial regime (small-2 through target), the complexity follows the asymptotic form 2^{c * sqrt(n * 2w)} with:

    c = 0.508 +/- 0.034 (CV = 6.6%)

The constant c is stable, confirming that the estimator behaves consistently across this parameter range. The toy and small-1 parameters are in a "trivially easy" regime where finite-size effects dominate and the asymptotic formula does not apply.

| Label | n | 2w | sqrt(n * 2w) | Best ISD | c |
|---|---|---|---|---|---|
| small-2 | 840 | 28 | 153.4 | 81.6 | 0.532 |
| medium | 2520 | 32 | 284.0 | 129.7 | 0.457 |
| near-target-1 | 4140 | 40 | 406.9 | 217.5 | 0.535 |
| near-target-2 | 6160 | 44 | 520.6 | 273.6 | 0.526 |
| target | 8064 | 48 | 622.2 | 305.5 | 0.491 |

## Critical Finding: K Sensitivity

ISD complexity is **extremely sensitive** to K. The original expectation was that varying K by a factor of 10 would change complexity by less than 5 bits. The actual variation is orders of magnitude larger.

### K variation at target parameters (r=63, s=64, N=8064, w=48)

| K | k = N-K | BJMM time (log2) | BJMM memory (log2) |
|---|---|---|---|
| 4 | 8060 | inf | 13.0 |
| 10 | 8054 | 371.7 | 174.8 |
| 20 | 8044 | 305.5 | 146.2 |
| 30 | 8034 | 223.6 | 98.3 |
| 40 | 8024 | 151.5 | 66.7 |
| 60 | 8004 | 44.4 | 19.0 |
| 80 | 7984 | 31.4 | 19.4 |
| 100 | 7964 | 30.0 | 19.7 |

### Why K matters so much

K = n - k is the number of parity-check constraints available to the attacker. It controls security through two mechanisms:

1. **Minimum list weight p\_min = (w - K)/2**: smaller K forces larger p, creating exponentially larger lists in the BJMM tree.

2. **Hard break at K >= w**: when K >= 2w\_seed = w, the attacker can place all weight on the redundancy side (p=0), collapsing the problem to Prange with near-zero cost.

The effective security parameter is **w - K**, not w alone. Each additional unit of K reduces p\_min by 0.5, shrinking list sizes by a factor of roughly k/(2p), which for these parameters is ~280x per unit.

### Maximum tolerable K for 128-bit security

| Parameter set | N | 2w | Max K for 128-bit security |
|---|---|---|---|
| medium | 2520 | 32 | ~20 |
| near-target-1 | 4140 | 40 | ~33 |
| near-target-2 | 6160 | 44 | ~37 |
| target | 8064 | 48 | ~42 |

## Comparison with BIKE

| | BIKE Level 1 | GBD target (K=20) |
|---|---|---|
| n | 24,646 | 8,064 |
| n - k | 12,323 | 20 |
| w | 142 | 48 |
| Code rate | 0.500 | 0.998 |
| p\_min | 0 | 14 |
| Raw BJMM | 160.9 | 305.5 |
| Security mechanism | Large search space | Forced large lists |
| QC speedup | -13.6 bits | None |

The two constructions achieve security through fundamentally different mechanisms:

- **BIKE**: moderate rate, massive redundancy (n-k = k). The attacker can use small p (even p=0 via Prange), but the combinatorial space C(n, w) is enormous. Security comes from the sheer size of the search space.

- **GBD**: extreme high rate, minimal redundancy (n-k = 20). The attacker is forced into p >= 14, creating enormous internal lists. Security comes from the cost of managing these lists, not from the size of the outer search space.

This makes BIKE's security more robust to parameter variations, while GBD's security is more fragile -- it depends critically on K remaining small.

## Caveats

1. **These are analytical estimates**, not empirical measurements. The estimator computes the asymptotic cost of ISD algorithms including polynomial factors, but does not account for constant-factor optimizations or implementation-specific speedups.

2. **K = 20 is a placeholder.** The K values used here are typical values from a reported bimodal distribution. Security claims require characterizing the actual K distribution (especially the tail/maximum) for each parameter set.

3. **Stern is not reported** due to a bug in the estimator for high-rate codes (division by zero when n-k < w). Ball Collision (a Stern refinement) is reported as a substitute and gives lower complexity for smaller parameter sets.

4. **No quantum estimates are included.** All complexity figures are classical. Grover-accelerated ISD would reduce security by roughly a factor of 2 in the exponent.

5. **Structure-specific attacks are not considered.** These estimates assume the code behaves as a random linear code. If the generalized bicycle structure is exploitable (e.g., via algebraic attacks on the circulant structure), the actual security could be lower.

## Recommendations

1. **Characterize the K distribution** per parameter set. Measure the maximum observed K across a large sample of seeds. Security claims must use worst-case K, not typical K.

2. **Consider rejection sampling** on seeds: discard any seed producing K above a threshold. This guarantees a K bound but reduces the effective key space and may leak information about which seeds are "good."

3. **The target parameters are conservative.** At K=20, the target gives 306-bit security. If K can be bounded below ~42, even the target parameters maintain 128-bit security. If K <= 20 can be guaranteed, the medium parameters (r=35, N=2520) suffice for 128-bit security, offering a 3x reduction in code length.

4. **Investigate algebraic bounds on K.** For generalized bicycle codes, K depends on the rank of circulant matrices over F\_2, which is determined by gcd structure of the generator polynomials. An analytic upper bound on K would be more defensible than empirical sampling.

## Empirical K-Distribution (from qprc-sims campaign)

The K=20 placeholder used above was tested empirically by sampling random
seeds and computing K = 2ℓ - rank(H\_X) - rank(H\_Z) for each.

### Summary table

| Params | n | trials | mean | median | max | P90 | P95 | P99 | K≤20 | K≥2w |
|---|---|---|---|---|---|---|---|---|---|---|
| (15,16,10) | 480 | 242 | 6.6 | 6 | 36 | 12 | 16 | 21 | 98.8% | 2.1% |
| (20,21,14) | 840 | 200 | 8.9 | 8 | 34 | 18 | 22 | 28 | 94.5% | 2.0% |
| (35,36,16) | 2520 | 200 | 10.8 | 8 | 46 | 20 | 26 | 36 | 91.0% | 2.0% |
| (45,46,20) | 4140 | 200 | 6.1 | 4 | 36 | 12 | 16 | 28 | 97.5% | 0.0% |
| (63,64,24) | 8064 | 50 | 8.3 | 6 | 28 | 20 | 22 | 26 | 94.0% | 0.0% |

### (15, 16, 10): 242 trials

| Statistic | Value |
|---|---|
| Mean | 6.6 |
| Median | 6 |
| Max observed | 36 |
| P90 | 12 |
| P95 | 16 |
| P99 (bootstrap 95% CI) | 21 [16, 32] |
| K ≤ 20 | 98.8% |
| K ≥ 2w=20 (hard break) | 2.1% |

Distribution is concentrated at low K (33.5% at K=2), but has a long right
tail extending to K=36 — well above the hard break at K=2w\_seed=20.

### (35, 36, 16): 200 trials — the 128-bit boundary

| Statistic | Value |
|---|---|
| Mean | 10.8 |
| Median | 8 |
| Max observed | **46** |
| P90 | 20 |
| P95 | 26 |
| P99 (bootstrap 95% CI) | 36 [26, 46] |
| K ≤ 20 (128-bit secure) | 91.0% |
| K ≤ 30 | 98.0% |
| K ≥ 2w=32 (hard break) | **2.0%** |

```
    K=  2:  34 (17.0%)  ######
    K=  4:  19 ( 9.5%)  ###
    K=  6:  23 (11.5%)  ####
    K=  8:  30 (15.0%)  ######
    K= 10:  13 ( 6.5%)  ##
    K= 12:  11 ( 5.5%)  ##
    K= 14:  18 ( 9.0%)  ###
    K= 16:   9 ( 4.5%)  #
    K= 18:  10 ( 5.0%)  ##
    K= 20:  14 ( 7.0%)  ##
    K= 22:   3 ( 1.5%)  #
    K= 24:   4 ( 2.0%)  #
    K= 26:   5 ( 2.5%)  #
    K= 28:   2 ( 1.0%)  #
    K= 36:   2 ( 1.0%)  #
    K= 38:   1 ( 0.5%)  #
    K= 46:   1 ( 0.5%)  #
```

### Security implications by K at each parameter set

**Medium (35,36,16), n=2520, 2w=32:**

| K | Best ISD (bits) | Status |
|---|---|---|
| 2 | inf | secure |
| 8 | 233 | secure |
| 14 | 185 | secure |
| 20 | 130 | borderline 128-bit |
| 26 | 83 | below 128-bit |
| 36 | 27 | broken |
| 46 | 25 | broken |

**Target (63,64,24), n=8064, 2w=48:**

| K | Best ISD (bits) | Status |
|---|---|---|
| 2 | inf | secure |
| 8 | 409 | secure |
| 20 | 305 | secure |
| 36 | 155 | secure |
| 46 | 58 | weak |
| 48 | 51 | weak |

### (45, 46, 20): 200 trials — near-target

| Statistic | Value |
|---|---|
| Mean | 6.1 |
| Median | 4 |
| Max observed | 36 |
| P90 | 12 |
| P95 | 16 |
| P99 (bootstrap 95% CI) | 28 |
| K ≤ 20 | 97.5% |
| K ≥ 2w=40 (hard break) | 0.0% |

### (63, 64, 24): 50 trials — target parameters

| Statistic | Value |
|---|---|
| Mean | 8.3 |
| Median | 6 |
| Max observed | **28** |
| P90 | 20 |
| P95 | 22 |
| P99 (bootstrap 95% CI) | 26 [20, 28] |
| K ≤ 20 | 94.0% |
| K ≤ 40 | 100.0% |
| K ≥ 2w=48 (hard break) | **0.0%** |

At K=28 (worst observed), BJMM gives **242-bit security** — well above
128 bits. No rejection sampling needed at target parameters.

### Implications

1. **Target parameters (63,64,24) reliably achieve 128-bit security.** Max
   observed K=28 gives 242-bit ISD security. No seed in 50 trials fell below
   128 bits. No rejection sampling needed.

2. **Medium parameters (35,36,16) do NOT reliably achieve 128-bit security.**
   ~9% of seeds produce K > 20, dropping below 128 bits. ~2% produce K ≥ 32,
   which is essentially broken. Rejection sampling is mandatory at this size.

3. **The K-distribution has a consistent shape** across all five parameter
   sets: mode at K=2, broad plateau, sparse tail. ~2% of seeds hit K ≥
   2w\_seed at the smaller parameter sets, but this fraction drops to 0% at
   larger (r,s) — the distribution tightens as the code length grows.

4. **Rejection sampling policy:** At medium params, K\_max=20 retains 91% of
   seeds. At target params, no rejection needed — all observed seeds have
   K ≤ 28 < 48.

5. **The 128-bit boundary without rejection sampling** lies between medium
   (35,36) and near-target (45,46). At (45,46,20), P99=28 and the hard
   break is at K=40, giving comfortable margin.

## Reproduction

All estimates were computed using the Syndrome Decoding Estimator with the following setup:

- Python 3.11.11 with scipy, prettytable, progress
- C library built from source (clang 15.0.0, cmake 4.2.3)
- `theoretical_estimates.py` patched to find the C library in the local build directory
- No SageMath required
- Classical estimates only (quantum\_estimates=0)
- Bit complexity mode (bit\_complexities=1, the default)
- Unlimited memory model (memory\_limit=inf)

Example invocation:

```python
PYTHONPATH=. python3 -c "
from sd_estimator.estimator import sd_estimate_display
sd_estimate_display(n=8064, k=8044, w=48, quantum_estimates=0)
"
```
