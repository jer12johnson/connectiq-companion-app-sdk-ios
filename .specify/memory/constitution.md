<!--
Sync Impact Report
Version change: [TEMPLATE] → 1.0.0 (initial ratification)
Modified principles: n/a (first concrete adoption; all 9 principles newly defined)
Added sections:
  - Core Principles I–IX (Risk-First Phasing; Fork, Don't Rebuild; Telemetry Off the LLM Path;
    Push-to-Talk by Default; Speed Over Polish for Mid-Run Speech; Visible Degradation, Never
    Silent Failure; Personal-Use Privacy Scope; Test What You Can, Field-Test the Rest;
    Personal-Use Cloud Budget)
  - Governance
Removed sections:
  - Generic [SECTION_2_NAME] / [SECTION_3_NAME] template slots (no content beyond the nine
    principles was supplied; removed rather than left as placeholders)
Deferred / TODO placeholders: none
Templates requiring follow-up: none checked yet — re-validate plan/spec/tasks templates against
  these principles the next time they are used (no existing dependent artifacts to update yet)
-->

# AI IQ Pacing Constitution

## Core Principles

### I. Risk-First Phasing
Build in the order that retires the biggest unknown first: Garmin connectivity proof-of-concept,
then a non-conversational rule-based pacer, then the conversational layer, then proactive
scheduling, then hardening. A phase MUST NOT start until the prior phase's exit criteria are
verified in a real run on real hardware — passing in the simulator alone is not sufficient.
**Rationale**: Connectivity and hardware behavior are the least predictable parts of this project;
sequencing work to fail fast on those risks first avoids investing in conversational or scheduling
features on top of an unproven transport layer.

### II. Fork, Don't Rebuild
The watch and phone components MUST fork Garmin's official Comm sample and the
`connectiq-companion-app-sdk-ios` reference app rather than being written from scratch.
**Rationale**: Garmin's reference code already encodes the correct BLE/GCM handshake behavior;
reimplementing it is wasted effort and a source of subtle protocol bugs for a single-developer,
personal-use project.

### III. Telemetry Off the LLM Path
Deterministic, numeric feedback — splits, pace-vs-target callouts — MUST go straight to templated
TTS and MUST NOT be routed through the LLM. The LLM is reserved for responses that require
judgment or conversation.
**Rationale**: Real-time numeric feedback needs to be fast and exactly correct every time; an LLM
adds latency and non-determinism with no benefit for output that is already fully specified by the
telemetry itself.

### IV. Push-to-Talk by Default
Voice input MUST start push-to-talk, not open-mic, given wind/breathing noise and battery cost
outdoors. Open-mic MAY be revisited only if push-to-talk proves too disruptive in practice.
**Rationale**: Open-mic capture outdoors during running is unreliable (wind/breath noise) and
battery-expensive; push-to-talk is the safer default until real usage shows otherwise.

### V. Speed Over Polish for Mid-Run Speech
Wherever a vendor tradeoff exists between response quality and response speed for anything spoken
mid-run, speed MUST win.
**Rationale**: A slow but polished response is worse than a fast, good-enough one when the user is
mid-stride and waiting on feedback; latency directly affects usability during exercise.

### VI. Visible Degradation, Never Silent Failure
Connectivity loss — watch-to-phone or phone-to-cloud — MUST be detected and handled visibly. It
MUST NOT be allowed to fail invisibly mid-run.
**Rationale**: A silent failure during a run means the user only discovers the app stopped working
after the fact, when nothing can be done about it; visible degradation lets the user adapt in the
moment.

### VII. Personal-Use Privacy Scope
Privacy and data handling MUST stay sensible and proportionate to a single-user tool — GPS/HR data
is not handled carelessly — without over-building for App Store review or public-distribution
requirements this project does not yet need. This scope MUST be revisited if the project ever
shifts toward sharing or publishing.
**Rationale**: Building compliance infrastructure for distribution scenarios that don't exist yet
would divert effort from the actual product risk areas without a corresponding benefit.

### VIII. Test What You Can, Field-Test the Rest
Logic that can be validated without real hardware — the rules engine's thresholds, message
parsing, the function-calling tool, scheduler/turn-taking logic — MUST have automated test
coverage. Logic that depends on actual BLE behavior, the GCM handshake, or real running conditions
MUST be validated through field testing instead; the absence of automated coverage there is not a
gap in rigor, it is the correct tool for that layer.
**Rationale**: Automated tests can't meaningfully simulate real BLE/GCM hardware behavior or
outdoor running conditions, so demanding coverage there would produce false confidence rather than
real verification.

### IX. Personal-Use Cloud Budget
Vendor and architecture choices for the conversational phase MUST favor cost-effective options
sized for one person's running habit, not enterprise pricing tiers — but never at the expense of
Principle V (Speed Over Polish) or Principle VI (Visible Degradation, Never Silent Failure). No
hard dollar ceiling is set yet; this MUST be revisited once Phase 3 produces real usage data
against real vendor pricing.
**Rationale**: This is a single-user tool, so enterprise-tier vendor pricing and scaling
architecture would be over-engineering; but cost minimization must never be allowed to compromise
the two principles that protect the in-run user experience.

## Governance

This constitution supersedes all other project practices and templates. Any spec, plan, task list,
or implementation that conflicts with a principle here MUST be revised or the conflicting principle
MUST be amended first — silently overriding a principle in downstream artifacts is not permitted.

**Amendment procedure**: Amendments are made by editing this file directly, prepending an updated
Sync Impact Report, and recording the rationale for the change inline with the affected principle.
There is no separate approval body since this is a single-developer project; the acting developer
is the approver.

**Versioning policy**: This constitution follows semantic versioning:
- **MAJOR** — a principle is removed or redefined in a backward-incompatible way.
- **MINOR** — a new principle or materially expanded section is added.
- **PATCH** — wording clarifications, typo fixes, or non-semantic refinements.

**Compliance review**: Each phase transition under Principle I (Risk-First Phasing) is also a
compliance checkpoint — before advancing to the next phase, confirm the completed phase's work
did not violate any other principle (e.g., that telemetry didn't leak onto the LLM path, that
failures during field testing were surfaced rather than silently swallowed).

**Version**: 1.0.0 | **Ratified**: 2026-08-03 | **Last Amended**: 2026-08-03
