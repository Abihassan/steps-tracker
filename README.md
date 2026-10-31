# Stride — Software Engineering Roadmap

Project goal: Build an Android step-tracking app first, then extend it into a smartwatch fitness system that can track steps independently on the watch and synchronize with the phone.

Given your available time of 2–4 hours per day and the fact that you haven't chosen a smartwatch yet, I recommend a phased plan over approximately 14–22 weeks for a functional phone-and-watch system, with additional time for production-level polish if needed.

The biggest architectural decision is to make the phone and watch independent activity-recording devices from the start of the design. That means we can build the phone app now without making it the only source of truth for step counting.

## 1. Revised development timeline

Weeks 1–6

### Phase 1 — Android MVP

First usable APK- Expo, React Native, TypeScript and NativeWind.
- Home dashboard, daily goal and history.
- SQLite persistence and settings.
- Health Connect integration and permission handling.
- Build and test an installable APK.

Weeks 7–9

### Phase 2 — Shared fitness architecture

- Define a common activity data model.
- Separate step sources from business logic.
- Implement deduplication and synchronization rules.
- Add schema versioning, import/export and tests.
- Document the watch communication protocol.

Weeks 10–12

### Phase 3 — Select and prototype the watch

- Confirm the target platform and hardware.
- Set up the native watch project.
- Read step data through supported wearable APIs.
- Create the watch dashboard and local persistence.
- Verify that tracking works without the phone nearby.

Weeks 13–17

### Phase 4 — Phone/watch synchronization

- Exchange activity summaries and goal settings.
- Handle temporary disconnection and reconnection.
- Merge records without double counting.
- Track synchronization status and recover from failures.
- Test data consistency on both devices.

Weeks 18–22

### Phase 5 — Hardening and release

- Battery and background-execution testing.
- Date boundaries, reboot and permission tests.
- Offline synchronization and conflict testing.
- Final APK build and watch installation.
- Installation guide and release checklist.

These are planning windows, not fixed deadlines. The watch phase depends on which device you choose, and a functional prototype may arrive sooner than a fully tested release.

## 2. Recommended architecture

Use a native Android watch app and a React Native phone app, connected by a synchronization layer.

## Android phone

React Native · Expo · TypeScript · NativeWind

Dashboard, goals, history, Health Connect and SQLite

Local device-to-device synchronization

## Smartwatch

Native watch UI · Kotlin for Wear OS, if selected

Independent step tracking, local records and offline operation

Shared domain and data contracts

Activity models · Goal rules · Record IDs · Source reconciliation · Sync protocol

### Technology choices

| Layer | Recommended technology |
| --- | --- |
| Phone UI | React Native + Expo |
| Phone styling | NativeWind |
| Phone language | TypeScript |
| Phone persistence | SQLite + AsyncStorage |
| Phone health data | Android Health Connect |
| Wearable UI | Native platform UI; Kotlin + Jetpack Compose for Wear OS |
| Wearable tracking | Wearable platform's supported step/health APIs |
| Wearable persistence | Local database on the watch |
| Communication | Platform-supported device messaging and synchronization |
| Backend | None required for the initial release |

Important: Expo is a good choice for the phone, but the watch app should not depend on Expo or on the phone being connected. A standalone watch app must be able to record activity, store it locally, and synchronize later.

## 3. The most important engineering challenge: avoid double counting

Suppose the phone records 4,000 steps and the watch independently records 5,000 steps. We cannot simply add them and report 9,000 steps: both devices may have counted many of the same physical steps.

The shared data model should retain the origin and identity of each record.

Example:

ts
```
type ActivityRecord = {
  id: string;
  deviceId: string;
  source: "phone_sensor" | "watch_sensor" | "health_connect";
  startTime: string;
  endTime: string;
  steps: number;
  syncedAt?: string;
};

type DailyGoal = {
  targetSteps: number;
  updatedAt: string;
};
```

This is an illustrative starting model, not the final database schema.

### Synchronization rules

- Each record gets a stable ID so repeated synchronization is idempotent.
- Each device saves its own records before attempting synchronization.
- Disconnections never erase unsynchronized activity.
- Daily totals are calculated using a defined source-of-truth policy.
- Overlapping phone and watch measurements are reconciled rather than blindly summed.
- Timestamps are stored consistently, while daily summaries use the user's local calendar date.
- A sync failure should leave the last saved records intact.

For a first release, we should choose one authoritative source for the combined daily total whenever possible. If independent sources must be combined, we will need a tested reconciliation strategy and clear handling for overlapping intervals.

## 4. Milestones and acceptance criteria

## Project milestone tracker

0 of 7

[ ]M1 — Phone foundationWeeks 1–2The app launches, navigation works, and the dashboard renders correctly.

[ ]M2 — Real step dataWeeks 3–4Permissions, real step readings, and unavailable-data states work on a physical phone.

[ ]M3 — Phone MVPWeeks 5–6Goals, SQLite history, persistence and an installable APK are verified.

[ ]M4 — Shared data modelWeeks 7–9Stable IDs, schema migrations, source metadata and sync tests are in place.

[ ]M5 — Watch prototypeWeeks 10–12The selected watch counts and stores steps without requiring the phone to be nearby.

