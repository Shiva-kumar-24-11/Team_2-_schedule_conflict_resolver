# Test Results Summary

## Official Benchmark Dataset

**Location:** `Project/test_cases.json`

**Total cases:** 20 (T01 through T20)

**Coverage:**
- Basic scheduling: T01-T03
- Move-before-drop: T04
- Timezone handling: T05-T06, T12
- Dependencies: T07-T09
- Window constraints: T10-T11
- Impossible cases: T13-T15
- Invalid input: T16-T17
- Safety: T18-T19
- Free-text extraction: T20

---

## Verified Stored Evidence

### Manual Checker Results (from `part1.txt`)

| Test | Result | Checker Run | Evidence Line |
|---|---|---|---|
| T04 | **FAIL** | `python run_eval.py check T04 out.txt` | part1.txt line 540-541 |
| T06 | **FAIL** | `python run_eval.py check T06 out.txt` | part1.txt line 536-537 |
| T09 | **PASS** | `python run_eval.py check T09 out.txt` | part1.txt line 542-543 |
| T13 | **PASS** | `python run_eval.py check T13 out.txt` | part1.txt line 544-545 |

**Note:** These results came from manually:
1. Displaying a test case with `python run_eval.py show <test_id>`
2. Submitting the input to an unspecified external model
3. Saving the model response to `out.txt`
4. Running `python run_eval.py check <test_id> out.txt`

**Model identity:** UNKNOWN

---

## Automated Benchmark Status

**Status:** NOT COMPLETED

### Why Automated Run Has Not Been Completed

File: `Project/run_eval.py` lines 23-26

```python
def call_model(system, messages):
    """Send a system prompt and a list of {"role": "user"|"assistant", "content": str}; return the reply text.
    FILL THIS IN for the model your team is permitted to use. Use temperature 0 if the API allows it."""
    raise NotImplementedError("Fill in call_model() in run_eval.py for your permitted model.")
```

The function `call_model()` is a placeholder and raises `NotImplementedError`. To run an automated benchmark, this function must be implemented with actual API calls to a model.

### Full Run Attempt (from `part1.txt` line 374-386)

```
PS D:\Downloads\Project> python run_eval.py V3
Traceback (most recent call last):
  File "D:\Downloads\Project\run_eval.py", line 121, in <module>
    main(sys.argv[1:])
  File "D:\Downloads\Project\run_eval.py", line 106, in main
    data, out, trace = run_case(c, VERSIONS[version], repair="--repair" in flags, guard="--no-guard" not in flags)
  File "D:\Downloads\Project\run_eval.py", line 61, in run_case
    raw = call_model(system, messages)
  File "D:\Downloads\Project\run_eval.py", line 26, in call_model
    raise NotImplementedError("Fill in call_model() in run_eval.py for your permitted model.")
NotImplementedError: Fill in call_model() in run_eval.py for your permitted model.
```

---

## Pass Rate

**NOT CALCULATED**

### Why a Pass Rate Cannot Be Reported

1. **Incomplete evidence:** Only 4 of 20 cases have checker results. The other 16 cases (T01, T02, T03, T05, T07, T08, T10, T11, T12, T14, T15, T16, T17, T18, T19, T20) are not verified.

2. **Non-representative sample:** The four verified cases were selected manually, not randomly. They happen to be specific edge cases (two R7 violations, one inherited priority, one impossible case).

3. **Unknown model:** The model used to generate the responses is not documented. Pass rate percentages are only meaningful when attributed to a specific model, version, and configuration.

4. **Unknown prompt version:** The manual responses were not generated through the automated `run_eval.py` framework, so the prompt version used (V1, V2, V3, V4, or something else) is unknown.

5. **Inconsistent evaluation:** A pass rate requires running all 20 cases through the same model with the same temperature and settings. The manual checks do not meet this standard.

### What the 4 Results Do Show

- The checker correctly identifies R7 violations (T04, T06 FAIL)
- The checker accepts complex correct outputs (T09 PASS)
- The checker recognizes infeasible cases (T13 PASS)

**This does not indicate the system's overall performance on the benchmark.**

---

## Prompt Version Testing Status

