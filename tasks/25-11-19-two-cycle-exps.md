# Two-cycle experiments

Email chains:

- Subj: 2-cycle IC-PEPR UDFMs
- Subj: Intel 18A PDK

## Target results

For s1196, then NVDLA, collect

- number of two-cycle PEPR faults
- number of TDF patterns
- percent detected by TDF
- number of patterns and coverage for top-off targeting two-cycle PEPR faults using TDF patterns

Plus

- number of two-cycle PEPR faults after filtering away based on single-cycle redundancy
- percent detected by TDF
- number of patterns and coverage for top-off targeting post-filter two-cycle PEPR faults using TDF patterns
  - not sure if we'd expect this to be different from results without filtering, but I think we've seen better ATPG efficiency with smaller fault lists.

## Action plan

First create repo for experiments

For s1196, then NVDLA

- Run TDF, storing patterns
- Generate two-cycle PEPR faults (`pepr/scripts/create_udfms_for_pixel_db.py`)
- Run fault simulation on two-cycle PEPR faults using TDF patterns
- Run ATPG on remaining faults

Then

- Generate single-cycle PEPR faults
- Run RFI + 1-pattern ATPG on faults
- Run fault simulation on non-redundant using TDF patterns (or reuse existing fault sim results)
- Run ATPG on remaining faults
