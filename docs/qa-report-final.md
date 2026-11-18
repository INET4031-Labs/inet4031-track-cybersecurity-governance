# QA Report: Final Sprint, Cybersecurity and Governance

| | |
|---|---|
| **Section** | Shared VM (team ___) / Local VM |
| **Test author(s)** | ______________________ (Shared VM: QA) |
| **Tester** | ______________________ (Local VM: you are author and tester) |
| **Commit tested** | `git rev-parse --short HEAD` = ________ |
| **Dates** | Cases written: ________ Cases run: ________ |

## How to use this report

1. **Write the test cases first,** in Week 11, before the work they test exists. A test written after the fact tends to test what was built instead of what was asked for.
2. **Write the expected result before you run anything.** If the actual result surprises you, that is a finding, not a typo to fix.
3. **Paste real output or point to an evidence file.** "Works" is not a result. A command and its output is.
4. **Cover every success criterion with at least one case,** and include at least one negative test (something that must fail or be blocked) and the rebuild proof of life.
5. **Re-run every case in Week 13** on the rebuilt VM and update the status. A case that passed only on the old VM is a rebuild gap.

If your team replaced a criterion through [MAKE-IT-YOURS](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/MAKE-IT-YOURS.md), test the replacement instead, and link the approved design brief.

---

## Test Cases

Copy the block for each new case. Six slots are a starting point, not a limit.

<details>
<summary>Example of a filled-in case (delete when you start)</summary>

**TC-00: Web containers do not run as root**

| Field | Value |
|---|---|
| Criterion | 1 |
| Steps | Read the security context of the running `web` (Local VM) or Flask (Shared VM) Deployment with `kubectl get deploy ... -o jsonpath=...` |
| Expected (written before running) | `runAsNonRoot` is true and the user ID is not 0; matches the C-01 row in `controls.csv` |
| Actual | See `capstone/cybersecurity-governance/evidence/<file>`; summary: ... |
| Status | Pass |
| Run on | Original VM, YYYY-MM-DD; rebuilt VM, YYYY-MM-DD |

</details>

### TC-01: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-02: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-03: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-04: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-05: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

### TC-06: (name your check)

| Field | Value |
|---|---|
| Criterion | |
| Type | Positive / Negative (must fail or be blocked) / Rebuild |
| Preconditions | |
| Steps (exact commands) | |
| Expected (written before running) | |
| Actual (output or evidence path) | |
| Status | [ ] Pass [ ] Fail [ ] Blocked |
| Run on | Original VM: ____ Rebuilt VM: ____ |
| Notes | |

---

## Success Criteria Verification

Evidence paths are relative to `capstone/cybersecurity-governance/` and come from [EVIDENCE-CHECKLIST](https://github.com/INET4031-Labs/inet4031-capstone-guide/blob/main/EVIDENCE-CHECKLIST.md). Change a path if your file is named differently.

| Criterion | Evidence | Test case(s) | Met? |
|---|---|---|---|
| 1. Control matrix with 15+ controls, each with status and an evidence path | `controls.csv` | | [ ] Yes [ ] No |
| 2. Image and manifest scans, with HIGH/CRITICAL image findings reduced (before and after) | `evidence/trivy-image-before.json`, `evidence/trivy-image-after.json`, manifest scans | | [ ] Yes [ ] No |
| 3. Full-history gitleaks report with 0 unresolved findings | `evidence/gitleaks-report.json` | | [ ] Yes [ ] No |
| 4. 8+ `kubectl auth can-i` checks and 2+ blocked NetworkPolicy tests, all matching written predictions | `evidence/rbac-checks.txt`, `evidence/netpol-tests.txt` | | [ ] Yes [ ] No |
| 5. Risk register (8+ risks), 3 one-page policies, evidence script under 5 minutes | `risk-register.md`, `policies/`, the script and one timestamped output folder | | [ ] Yes [ ] No |

---

## Rebuild Verification (Week 13)

| Check | Result |
|---|---|
| Rebuild date and VM (new VirtualBox VM, or Shared VM queue slot) | |
| Started from commit | |
| Proof of life, Shared VM: The evidence script produces a new timestamped folder; the statuses in `controls.csv` still match what the new evidence shows | [ ] Pass [ ] Fail |
| Proof of life, Local VM: Same as the Shared VM | [ ] Pass [ ] Fail |
| Every app change came back (schema, routes, services, images) | [ ] Yes [ ] No |
| Gaps logged in `capstone/cybersecurity-governance/REBUILD-GAPS.md` and fixed by pull request | ___ gaps found, ___ fixed |
| All test cases above re-run on the rebuilt VM | [ ] Yes [ ] No |

---

## Deliverables Verification

- [ ] Implementation committed under `capstone/cybersecurity-governance/` (merged by pull request)
- [ ] `capstone/cybersecurity-governance/RUNBOOK.md`: operate, check, and recover
- [ ] `capstone/cybersecurity-governance/TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages)
- [ ] `capstone/cybersecurity-governance/ARCHITECTURE.md` with a diagram and a "What we changed and why" section
- [ ] "How to run this" section in the top-level README
- [ ] `capstone/cybersecurity-governance/REBUILD-GAPS.md` from the Week 13 rebuild
- [ ] This QA report, with every case run and signed
- [ ] Shared VM: `docs/final-sprint-retrospective.md`. Local VM: the Individual Reflection section
- [ ] `capstone/cybersecurity-governance/DESIGN-BRIEF.md`, only if a criterion was replaced, with the approval noted
- [ ] Discussion answers in the Google Doc
- [ ] Final presentation and Week 14 rebuild presentation prepared (live on the Shared VM, recorded on the Local VM)
- [ ] Track-specific: `controls.csv`
- [ ] Track-specific: `risk-register.md` and `policies/`
- [ ] Track-specific: The evidence script and at least one timestamped evidence folder

---

## Rework Required

| # | Failed case or criterion | What is wrong | Fix and owner | Re-test date | Re-test result |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |

---

## Sign-Off

**Overall status:** [ ] All criteria met [ ] Met with documented exceptions [ ] Rework required

**Exceptions** (criteria not met, and why):

```

```

| Role | Name | Date |
|---|---|---|
| Test author | | |
| Tester (Local VM: same person) | | |

**Notes:** observations about technical quality and, on the Shared VM, team coordination.
