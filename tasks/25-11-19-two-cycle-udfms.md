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
