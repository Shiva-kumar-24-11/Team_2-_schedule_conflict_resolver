# Member 3 – Failure Log and Checker Evidence

## Manual Checker Results

| Test | Version | Failure/Result | Evidence | Root Cause / Observation | Fix Status |
|---|---|---|---|---|---|
| **T04** | Unknown | **FAIL** | part1.txt line 540-541 | Event `a` moved earlier than its original UTC start (2026-10-12T04:30Z), violating R7. Earlier attempt also failed with invalid JSON shape. | V4 proposed to reinforce R7 in hard constraints and self-check; not yet tested |
| **T06** | Unknown | **FAIL** | part1.txt line 536-537 | Event `b` moved earlier than its original UTC start (2026-10-12T08:00Z), violating R7. Hidden UTC overlap not resolved by moving forward. | V4 proposed; not yet tested |
| **T09** | Unknown | **PASS** | part1.txt line 542-543 | Inherited priority correctly applied: low-priority task `a` kept because mandatory task `b` depends on it. Event `c` dropped per R5. | No fix required based on this evidence |
| **T13** | Unknown | **PASS** | part1.txt line 544-545 | Impossible case correctly identified: two mandatory fixed events overlap in UTC. Checker accepted infeasible status. | No fix required based on this evidence |

---

## Detailed Failure Analysis

### T04: Invalid JSON and R7 Violation

**Stored log evidence:**
```
PS D:\Downloads\Project> python run_eval.py check T04 out.txt
T04 FAIL: output was not valid JSON in the required shape
PS D:\Downloads\Project> python run_eval.py check T04 out.txt
T04 FAIL: a: moved earlier than its original start 2026-10-12T04:30Z (R7)
```

**Test case (from test_cases.json):**
- Event `a`: Architecture review, 2026-10-12T10:00-11:00 Asia/Kolkata, priority 5, movable
- Event `b`: Dentist, 2026-10-12T10:30-11:00 Asia/Kolkata, priority 1, fixed
- Conflict: 10:30-11:00 UTC (both events)
- Expected: Event `a` moves, event `b` stays (R4: move before drop, even for high-priority movable)

**UTC times:**
- Event `a` original: 2026-10-12T04:30-05:30Z
- Event `b` fixed: 2026-10-12T05:00-05:30Z
- Overlap: 2026-10-12T05:00-05:30Z

**What went wrong:**
The model scheduled event `a` at an earlier UTC time than 2026-10-12T04:30Z. This is precisely what R7 forbids: moved events must respect the original time as a lower bound.

**Why this matters:**
R7 is a hard constraint on direction of movement. It prevents the model from finding "clever" rearrangements that technically fit but violate planning expectations. If someone originally planned a 10:00am meeting, moving it to 9:30am (earlier than originally intended) is not acceptable, even if it creates a smaller time shift.

**Fix (V4 proposal):**
- Make "moved events never start earlier" a hard constraint in R2, not just a placement hint
- State the rationale explicitly in R2
- Repeat it in R7 as an enforcement rule
- Add it to the self-check: "every moved event starts at or after its own start_utc in 'normalized'"

**V4 Status:** Not tested

---

### T06: Hidden Timezone Conflict, R7 Violation

**Stored log evidence:**
```
PS D:\Downloads\Project> python run_eval.py check T06 out.txt
T06 FAIL: b: moved earlier than its original start 2026-10-12T08:00Z (R7)
```

**Test case (from test_cases.json):**
- Event `a`: Payroll cut-off, 2026-10-12T14:00-15:00 Asia/Kolkata, priority 4, fixed
- Event `b`: UK supplier call, 2026-10-12T09:00-10:00 Europe/London, priority 2, movable
- Window: 2026-10-12T09:00-18:00 Asia/Kolkata
- Expected: Both events kept, no drops

**Local times (appear non-conflicting):**
```
Asia/Kolkata:  14:00-15:00 (event a)
Europe/London: 09:00-10:00 (event b)
```
These look 5+ hours apart, so no conflict.

**UTC times (actual conflict):**
```
Event a (Asia/Kolkata 14:00-15:00):       2026-10-12T08:30-09:30Z
Event b (Europe/London 09:00-10:00):       2026-10-12T08:00-09:00Z
```
Overlap: **2026-10-12T08:30-09:00Z** (30 minutes)

