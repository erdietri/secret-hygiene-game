> **Editor note — personalize, then remove this note and the idea menu.**
> This is the input to **specify**. Describe observable behavior and learning;
> save implementation choices for `PLAN.md`. Replace every bracketed field.
> The defaults below deliberately fit a short workshop. Change them explicitly
> if your mechanic needs something else. Resolve grading and scope questions
> during **clarify**, before planning the implementation.

# Secret Hygiene Game · First playable specification

## The pitch

**Learn to harden a network automation workflow like a pro**

The player is a network automation engineer hardening an automation workflow in a CI/CD job that pushes config to a network fleet. Each round shows a short pipeline assembled from a few decisions — where a credential comes from, how it's logged, what gets committed or uploaded — and asks the player to choose how the secret is handled at each hop. A correct decision returns a clean Leak Report: every surface (git history, process list, console, stack trace, build artifact) reads CLEAN and the lock turns green at zero exposure. An incorrect decision lights the leaked surfaces EXPOSED in red, showing the exact evidence — the password in a committed file, the token in a debug log line — and traces it back to the decision that caused it, so the fix is obvious on re-run. The session ends when the player clears all three levels with zero secrets exposed across every surface.

## Who this is for

- **Learner:** Network automation engineers and "script kiddies" desiring a code security education
- **Assumed knowledge:** networking, network automation/programmability
- **What the briefing teaches:** **What the briefing teaches:** Three terms and one rule.
- **Secret** — a credential that grants access and must stay unreadable to others: a device password, API token, SSH key, or SNMP community string.
- **Leak surface** — a place a secret can end up where it shouldn't: git history, the process list / shell history, console & CI logs, stack traces, and uploaded build artifacts. These are the surfaces the Leak Report scans.
- **Exposure** — the number of surfaces the secret is readable on after a run. The target is zero. The rule: for each decision, trace where the secret travels — where it's stored, what prints it, what gets committed or uploaded — because a pipeline can complete its task and still leak. A round is won only when the task works *and* exposure is zero.
- **Why interaction helps:** it's not always obvious where secrets are leaked.
- **Session target:** 5–10 minutes for a first playthrough, with unlimited thinking time.

## Learning outcomes and evidence

## Learning outcomes and evidence

| ID | After playing, the learner can… | The game asks them to… | Evidence of success |
| --- | --- | --- | --- |
| LO1 | Identify where a credential leaks based on how it is stored, passed, and logged | Choose the credential source and the log statement for a config-backup script | Leak Report shows 0 exposure — git history, process list, and console all CLEAN — with the task still completing |
| LO2 | Compare competing config-handling choices and select the one that keeps a templated secret out of the repo and its logs | Choose the template value, what gets committed, and the render verbosity | Both git history and the CI render log read CLEAN (0 exposure); the placeholder-at-source path is reached |
| LO3 | Diagnose an indirect leak — trace a secret that reaches a build artifact through the log and stack trace it inherits | Choose the API log level, the error-handling path, and what the job uploads | Console, stack trace, and build artifact all read CLEAN (0 exposure) across the failed-device run |

## The technical model

