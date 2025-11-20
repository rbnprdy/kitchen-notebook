# Two-cycle UDFMs

[Shawn's google doc](https://docs.google.com/document/d/1hSw3mEaZYD6MQHoaNxHFXXk120M9YVlw1MUdkjHuoi8) describes method for creating two-cycle UDFMs from a set of single-cycle UDFMs. This can be implemented directly in code, flag to turn on nested loop where single-cycle UDFMs are generated.

## Considerations

- Make sure controllability and observability is taken into account properly (filtering on controllability of the first cycle and both on second cycle)
