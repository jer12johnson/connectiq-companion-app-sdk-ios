# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a fork of Garmin's official `ConnectIQ` Mobile SDK for iOS (`garmin/connectiq-companion-app-sdk-ios`, forked to `jer12johnson/connectiq-companion-app-sdk-ios`). `origin` points at the fork; `upstream` points at Garmin's repo for pulling future SDK updates.

The SDK itself is a **prebuilt binary framework** (`ConnectIQ.xcframework`) distributed as a Swift Package (`Package.swift`) — there is no SDK source to build, lint, or test. The public API surface is defined entirely by the Objective-C headers under `ConnectIQ.xcframework/ios-arm64/ConnectIQ.framework/Headers/` (`ConnectIQ.h`, `IQDevice.h`, `IQApp.h`, `IQAppStatus.h`, `IQConstants.h`).

This repo also serves as the workspace for a larger project — an AI-assisted running pacer that talks to a Garmin watch over ConnectIQ, described by the constitution below. The Garmin SDK is being consumed here, not modified; project-specific pacer code (watch/phone forks of Garmin's Comm sample, the rules engine, the conversational layer) lives in directories added on top of this SDK checkout as the project progresses through its phases.

## Project governance: read the constitution first

**`.specify/memory/constitution.md`** defines 9 binding principles for how this project must be built (phasing order, forking Garmin's reference code instead of rewriting it, keeping deterministic telemetry off the LLM path, push-to-talk-by-default voice input, latency-over-polish for in-run speech, visible degradation on connectivity loss, personal-use-scoped privacy/budget, and what does vs. doesn't get automated test coverage). Any spec, plan, or implementation must comply with it — read it before proposing architecture or scope for new work in this repo.

## Spec-Kit workflow

This repo uses [GitHub Spec Kit](https://github.com/github/spec-kit) for spec-driven development, installed via the `specify` CLI (`pipx install git+https://github.com/github/spec-kit.git`). Shared infrastructure lives in `.specify/`; Claude Code skills live in `.claude/skills/`.

Skills are invoked as slash commands, in this order per feature:

1. `/speckit-constitution` — create/amend `.specify/memory/constitution.md` (governance only; never touches source)
2. `/speckit-specify` — create a baseline feature spec
3. `/speckit-clarify` *(optional)* — de-risk ambiguous areas before planning
4. `/speckit-plan` — create an implementation plan from the spec
5. `/speckit-tasks` — generate actionable tasks from the plan
6. `/speckit-checklist` *(optional)* — generate quality checklists after planning
7. `/speckit-analyze` *(optional)* — cross-artifact consistency check after tasks, before implementing
8. `/speckit-implement` — execute the tasks
9. `/speckit-converge` — assess the codebase and append remaining work as tasks
10. `/speckit-taskstoissues` — turn tasks into tracked issues

`create-new-feature.sh` (in `.specify/scripts/bash/`) is what `/speckit-specify` calls to scaffold a new feature branch/spec directory; `check-prerequisites.sh`, `setup-plan.sh`, and `setup-tasks.sh` back the later steps in the pipeline the same way — you generally don't call these scripts directly, the skills do.

**`.claude/` is gitignored** (may contain agent-local credentials/tokens) — only `.specify/` and this file are committed.

## Working with the ConnectIQ SDK API

The entire SDK is accessed through a single singleton, `[ConnectIQ sharedInstance]` (`ConnectIQ.h`). Key things to know before writing code against it:

- **Initialization is URL-scheme or universal-link based** (`initializeWithUrlScheme:uiOverrideDelegate:` or `initializeWithUniversalLinks:uiOverrideDelegate:`) — the app must be registered to receive callbacks from Garmin Connect Mobile (GCM), which brokers all device communication. There is no direct BLE connection from this SDK; GCM is a required intermediary.
- **Device discovery is delegate-based, not polling.** Register via `registerForDeviceEvents:delegate:` (`IQDeviceEventDelegate`) and wait for `deviceStatusChanged:status:` to reach `IQDeviceStatus_Connected`, then wait further for `deviceCharacteristicsDiscovered:` before the device is actually ready to communicate — a device can report "Connected" before its BLE characteristics are discovered.
- **App messaging is also delegate-based.** Register via `registerForAppMessages:delegate:` (`IQAppMessageDelegate`) to receive `receivedMessage:fromApp:`, and send via `sendMessage:toApp:progress:completion:`, which reports both a progress callback (bytes sent) and a completion callback (`IQSendMessageResult`).
- **`IQSendMessageResult`** (`IQConstants.h`) has granular failure cases (`DeviceNotAvailable`, `AppNotFound`, `DeviceIsBusy`, `Timeout`, `MaxRetries`, `InsufficientMemory`, etc.) — don't collapse these to a single failure branch; the constitution's "visible degradation, never silent failure" principle expects connectivity loss to be distinguishable from other failure modes.
- **`IQDevice`** and **`IQApp`** are simple value objects (UUID + metadata) — `IQDevice` wraps a Garmin device's UUID/model/friendly name/part number, `IQApp` wraps a ConnectIQ app's UUID/store UUID tied to a device. Both conform to `NSSecureCoding`.
