# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a fork of Garmin's official `ConnectIQ` Mobile SDK for iOS (`garmin/connectiq-companion-app-sdk-ios`, forked to `jer12johnson/connectiq-companion-app-sdk-ios`). `origin` points at the fork; `upstream` points at Garmin's repo for pulling future SDK updates.

The SDK itself is a **prebuilt binary framework** (`ConnectIQ.xcframework`) distributed as a Swift Package (`Package.swift`) — there is no SDK source to build, lint, or test. The public API surface is defined entirely by the Objective-C headers under `ConnectIQ.xcframework/ios-arm64/ConnectIQ.framework/Headers/` (`ConnectIQ.h`, `IQDevice.h`, `IQApp.h`, `IQAppStatus.h`, `IQConstants.h`).

**This repo is consumed, not built on.** The actual product — an AI-assisted running pacer that talks to a Garmin watch over ConnectIQ — lives in a separate repo, [`jer12johnson/ai-iq-pacing`](https://github.com/jer12johnson/ai-iq-pacing), which depends on this repo as a Swift Package (`Package.swift` dependency pinned to this repo's `main` branch). This repo briefly doubled as that project's workspace before the split; don't add pacer-specific source here — it belongs in `ai-iq-pacing` so this fork stays clean and easy to sync with `upstream` (`garmin/connectiq-companion-app-sdk-ios`) via `git fetch upstream && git merge upstream/main`.

## Project governance

The project constitution and spec-kit workflow live in `ai-iq-pacing`, not here — this repo doesn't have `.specify/`/`.claude/skills/` scaffolding of its own (removed after the split; it was a leftover duplicate that risked drifting out of sync with the canonical copy). Treat `ai-iq-pacing`'s `.specify/memory/constitution.md` as the source of truth for any governance question, and run spec-kit's `/speckit-*` skills there.

## Working with the ConnectIQ SDK API

The entire SDK is accessed through a single singleton, `[ConnectIQ sharedInstance]` (`ConnectIQ.h`). Key things to know before writing code against it:

- **Initialization is URL-scheme or universal-link based** (`initializeWithUrlScheme:uiOverrideDelegate:` or `initializeWithUniversalLinks:uiOverrideDelegate:`) — the app must be registered to receive callbacks from Garmin Connect Mobile (GCM), which brokers all device communication. There is no direct BLE connection from this SDK; GCM is a required intermediary.
- **Device discovery is delegate-based, not polling.** Register via `registerForDeviceEvents:delegate:` (`IQDeviceEventDelegate`) and wait for `deviceStatusChanged:status:` to reach `IQDeviceStatus_Connected`, then wait further for `deviceCharacteristicsDiscovered:` before the device is actually ready to communicate — a device can report "Connected" before its BLE characteristics are discovered.
- **App messaging is also delegate-based.** Register via `registerForAppMessages:delegate:` (`IQAppMessageDelegate`) to receive `receivedMessage:fromApp:`, and send via `sendMessage:toApp:progress:completion:`, which reports both a progress callback (bytes sent) and a completion callback (`IQSendMessageResult`).
- **`IQSendMessageResult`** (`IQConstants.h`) has granular failure cases (`DeviceNotAvailable`, `AppNotFound`, `DeviceIsBusy`, `Timeout`, `MaxRetries`, `InsufficientMemory`, etc.) — don't collapse these to a single failure branch; the constitution's "visible degradation, never silent failure" principle expects connectivity loss to be distinguishable from other failure modes.
- **`IQDevice`** and **`IQApp`** are simple value objects (UUID + metadata) — `IQDevice` wraps a Garmin device's UUID/model/friendly name/part number, `IQApp` wraps a ConnectIQ app's UUID/store UUID tied to a device. Both conform to `NSSecureCoding`.