[ ]M6 — SynchronizationWeeks 13–17Offline activity transfers safely, repeated sync creates no duplicate records, and both devices recover after disconnection.

[ ]M7 — Release candidateWeeks 18–22Device, battery, reboot, permission and date-boundary tests pass; APK and watch builds are documented.

 Copy milestone checklist

These milestones should be treated as engineering gates: we only move to the next phase after its acceptance criteria are met.

## 5. README changes

The original README should be updated to reflect the phone-and-watch roadmap rather than describing a phone-only app.

### Replace the project overview

Writing

# Stride — Cross-Device Step Tracker

Stride is an offline-first fitness application designed for Android phones and future smartwatch support.

The phone application uses React Native, Expo, TypeScript, and NativeWind. A compatible smartwatch will run an independent native tracking application capable of recording and storing steps without a phone connection.

## Project Vision

Build a privacy-focused activity-tracking system with:

- Daily step counts and configurable goals.
- Local activity history and progress charts.
- Independent phone and smartwatch tracking.
- Reliable synchronization after connectivity is restored.
- Local-first storage with no mandatory account or cloud backend.
- Installable Android APK and a separate watch installation package where supported.

## Platform Strategy

- **Phone:** React Native + Expo + TypeScript + NativeWind.
- **Phone storage:** SQLite for durable records; AsyncStorage for preferences.
- **Phone health integration:** Android Health Connect, where supported.
- **Smartwatch:** Platform-specific native application selected after evaluating compatible devices.
- **Wear OS option:** Kotlin + Jetpack Compose + supported wearable health APIs.
- **Synchronization:** A versioned, device-to-device protocol with stable record identifiers, retry handling, and duplicate prevention.

The phone and watch must remain independently functional. Neither device should require an active connection to the other to record and save its own activity.

## Extended Development Roadmap

1. Establish the Android application foundation.
2. Implement and test real step-data integration.
3. Add local storage, goals, and activity history.
4. Define shared domain models and synchronization contracts.
5. Select smartwatch hardware and implement independent tracking.
6. Implement offline-first synchronization and source reconciliation.
7. Perform device, battery, permission, and recovery testing.
8. Generate release builds and document installation.

## Cross-Device Data Requirements

Every activity record must identify its source, device, time interval, and stable record ID.

Synchronization must be idempotent, preserve locally saved records, and recover after disconnection.

The application must not blindly add phone and watch totals because their measurements can overlap.

Daily totals must follow a documented source-of-truth and reconciliation policy.

## Release Targets

### Phone MVP

- Verified real step data.
- Configurable daily goals.
- Persistent activity history.
- Offline operation.
- Installable Android APK.

### Phone and Watch Release

- Independent step recording on the selected watch.
- Local persistence on both devices.
- Reliable synchronization and conflict handling.
- Verified behavior when the phone is unavailable.
- Tested phone and watch installation packages.

**Current status:** Architecture and roadmap defined. Implementation and hardware validation remain pending.

### Add this directory structure to the README

text
```
stride/
├── app/                         # Expo routes and phone screens
├── components/                  # Reusable phone UI
├── features/
│   ├── steps/                   # Step state and calculations
│   ├── goals/                   # Goal management
│   ├── history/                 # Activity summaries
│   └── sync/                    # Sync orchestration
├── domain/
│   ├── models/                  # Shared data contracts
│   ├── policies/                # Daily totals and reconciliation
│   └── usecases/                # Platform-independent rules
├── services/
│   ├── health/                  # Health Connect integration
│   ├── sensors/                 # Phone sensor integration
│   ├── database/                # SQLite repositories
│   └── sync/                    # Phone communication adapter
├── tests/
│   ├── unit/
│   └── integration/
├── watch/                       # Separate native watch project later
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DATA_MODEL.md
│   ├── SYNC_PROTOCOL.md
│   └── TEST_PLAN.md
├── app.json
├── eas.json
└── README.md
```

The `watch/` directory can remain a placeholder until we select the watch platform. We should not generate native watch code before that decision, because the implementation and installation process will depend on the platform.

## 6. Risks we should plan for

- Watch selection: Different operating systems expose different sensor APIs and background capabilities.
- Battery life: Continuous tracking and synchronization must be tested on actual hardware.
- Data consistency: Two devices can count the same steps independently.
- Background execution: Operating systems may restrict background tasks or delay synchronization.
- Local-only storage: Device loss or app removal can erase records, so export and backup options should be considered.
- Release compatibility: Android APKs, Wear OS packages, and watch installation requirements may differ.

## 7. Your first development week

I'd begin with these tasks in order:

1. Create the Expo project and configure TypeScript.
2. Configure NativeWind and the navigation structure.
3. Build the home dashboard with a clearly labeled empty-data state.
4. Set up SQLite and the initial daily activity schema.
5. Add daily goal settings and unit tests.
6. Document the data model before implementing device synchronization.

This establishes a usable foundation without spending time on watch-specific code before we know the target hardware.

My recommendation: Target a working phone MVP in 4–6 weeks, then choose the watch before starting native wearable development. With 2–4 hours of focused work per day, plan around 14–22 weeks for the first functional phone-and-watch system, and reserve extra time for real-device validation and release polish.