| Version | Defined? | Executable? | Tested? | Evidence |
|---|---|---|---|---|
| **V1** | ✅ YES (prompts new.py line 171-176) | ❌ NO — requires `call_model()` | ❌ NOT TESTED | Baseline prompt |
| **V2** | ✅ YES (prompts new.py line 181) | ❌ NO — requires `call_model()` | ❌ NOT TESTED | Added R1-R7 rules |
| **V3** | ✅ YES (prompts new.py line 183) | ❌ NO — requires `call_model()` | ❓ PARTIALLY — 4 manual checks with unknown model | Refined prompt + few-shot examples |
| **V3 + repair** | ✅ YES (run_eval.py line 60-69) | ❌ NO — requires `call_model()` | ❌ NOT TESTED | Repair loop framework ready |
| **V4** | ✅ YES (prompts new.py line 208-210) | ❌ NO — requires `call_model()` | ❌ NOT TESTED | Proposed fix for R7 failures |

### V4 Specifics

File: `prompts new.py` lines 185-210

**V4 is a proposed improvement**, not a tested version.

**Changes from V3:**
1. **R2 (hard constraints):** Adds explicit language about never moving events earlier than their original start
2. **R7 (placement):** Reinforces the same constraint with examples of rejected alternatives
3. **Self-check:** Adds a verification step to ensure all moved events satisfy the directional constraint

**Design rationale:**
V3 mentioned the "not earlier" rule only once. T04 and T06 failures suggest it was either missed or deprioritized. V4 makes it impossible to overlook by embedding it as a hard constraint (not a hint) and requiring explicit verification.

**Status:** **NOT TESTED**

---

## Evidence Limitations

### What Is Known With Certainty

✅ 20 test cases are defined in `test_cases.json`

✅ Four manual checker runs are documented in `part1.txt`:
   - T04 FAIL (R7 violation)
   - T06 FAIL (R7 violation)
   - T09 PASS (inherited priority)
   - T13 PASS (impossible case)

✅ The programmatic checker (`check_schedule()`) correctly identifies the violations

✅ V4 prompt is defined with proposed fixes

### What Is Unknown

❌ Which model produced the responses for T04, T06, T09, T13

❌ Which prompt version (V1, V2, V3, V4) was used in the manual checks

❌ Whether V4 fixes the T04 and T06 failures

❌ How the 16 unverified cases perform

❌ What the pass rate is for any prompt version on the full 20-case benchmark

### Consequences

- **No overall accuracy or pass rate can be claimed without further testing**
- **No performance comparison between V1, V2, V3, and V4 exists**
- **The framework is complete but awaiting implementation of `call_model()` for automated execution**

---

## Next Steps for Complete Evaluation

To complete the automated benchmark:

1. **Implement `call_model()` in `run_eval.py`** with a specific model API (OpenAI, Gemini, etc.)
2. **Set temperature = 0** for deterministic results
3. **Run:** `python run_eval.py V1` (all 20 cases)
4. **Run:** `python run_eval.py V2` (all 20 cases)
5. **Run:** `python run_eval.py V3` (all 20 cases)
6. **Run:** `python run_eval.py V3 --repair` (with repair loop)
7. **Run:** `python run_eval.py V4` (all 20 cases)
8. **Store results** in `runs/` directory with timestamps
9. **Compare pass rates** across versions
10. **Document which version performs best** on the official benchmark

---

## Summary

| Metric | Value | Status |
|---|---|---|
| **Official test cases** | 20 | ✅ Defined |
| **Manual checker evidence** | 4/20 | ✅ Stored in part1.txt |
| **Automated benchmark runs** | 0/5 versions | ❌ Incomplete (`call_model()` not implemented) |
| **V4 testing** | Not tested | ❌ Proposed, awaiting implementation |
| **Repair loop testing** | Not tested | ❌ Framework ready, awaiting model |
| **Overall pass rate** | Not calculated | ❌ Cannot be claimed |

**Conclusion:** The testing infrastructure and evaluation methodology are in place. The framework awaits implementation of `call_model()` to enable automated benchmarking and final comparison of prompt versions.
