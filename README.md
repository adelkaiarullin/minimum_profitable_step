# Minimum profitable step
In pairwise statistical arbitrage, trades target the spread. Basic strategy: open when spread hits |a|, close when it drops below |b| (|a| > |b|). Advanced approach uses step s > 0—signals at levels k·s. The repo determines the smallest profitable s.


## Description of the minimum step estimation algorithm

Fasten your seatbelts, this won’t be the safest reading.

### Derivation of the minimum step formula
The income from a trade on an arbitrage pair is generated when opening a position at the level (a long position on the left leg and a short position on the right leg, for example) $`spread_{t}`$ and closing at the level $`spread_{t+1} = spread_{t} - step`$. The yield is calculated using the formula

$$
\mathrm{trade}_{PnL} = \Delta P_{a} \cdot c_{a} - N_{hedge} \cdot \Delta P_{b} \cdot c_{b} \quad (1)
$$

, where $`\Delta P_{a} \cdot c_{a}`$ is the change in points per trade multiplied by the value of a point in dollars. Similarly for the second leg, but taking into account the hedging coefficient $`N_{hedge}`$.
We need $`\mathrm{trade}_{PnL} > 2 \cdot \text{commission} \cdot (1 + N_{hedge})`$, and based on this constraint, we will build all subsequent reasoning. The hedging coefficient is calculated as follows: $`N_{hedge} = \beta \cdot \frac{P_{a} \cdot c_{a}}{P_{b} \cdot c_{b}}`$, then formula (1) can be transformed into the following form.

$$
(R_{a} - 1) \cdot P_{a} \cdot c_{a} - \beta \cdot \frac{P_{a} \cdot c_{a}}{P_{b} \cdot c_{b}} \cdot (R_{b} - 1) \cdot P_{b} \cdot c_{b} > 2 \cdot \text{commission} \cdot (1 + N_{hedge}) \quad (2)
$$

where $`R_{a}`$ and $`R_{b}`$ are the returns on the trade ($`P_{\text{close-trade}} / P_{\text{open-trade}}`$). Simplify the formula by removing the unnecessary parts

$$
R_{a} + \beta \cdot R_{b} > 1 - \beta + \frac{2 \cdot \text{commission} \cdot (1 + N_{hedge})}{P_{a} \cdot c_{a}} \quad (3)
$$

In formula (3), $`R_{a}`$ and $`R_{b}`$ remain unknown, but the dimensionality of the problem can be reduced based on the assumption that the change in the spread $`\Delta \text{spread} = \ln R_{a} - \beta \cdot \ln R_{b}`$ can be expanded into a Taylor series, and only the first terms of the series can be taken, i.e., $`\ln R_{a} - \beta \cdot \ln R_{b} \approx (R_{a} - 1) - \beta \cdot (R_{b} - 1)`$, and inequality (3) can be rewritten as follows:

$$
\ln R_{a} - \beta \cdot \ln R_{b} = \Delta \text{spread} = \frac{2 \cdot k \cdot \text{commission} \cdot (1 + N_{hedge})}{P_{a} \cdot c_{a}} \quad (4)
$$

Equation (4) is written as the final equation for finding the minimum profitable step $`\Delta \text{spread} = step`$, the term $`\frac{2 \cdot k \cdot \text{commission} \cdot (1 + N_{hedge})}{P_{a} \cdot c_{a}}`$ is multiplied by k to account for slippage and other execution risks.

## Code for optimization and finding the minimum step

### Calculation code

```python
def min_spread_step_approx(
    beta: float,
    price_a: float,
    price_b: float,
    cs_a: float,
    cs_b: float,
    commission: float = 4,
    k: int = 20,
) -> float:
    """Finding the minimum grid spacing"""
    base_right_order = beta * (price_a * cs_a) / (price_b * cs_b)
    return k * 2 * commission * (1 + base_right_order) / (price_a * cs_a)
```

### Example

```python
pure_step = min_spread_step_approx(
    0.36577227009573265,
    0.65,
    0.0065,
    100000.0,
    12500000.0,
    4,
)
...
step / ou_std # 0.15883579971723014
```