- **Rule being taught:** A credential is *exposed* if its literal value is readable on any persisted or shared surface after the pipeline runs — source/VCS, process arguments & shell history, console/CI logs, stack traces, or build artifacts. A round is passed only when the task completes **and** exposure = 0. Tie-break / accept-rule: all surfaces must be clean simultaneously — there is no precedence or "least-bad" ranking among them, and a pipeline that completes its task but exposes the secret still fails. Where more than one decision set yields exposure = 0 (Level 2), the game accepts **any** zero-exposure path; it does not require a single canonical answer.
- **Context:** Vendor-neutral, tool-agnostic simplified model. Idioms are drawn from common Python network-automation stacks — `os.environ` / git-ignored `.env` vaults, Jinja-style templating, a generic Git repo, a generic CI runner — but no specific product, vendor, or version is simulated.
- **Assumptions:** When a round starts: the automation task itself always succeeds (only secret handling varies); a vault / git-ignored `.env` exists as the "safe source"; the repo is version-controlled and anything committed is readable by anyone with repo access; CI logs and uploaded artifacts are retained and readable by others; exactly one secret is in play; the player controls only the decisions shown — the rest of the pipeline is fixed.
- **Excluded behavior:** The game does not model secret rotation, expiry, or revocation; access control nuance (private-repo ACLs, encryption-at-rest, short-lived or just-in-time credentials as mitigations); partial exposure (masking, last-4, hashing); secret-scanner detection, entropy, or false positives; alternatives to shared secrets (mTLS, cert-based auth); multi-secret or transitive/third-party leaks; and the blast radius of a leak once it happens. A surface either contains the literal secret or it does not.
- **Authoritative references:**
  - OWASP Secrets Management Cheat Sheet — keep secrets out of source, inject from a manager/vault → Level 1 source choice.
  - The Twelve-Factor App, Factor III (Config) — store config/secrets in the environment → env/vault as the safe source.
  - CWE-798 Use of Hard-coded Credentials (cwe.mitre.org/data/definitions/798.html) → hardcoded secret in source.
  - CWE-214 Invocation of Process Using Visible Sensitive Information (…/214.html) → secret passed as a CLI arg, visible in the process list.
  - CWE-532 Insertion of Sensitive Information into Log File (…/532.html) → secret in console/CI logs and uploaded log artifacts.
  - CWE-209 Generation of Error Message Containing Sensitive Information (…/209.html) → token/connection string in a stack trace.
  - CWE-540 Inclusion of Sensitive Information in Source Code (…/540.html) → secret baked into a committed rendered config.

Tell the learner about these boundaries in plain language. If two answers can both be valid, define how the game accepts them or remove that ambiguity from the first version.

## The repeatable game loop

1. **Brief:** introduce one concept, its terms, and a worked example different from the challenge.
2. **Inspect:** show [PACKET/TABLE/RECORDS/TOPOLOGY] with all decision-relevant values visible.
3. **Choose:** the player [SELECTS/ORDERS/TYPES/BUILDS] and explicitly submits the decision.
4. **Explain:** display the result, the relevant values, and the rule that connects them.
5. **Retry or advance:** a wrong answer leaves the round playable; a correct answer enables Next.
6. **Reflect:** the level summary shows completed rounds, valid attempts, and assistance used.

**Worked interaction:** [WRITE A SHORT SCENE WITH REAL VALUES: WHAT THE PLAYER SEES, A PLAUSIBLE WRONG MOVE, THE EXACT FEEDBACK, AND THE CORRECT MOVE].

## Required stories and acceptance criteria

### US1 · Make and understand a decision — P1

As a learner, I want to apply the rule to visible evidence and understand the result.

- **AC1:** Given a fresh session, when I start, then I see the first briefing before answering a challenge.
- **AC2:** Given [SPECIFIC VALID CHALLENGE], when I submit [CORRECT CHOICE], then [EXACT OUTCOME] and an explanation appear.
- **AC3:** Given that challenge, when I submit [PLAUSIBLE WRONG CHOICE], then feedback identifies [SPECIFIC MISCONCEPTION], completion stays unchanged, and I can retry.
- **AC4:** Given no selection or malformed input, when I submit, then input guidance appears and attempts/completion remain unchanged.

### US2 · Learn from help and finish the game — P1

As a learner, I want help when stuck and a clear view of my progress.

- **AC5:** A hint first explains the method. Further help can show a worked solution; assistance is recorded without marking the round complete.
- **AC6:** Each valid submission counts as one attempt until the round is solved. Correct submission completes it once; repeated submission after success has no effect.
- **AC7:** A level completes after both of its rounds are solved. Next opens the following round or level; the final round opens a session summary.
- **AC8:** Restart asks for confirmation, then clears the current session and returns to the first briefing. Cancel preserves the session.

### US3 · Play with accessible controls — P1

As a keyboard user, I want to inspect the challenge, choose, submit, get help, and advance without a mouse.

- **AC9:** All core controls have visible focus and accessible names; keyboard interaction reaches every core action.
- **AC10:** Correct/incorrect feedback includes text, remains available to read, and is announced to assistive technology. Motion can be reduced without losing information.

## Version-one content

Default: **three levels, two rounds per level, six rounds total**. Use one new example per round and the same core mechanic throughout. Change these counts and AC7 together if you choose a smaller scope.

