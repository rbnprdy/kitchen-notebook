# Two-cycle UDFMs

[Shawn's google doc](https://docs.google.com/document/d/1hSw3mEaZYD6MQHoaNxHFXXk120M9YVlw1MUdkjHuoi8) describes method for creating two-cycle UDFMs from a set of single-cycle UDFMs. This can be implemented directly in code, flag to turn on nested loop where single-cycle UDFMs are generated.

## Considerations

- Make sure controllability and observability is taken into account properly (filtering on controllability of the first cycle and both on second cycle)

## Brainstorm

There's a `get_io_values` function in [`udfm.py`](/packages/pepr/src/pepr/udfm.py) that seems to try to handle `algorithm`, but looks like it's not being used anywhere.

Flow and script use `UDFMCreator` from [`flow/udfm.py`](packages/pepr/src/pepr/flow/udfm.py), which itself calls the `process_region_batch` funciton, that finally calls `udfm.py:generate_udfms_from_signalset_data_list`. So I think we just need to update that function to handle two-cycle algorithm.

Also, I believe `udfm.py:generate_udfms_from_signalset_data` is used for generating gate-exhaustive UDFMs, so if we want to create those we'll need to update that code too (maybe we can use those for top-off?). I forgot what else Chris was using that for, should check with him.

## Steps

- modify [`udfm.py`](/packages/pepr/src/pepr/udfm.py)`:generate_udfms_from_signalset_data_list` function to handle two-cycle PEPR.
- remove `udfm.py:get_io_values` function

## Implementation scratch

Looks like `generate_udfms_from_signalset_data` generates the faults for a single region. Need to understand this argument:

`signalset_data_list: list[SignalSetData]`

Defined in [`pixel.py`](/packages/pepr/src/pepr/pixel.py). Looks like each `SignalSetData` object is defined for a single component.

SignalSetData looks like this:

```python
class SignalSetData:
    """Holds controlling and observing signal assignments for a component.

    Used in UDFM generation.
    """

    controlling: set[frozenset[NamedSignalAssignment]]
    observing: list[tuple[tuple[SignalName, str], set[NamedSignalAssignment]]]
```

I'm assuming the inner `frozenset[NamedSignalAssignment]` is the frozen set of assignments that result in the same controlling value for those signals. And then the outer set

## Fault/test breakdown

Have to be deliberate about what tests are grouped together into a single fault.

Current (single-cycle) implementation groups together each possible way to control non-target component signals to a specific state into a single fault
(by "non-target component" here I mean the component that is not being actively observed for a given fault).

So, if an intra-cell signal from a non-target component is included in the region, and there are multiple ways to set that intra-cell signal to a specific value (i.e., state), then each possible way will be a different test in the same fault.

However, if there are multiple ways to observe signal(s) from the target component at the same value, these are **separated into different faults**.

Is that what we want? I think so, but I forgot our reasoning.

Further, for two-cycle, what do we want? For non-target components, we now have two states, one for each cycle. I think the straightforward extension of our current single-cycle behavior would be grouping together the product of each possible way to control non-target component signals to state-1 and state-2 as different tests in the same fault. For observation, we would separate the multiple ways to observe state-2 into different faults. But what about the multiple ways to set state-1 for the target component? Would we want those in the same fault or different faults? Answer probably follows from whatever our reasoning was to separate observations in the first place.

### Here's my current question (gonna try zooming someone)

(Going to switch to the language from the paper for consistency:

- **component**: signal inside cell
- **cell**: instance of a standard cell)

For the examples in the ITC25 paper where there’s only **one intra-cell component per cell** included in a region, there’s no fundamental reason why we couldn’t include multiple ways to observe the single intra-cell signal as different tests in the same fault, right? Because in this case there’s only one intra-cell component, so we know the X came from that component. E.g., in Fig 5b, when generating UDFMs, items 2 and 8 could be included as different tests for the same fault? Or is there some other conservatism fundamental to the X-based approach that I’m missing?

Quote from Shawn:

> This fast approach is conservative and has no knoweldge if the X is stemming from all the intra-cell components of interest or some subset of them

Is there a flaw in the following framing:

While rasterizing our design for ATPG, we have a region that contains multiple intra-cell components from the same cell. So our PEPR assumption is that when applying a specific region state, some subset of them could be faulty. Our goal then is to generate test patterns such that we will detect each possible subset being faulty at least once. It’s of course possible that a single pattern could detect multiple subsets.

From the paper:

“For a cell with N internal components, there are 2^N − 1 possible sets that undergo the X-propagation analysis. However, in practice, all possible 2^N − 1 sets do not have to be explicitly considered because larger sets that contain a smaller set where a given cell-input pattern propagates an X do not have to be explicitly analyzed. Why? Because an X value cannot block error propagation, nor force a component to a logic 0 or 1, setting another intra-cell component to an X value will not prevent propagation already established.”

So does that mean that if a cell input observes S_0, it will have to observe S_1 > S_0 as well? So if there are two cell-input assignments that observe S_0, both will detect S_1 as well, so couldn’t they be included as different tests in the same fault? As detecting either means success at detecting that fault? Wouldn’t the knowledge of where the X is stemming from only matter for differentiating between the S_0 and S_1, not detection?
