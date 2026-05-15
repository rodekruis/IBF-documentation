
# Decision for `all_touched = false` for Population Zonal Statistics

## Summary

`zonal_stats` from the Python library `rasterstats` should use `all_touched = false`. This document shows the reasoning and comparisons.

## Background

We calculate population per administrative area using raster data (e.g., population GeoTIFFs or data PNGs) in both the pipeline and data management code. The `all_touched` parameter in raster zonal statistics determines how pixels are counted.

- **all_touched = true**: Any pixel touched by a polygon is counted (can double-count pixels on boundaries).
- **all_touched = false**: Only pixels whose center is inside the polygon are counted (may slightly undercount).

[ExactExtract](https://github.com/rodekruis/exposure-vulnerability-retrieval) is also included in the comparison. It is expected to be more accurate, but it requires a GeoTIFF input and specific polygon data formatting. That adds significant transformation effort and increases the risk of introducing bugs.

---

## Comparison Results

### 1. `zonal_stats` with `all_touched` set to true vs. false


| TRUE        | FALSE        | Diff   | Percent    |
|-------------|-------------|--------|------------|
| 4,700,995   | 4,688,285   | 12,710 | 0.27%      |
| 2,185,245   | 2,182,619   | 2,626  | 0.12%      |
| 26,991,843  | 26,981,217  | 10,626 | 0.04%      |
| 2,097,936   | 2,088,609   | 9,327  | 0.45%      |
| 773,141     | 770,695     | 2,446  | 0.32%      |
| 688,356     | 685,397     | 2,959  | 0.43%      |
| 669,703     | 664,310     | 5,393  | 0.81%      |
| 55,519,533  | 55,442,410  | 77,123 | 0.14%      |
| 6,483,010   | 6,466,582   | 16,428 | 0.25%      |
| 21,526,422  | 21,505,783  | 20,639 | 0.10%      |
| 7,279,923   | 7,272,857   | 7,066  | 0.10%      |
| 7,975,277   | 7,967,602   | 7,675  | 0.10%      |

**Observation:**
The difference is up to nearly 1%, which shows the impact that double-counting boundary pixels can have.

---

### 2. ExactExtract Comparison

#### `ExactExtract` vs. `zonal_stats` using `all_touched = FALSE`

| ExactExtract | all_touched=FALSE | Diff  | Percent    |
|--------------|------------------|-------|------------|
| 4,685,965    | 4,688,285        | -2,320| -0.05%     |
| 2,182,388    | 2,182,619        | -231  | -0.01%     |
| 26,981,544   | 26,981,217       | 327   | 0.00%      |
| 2,089,162    | 2,088,609        | 553   | 0.03%      |
| 770,764      | 770,695          | 69    | 0.01%      |
| 685,577      | 685,397          | 180   | 0.03%      |
| 664,375      | 664,310          | 65    | 0.01%      |
| 55,445,022   | 55,442,410       | 2,612 | 0.00%      |
| 6,466,264    | 6,466,582        | -318  | 0.00%      |
| 21,504,961   | 21,505,783       | -822  | 0.00%      |
| 7,272,797    | 7,272,857        | -60   | 0.00%      |
| 7,967,055    | 7,967,602        | -547  | 0.01%      |

#### ExactExtract vs. all_touched = TRUE

| ExactExtract | all_touched=TRUE | Diff    | Percent    |
|--------------|------------------|---------|------------|
| 4,685,965    | 4,700,995        | -15,030 | -0.32%     |
| 2,182,388    | 2,185,245        | -2,857  | -0.13%     |
| 26,981,544   | 26,991,843       | -10,299 | -0.04%     |
| 2,089,162    | 2,097,936        | -8,774  | -0.42%     |
| 770,764      | 773,141          | -2,377  | -0.31%     |
| 685,577      | 688,356          | -2,779  | -0.40%     |
| 664,375      | 669,703          | -5,328  | -0.80%     |
| 55,445,022   | 55,519,533       | -74,511 | -0.13%     |
| 6,466,264    | 6,483,010        | -16,746 | -0.26%     |
| 21,504,961   | 21,526,422       | -21,461 | -0.10%     |
| 7,272,797    | 7,279,923        | -7,126  | -0.10%     |
| 7,967,055    | 7,975,277        | -8,222  | -0.10%     |

**Observation:**
- ExactExtract and `zonal_stats` using `all_touched = false` are almost the same.
- Since ExactExtract is more costly to run and requires more complicated data transformations, `zonal_stats` with `all_touched = false` gives us roughly the same results with cleaner, faster code.

---

## References

- [rasterstats zonal_stats documentation](https://pythonhosted.org/rasterstats/manual.html#rasterstats.zonal_stats)
- [ExactExtract GitHub](https://github.com/rodekruis/exposure-vulnerability-retrieval)