**Why this is a critical test:**
T06 verifies that the scheduler correctly:
1. Normalizes both times to UTC using their respective time zones
2. Detects the UTC overlap
3. Resolves by moving the non-fixed event (b) forward in time, not earlier

The failure shows the model **did detect the overlap** (correctly) but then **moved event b to an earlier UTC time** (incorrectly). Instead of moving forward to the earliest valid slot after event a, it found a smaller shift backwards, which R7 forbids.

**Fix (V4 proposal):** Same as T04—strengthen R7 as a hard constraint.

**V4 Status:** Not tested

---

### T09: Inherited Priority (PASS)

**Stored log evidence:**
```
PS D:\Downloads\Project> python run_eval.py check T09 out.txt
T09 PASS
```

**Test case (from test_cases.json):**
- Event `a`: Prepare slides, 2026-10-12T11:00-12:00 Asia/Kolkata, priority 1, fixed
- Event `c`: Gym session, 2026-10-12T11:30-12:30 Asia/Kolkata, priority 3, fixed
- Event `b`: Client pitch (mandatory), 2026-10-12T15:00-16:00 Asia/Kolkata, priority 5, fixed, depends_on [`a`]
- Conflicts: `a` and `c` overlap (11:30-12:00)
- Expected: Event `c` dropped because its effective priority (3) is lower than `a`'s inherited priority (5, because `b` depends on `a`)

**Why this matters:**
R3 (dependencies) grants "inherited priority" to prerequisite tasks. Event `a` has explicit priority 1, but because the high-priority mandatory task `b` depends on it, `a` carries effective priority 5. This means `c` (priority 3) loses the conflict and is dropped.

**Evidence:**
The checker accepted the output, confirming that:
- Event `c` was dropped (expected_dropped: ["c"])
- Event `b` correctly depends on `a`
- No other violations (overlaps, fixed movement, etc.)

**Conclusion:** No fix required. This test passes, indicating the model correctly handles inherited priority.

---

### T13: Impossible Case (PASS)

**Stored log evidence:**
```
PS D:\Downloads\Project> python run_eval.py check T13 out.txt
T13 PASS
```

**Test case (from test_cases.json):**
- Event `a`: Thesis defence, 2026-10-12T11:00-12:00 Asia/Kolkata, priority 5, fixed, mandatory
- Event `b`: Visa interview, 2026-10-12T14:00-15:00 Asia/Singapore, priority 5, fixed, mandatory
- Expected status: `infeasible` (not `ok`)

**UTC times:**
- Event `a` (Asia/Kolkata 11:00-12:00): 2026-10-12T05:30-06:30Z
- Event `b` (Asia/Singapore 14:00-15:00): 2026-10-12T06:00-07:00Z
- Overlap: **2026-10-12T06:00-06:30Z**

**Why it's impossible:**
Both events are:
- Fixed (cannot move)
- Mandatory (cannot drop)
- Overlapping in UTC

No rule can resolve this. Per R2, fixed events keep their exact times and mandatory events are never dropped. Therefore, the input violates R2 and is infeasible.

**Evidence:**
The checker accepted status = `infeasible`, confirming the model correctly identified the impossible constraint conflict.

**Conclusion:** No fix required. The system correctly recognizes infeasibility and refuses to bend rules.

---

## Summary of Findings

### Common Pattern in T04 and T06

Both failures involve the same rule violation: **R7 placement constraint**.

- **T04:** Event moved to start earlier than original UTC time
- **T06:** Event moved to start earlier than original UTC time (hidden by timezone)

**Hypothesis:**
The model may be optimizing for time shift magnitude rather than adherence to directional constraints. Or it may not have clearly internalized that "earliest free slot" means earliest *after* the original start, not earliest in absolute terms.

**V4 Response:**
Make the constraint undeniable by stating it in multiple places (R2 hard constraint, R7 rule, self-check) and with explicit reasoning ("people planned around the original time").

### Passing Cases

T09 and T13 show the system correctly handles:
- Inherited priority through dependencies (R3)
- Recognition of impossible cases (R2 violation leads to infeasible status)

These suggest the core logic for complex cases works; the failures are specific to the directional movement constraint.

---

## Important Note

**These are manual checker results, not automated benchmark results.**

The model used is unknown, and only 4 of 20 test cases were checked. Do not calculate a pass rate or attribute these results to a specific prompt version or model without further investigation.

V4 remains a **proposal**, not a proven solution.
