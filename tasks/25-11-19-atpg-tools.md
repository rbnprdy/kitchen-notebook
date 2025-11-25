# ATPG Tools

Maybe should create a helper package called `atpgtools` that has funcitons for running RFI, ATPG, top off ATPG, etc. for use in experiments outside of `run-flow`. Or maybe the flow is flexible enough and do top-off just through the `simulate` functionality of it, can just add an option to read in patterns in for that. Even in that case could still add a small utility package for running SAF/TDF ATPG (and I guess at that point adding UDFM ATPG, RFI, and top off is pretty easy too).

## Implementation

Seems like `pytessent.atpg` actually has what we need already. Added a few new features for flexibility and it looks good.
