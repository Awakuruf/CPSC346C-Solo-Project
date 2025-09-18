# PIA scratch

## Data we collect
- user_id (hashed)
- coarse location grid
- aggregate history (30 days)
- optional environment, bait, target species

## Risks
- Re-identification if grid too fine
- Misuse of location history
- Data poisoning (fake reports)

## Mitigations
- k-anonymity for aggregates
- No raw GPS unless opt-in
- Raw logs deleted after 30 days