| Level | New concept | Prerequisite | Misconception to expose |
| --- | --- | --- | --- |
| 1 | [FOUNDATION / LO1] | [STARTING KNOWLEDGE] | [COMMON ERROR] |
| 2 | [APPLICATION / LO2] | Level 1 | [COMMON ERROR] |
| 3 | [TRANSFER OR EDGE CASE / LO3] | Level 2 | [COMMON ERROR] |

**Progression:** sequential on first playthrough; retry without penalty. Completion requires a correct submission in every round. Hints remain available. No points, lives, timer, leaderboard, or streak mechanic in v1.

**Session behavior:** progress is held for the current page session; reload starts again. Saving across reloads is a stretch feature and must be specified before it is added.

## Cases the implementation must satisfy

Write these before generating code. Use real values and independently checked expected results. Include additional cases for every shipped round.

| Case | Input state | Player action | Expected result | Why / source |
| --- | --- | --- | --- | --- |
| Correct | [CONCRETE VALUES] | [ACTION] | [RESULT] | [RULE + REFERENCE] |
| Plausible mistake | [CONCRETE VALUES] | [ACTION] | [RESULT AND EXPLANATION] | [MISCONCEPTION] |
| Boundary | [EXACT BOUNDARY VALUES] | [ACTION] | [RESULT] | [RULE + REFERENCE] |
| Unsupported content | [INVALID OR AMBIGUOUS AUTHOR DATA] | Open round | Visible content error; no grading or progress | Authoring error, not learner failure |

## Experience direction

- **Visual metaphor:** [WHAT MAKES THE CONCEPT VISIBLE].
- **One memorable moment:** [A SMALL EFFECT THAT REVEALS CAUSE AND EFFECT].
- **Always visible:** [VALUES AND LABELS NEEDED TO REASON].
- **Feedback voice:** patient and specific; explain the observed choice before offering the next step.
- **Layout:** challenge first, action next, explanation beside or below it. Don't hide technical values behind hover-only interactions.

## Non-goals

No user accounts, multiplayer, backend, live infrastructure access, runtime AI, leaderboard, or level editor. No simulation of [ADJACENT SYSTEM OR PROTOCOL BEHAVIOR]. Visual polish must fit the existing mechanic.

## Open decisions for clarify

- [QUESTION THAT COULD CHANGE A RULE, A LEARNING OUTCOME, OR THE SCOPE].
- [QUESTION THAT COULD CHANGE WHAT COUNTS AS CORRECT].

Delete this section when resolved, or record the answer beside each question. Don't let unresolved decisions become silent implementation guesses.

## Release evidence

All required acceptance criteria must have a recorded result. All six rounds and their explanations must be checked against the references. A teammate must complete the core flow using only the keyboard. The README must describe implemented behavior, exact run/test commands, and any remaining limits.

---

## Idea menu · remove before using this file as your specification

| Concept | Playful action | Good first scope | Write down before building |
| --- | --- | --- | --- |
| IPv4 routing | Send packets through labeled gates | Choose from an installed forwarding table | Matching rule; default route; no-match behavior; tie assumptions |
| Subnetting | Fit teams into address blocks | Allocate a few fixed requirements | Address model; reserved addresses; whether /31 and /32 are included |
| ACLs | Admit or stop packets at a checkpoint | Evaluate one ordered stateless list | Supported fields; matching semantics; default action |
| DNS | Trace a lookup like a detective | Resolve one name through a staged scenario | Cache state; record types; which actor sends each query |
| NAT/PAT | Repair a translation ledger | Translate a small fixed set of flows | Direction; mapping model; address/port fields |
| VLANs | Dispatch frames through labeled doors | Access/trunk decisions on a tiny topology | Vendor assumptions; tagging rules; allowed VLANs |
| BGP | Compare route cards | A declared subset of one vendor's selection process | Eligible routes; attribute comparison order; excluded tie-breaks |
| QoS | Dispatch traffic into queues | One explicitly modeled scheduling policy | Queue policy; units; offered load; what happens when capacity is exceeded |
| Packet analysis | Assemble evidence for a diagnosis | One curated trace and one fault | What the trace proves; what remains unknown |

Choose the action that makes the networking decision enjoyable. Keep the first version small enough to explain, build, and verify in the workshop.
