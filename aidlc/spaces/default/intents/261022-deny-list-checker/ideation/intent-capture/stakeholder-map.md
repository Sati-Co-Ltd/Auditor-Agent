# Stakeholder Map: 2026 NHSO DENY-list checker

## Key Stakeholders and Interests

| Stakeholder | Interest | Source |
|-------------|----------|--------|
| K'Pop, SATI CEO and product owner | Decides scope and priority for the checker | [Q4] |
| SATI's clinical coders | Agree the ground-truth sample cases the checker must match | [Q4] |
| Pod B | Extends the ground-truth sample cases to 15-25 | [Q10] |
| Medical-record coders at Sriphat Medical Center (primary users) | Check a case before claim submission and catch DENY risks before rejection | [Q1] |
| Medical Statistics auditors (secondary users) | Review DENY risks across cases | [Q1] |
| Doctors (indirect users) | Affected by DENY risks without using the checker | [Q9] |

## Decision-Makers vs. Influencers

| Stakeholder | Role | Decides | Source |
|-------------|------|---------|--------|
| K'Pop, SATI CEO and product owner | Decision-maker | Scope and priority | [Q4] |
| SATI's clinical coders | Decision-maker for test truth | Which sample cases and expected results are ground truth | [Q4] |
| Medical-record coders (primary users) | User | Nothing stated; see Assumptions & Open Questions | [Q1] |
| Medical Statistics auditors (secondary users) | User | Nothing stated; see Assumptions & Open Questions | [Q1] |
| Doctors (indirect users) | Indirect user | Nothing stated; see Assumptions & Open Questions | [Q9] |

## Communication Requirements

- The audiences that matter are K'Pop (scope and priority) and SATI's clinical coders (ground truth); no communication requirement or cadence has been stated for them, see Assumptions & Open Questions. [Q4]

## Assumptions & Open Questions

- [assumption] No communication requirement or reporting cadence has been stated for the product owner or the clinical coders.
- [assumption] How much say the primary, secondary and indirect users have beyond using the checker has not been stated.
