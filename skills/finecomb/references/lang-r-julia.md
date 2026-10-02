# A.26 R and Julia

Applies to: R and Julia applications, analytical jobs and native extensions. Shared scientific-computing concerns do not imply identical language semantics.

| Checkpoint | What counts as a problem |
| --- | --- |
| Missing and non-finite data | R `NA` or Julia `missing`, NaN and infinities reach comparisons, filters or aggregates without the intended missing-data policy |
| R vector recycling | Unequal lengths repeat shorter input and silently pair unrelated records; scalar assumptions fail on vectors |
| R indexing and coercion | Partial name matching, dropped dimensions, factor codes or implicit conversion change the data being validated |
| R evaluation environments | Formulas, promises, `eval` or sourced code run in an unintended environment or after required resources close |
| Julia indexing and views | Code assumes every array uses one-based contiguous storage; aliasing through views changes inputs unexpectedly; `@inbounds` removes needed checks |
| Julia arithmetic and dispatch | Fixed-width overflow, conversion or a newly selected method changes numerical results or invariants |
| Tasks and native libraries | Concurrent tasks share mutable arrays or random state; host and BLAS thread pools oversubscribe quotas; native calls violate GC or lifetime rules |
| Reproducibility | Global options, locale, package environments or random state make a resumed job process the same input differently |

Semantic references: [R language definition](https://stat.ethz.ch/R-manual/R-devel/doc/manual/R-lang.html), [Julia differences and semantics](https://docs.julialang.org/en/v1/manual/noteworthy-differences/). Also use [data processing](specialties/4.37-data-processing-batch-jobs-and-machine-learning.md) or [notebooks](specialties/4.74-notebooks-and-interactive-computing.md) when applicable.
