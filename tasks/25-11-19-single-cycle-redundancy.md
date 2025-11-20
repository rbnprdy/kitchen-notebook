# Single-cycle Redundancy

Functionality to filter two-cycle UDFMs based on either of their cycle being redundant.

Write out redundant faults from single-cycle PEPR run. Map UDFM names to constraints.

Maybe encode constraints into UDFM name? E.g.,

```python
signal_hash = hash('-'.join(sorted(signals)))
value_hash = hash('-'.join(values[signal] for signal in sorted[signals]))
udfm_name = f'pepr-{signal_hash}-{value_hash}'
# can add and hash second value too for 2T
```

When filtering in the two-cycle run, can store them as `dict[signal_hash, set[value_hash]]`, so we just do the dict lookup once per region, and then query the much smaller `set[value_hash]` for each condition.
