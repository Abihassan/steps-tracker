We’ll approach this like a real software engineering project: define the architecture, choose the right Android technologies, plan local data storage, document the implementation, and then build the app in manageable, testable stages.

One important technical decision: React Native + Expo + NativeWind (Tailwind CSS) + Android Health Connect is a strong foundation for this project. We'll use Expo to build an installable Android APK, and local storage so the app can work without an account or backend.

For step tracking, we should distinguish between reading steps recorded by Android's health platform and counting steps directly from the phone's sensors. The recommended initial implementation will use Android Health Connect where available, with an explicit plan for permissions, unavailable data, and testing. We should not pretend that a normal JavaScript timer can count real steps in the background.

# Step 1 — Create `README.md`

I'll define the project as a local-first Android step-tracking application. The README below will be our technical blueprint for implementation, testing, and APK delivery.

## Stride — Step Tracker

Android · React Native · Expo · NativeWind

Project spec v1.0

The product goal

A clean, reliable mobile app that tracks daily steps, shows progress toward a daily goal, stores historical records locally, and can be installed on Android without publishing to the Play Store.

Offline-firstPrivacy-focusedAPK build

 Copy project summary

## `README.md` — initial project specification

Writing

# Stride — Android Step Tracker

> An offline-first Android step-tracking application built with React Native, Expo, TypeScript, and NativeWind.

## 1\. Project Overview

Stride is a mobile application that helps users monitor their daily physical activity through step counting, progress tracking, historical summaries, and customizable daily goals.

The app prioritizes privacy, low resource usage, a clean interface, and reliable Android integration.

### Objectives

- Track daily steps using supported Android health or sensor APIs.
- Display daily progress toward a configurable step goal.
- Show historical step counts and weekly trends.
- Store settings and activity summaries locally.
- Operate without a user account or mandatory backend.
- Support Android permissions and lifecycle changes.
- Produce an installable Android APK using Expo's build tooling.

## 2\. Technology Stack

| Technology | Responsibility |
| --- | --- |
| React Native | Cross-platform mobile UI |
| Expo | Development workflow and native project tooling |
| TypeScript | Type-safe application logic |
| Expo Router | File-based navigation |
| NativeWind | Tailwind CSS-style utility classes |
| Android Health Connect | Access to supported health and step records |
| Android step sensor integration | Optional direct device step counting |
| AsyncStorage | Lightweight local preferences and settings |
| SQLite | Persistent daily activity records and history |
| React Native SVG | Progress rings and charts |
| Expo Application Services (EAS) | Android APK builds |

**Architecture decision:** Use Expo with a development build and native prebuild support where required. Do not assume Expo Go supports every native health or sensor integration.

## 3\. Core Features

### 3.1 Dashboard

- Current day's step count.
- Circular progress indicator.
- Configurable daily goal.
- Remaining steps to reach the goal.
- Distance estimate and activity duration when reliable data is available.
- Daily goal completion state.
- Last synchronization time and data availability status.

### 3.2 Step History

- Daily activity summaries.
- Seven-day overview.
- Weekly and monthly trends.
- Previous-day step totals.
- Empty states when activity data is unavailable.

### 3.3 Goals and Settings

- Default daily goal of 8,000 steps.
- Editable daily step goal.
- Metric display preferences.
- Permission status and data-source information.
- Clear local activity history.
- Privacy information.

### 3.4 Reliability and Privacy

- Handle denied and revoked permissions.
- Handle devices without a supported step source.
- Prevent duplicate records when synchronizing.
- Use local persistence across application restarts.
- Never fabricate real sensor readings.
- Do not upload health data to a server.

## 4\. Step Counting Strategy

Use Android Health Connect as the preferred source for historical and aggregated step records when supported and authorized.

An optional native sensor integration may use Android's step counter APIs for more immediate readings, subject to device support and runtime permissions.

The application must distinguish between:

1. **Recorded steps:** Steps retrieved from the supported health data source.
2. **Live sensor readings:** Values reported by a device's step sensor.
3. **Estimated activity:** Derived values such as approximate distance.

Do not add a cumulative hardware sensor reading directly to a Health Connect daily total. These sources can overlap and cause double counting.

### Android integration requirements

- Declare the appropriate Health Connect permissions.
- Request runtime access before reading step records.
- Check whether Health Connect is available on the device.
- Handle Android version differences and missing providers.
- Configure the required manifest entries and privacy declarations.
- Use a compatible native module or development build when needed.

If no valid data source is available, display an informative unavailable state instead of presenting simulated steps as real activity.

## 5\. Local Storage Design

Use SQLite as the primary database and AsyncStorage for small application preferences.

### Daily activity table

| Field | Type | Purpose |
| --- | --- | --- |
| id | INTEGER | Primary key |
| activity\_date | TEXT | Local calendar date, YYYY-MM-DD |
| steps | INTEGER | Recorded daily step total |
| distance\_meters | REAL | Optional distance estimate |
| active\_minutes | INTEGER | Optional activity duration |
| source | TEXT | Origin of the data |
| synced\_at | TEXT | Last successful synchronization |

Add a unique constraint on `activity_date` and use transactional upserts to prevent duplicate daily summaries.

If multiple independent sources must be retained, use a separate source-record table and aggregate carefully rather than overwriting or adding overlapping totals.

### Local preferences

Store the following locally:

- Daily step goal.
- Distance unit.
- Theme preference.
- Onboarding completion.
- Last successful synchronization metadata.

### Storage policy

- Keep activity records on the device.
- Use parameterized SQL statements.
- Validate stored values.
- Handle database migrations.
- Explain that clearing app data or uninstalling the app can remove locally stored records.
- Do not treat temporary cache directories as durable storage.

## 6\. Application Architecture

Separate presentation, business logic, storage, and native integrations.

Suggested responsibilities:

- `app/`: Screens and navigation.
- `components/`: Reusable UI components.
- `features/steps/`: Step state and tracking logic.
- `features/history/`: Historical summaries.
- `services/health/`: Health Connect integration.
- `services/sensors/`: Optional native sensor integration.
- `services/database/`: SQLite initialization, queries, and migrations.
- `services/storage/`: Preferences.
- `hooks/`: Reusable application hooks.
- `types/`: Shared TypeScript types.
- `constants/`: Theme and configuration.

Screens must not access the native health APIs or execute SQL directly. They should call typed services or feature hooks.

## 7\. UI and UX

Design direction: a modern fitness dashboard with a light background, green accent color, readable typography, rounded cards, and restrained animations.

Primary navigation:

1. Home
2. History
3. Settings

The home screen should prioritize the step total and goal progress. Provide loading, permission-denied, unavailable-source, empty-history, and error states.

Follow Android accessibility conventions, support scalable text, and ensure adequate contrast and touch-target sizes.

## 8\. Development Setup

### Prerequisites

- Node.js LTS compatible with the selected Expo SDK.
- npm.
- Android Studio and Android SDK.
- An Android emulator or physical Android device.
- An Expo account for EAS cloud builds.

### Create the application

```
npx create-expo-app@latest stride --template tabs
cd stride
```

Install NativeWind and its required Tailwind CSS dependencies according to the compatibility instructions for the selected Expo SDK.

Install supported Expo modules using `npx expo install` wherever applicable. Add SQLite and AsyncStorage for persistence.

Choose and install a compatible Health Connect integration before implementing native health access.

### Start development

```
npx expo start
```

For native health or sensor modules, create a development build instead of relying on Expo Go.

## 9\. Android APK Build

Configure the EAS build profile for an installable APK.

Example `eas.json`:

```
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "android": {
        "buildType": "app-bundle"
      }
    }
  }
}
```

Install and configure EAS CLI, authenticate, and initialize the EAS project before building.

```
npm install -g eas-cli
eas login
eas build:configure
eas build --platform android --profile preview
```

Download the generated APK from the EAS build page and install it on a compatible Android device.

The preview profile is intended for direct APK distribution. The production profile above produces an Android App Bundle for Play Store distribution.

## 10\. Testing Strategy

### Unit tests

- Goal progress calculations.
- Distance estimation.
- Date-boundary handling.
- Database upserts.
- Duplicate prevention.
- Input validation.

### Integration tests

- SQLite persistence after app restart.
- Health permission handling.
- Empty and unavailable data sources.
- Daily synchronization.
- Source overlap handling.
- Database migration behavior.

### Device tests

- Supported Android versions.
- Devices with and without a step sensor.
- Permission denial and revocation.
- Backgrounding and reopening the app.
- Reboot and daily date transitions.
- Airplane mode and offline usage.
- APK installation and upgrade.

Use physical-device tests to validate sensor behavior. Emulator-generated activity is not proof that real-world step tracking works.

## 11\. Implementation Roadmap

### Phase 1 — Foundation

- Create the Expo project.
- Configure TypeScript and NativeWind.
- Set up navigation and design tokens.
- Create the initial dashboard.

### Phase 2 — Persistence

- Implement SQLite schema and migrations.
- Add preferences storage.
- Build daily activity repositories.
- Test persistence and duplicate prevention.

### Phase 3 — Step integration

- Integrate Health Connect.
- Implement permissions and availability checks.
- Add synchronization and date handling.
- Add the optional native sensor path if required.

### Phase 4 — History and polish

- Build weekly and monthly summaries.
- Add settings and goal editing.
- Improve loading, error, and empty states.
- Add accessibility and performance checks.

### Phase 5 — Verification and release

- Test on physical Android devices.
- Validate background and daily synchronization behavior.
- Generate an APK using EAS.
- Install and verify the release build.
- Document installation and troubleshooting.

## 12\. Definition of Done

The first release is complete when:

- The application launches on a supported Android device.
- Actual step data is displayed when the configured source provides it.
- Permission and unsupported-device states work correctly.
- Daily goals can be changed.
- Activity history persists locally.
- Repeated synchronization does not duplicate step totals.
- The interface remains usable offline.
- No backend account is required.
- A signed, installable APK has been generated and tested.

## 13\. Scope and Limitations

- Health Connect availability and supported records depend on the Android device, operating system, provider, and granted permissions.
- Direct background sensor tracking may require additional native configuration and device-specific validation.
- Distance and active-minute values are estimates or optional data unless a reliable source supplies them.
- Local-only data is not automatically backed up or synchronized across devices.
- Battery consumption and background execution must be tested on real devices.

## 14\. Engineering Principles

- TypeScript strict mode.
- Clear separation of concerns.
- Small, testable modules.
- Minimal native permissions.
- No fake health measurements.
- No unnecessary network dependencies.
- Accessible UI components.
- Reproducible APK builds.
- Explicit error handling and data-source transparency.

---

**Project status:** Specification drafted. Implementation, device verification, and APK release are pending.

## Recommended project structure

This is the structure I recommend implementing after the README is finalized.

stride/

app/

Home, History, Settings and navigation

components/

Progress ring, cards, buttons and shared UI

features/steps/

Step state, goal calculations and synchronization

features/history/

Daily and weekly activity summaries

services/health/

Health Connect adapter and permissions

services/sensors/

Optional native step sensor adapter

services/database/

SQLite schema, migrations and repositories

services/storage/

Local preferences

hooks/

Reusable React hooks

types/

Shared TypeScript models

constants/

Theme, colors and configuration

README.md

Setup and engineering documentation

app.json / eas.json

Expo and Android build configuration

## Key engineering decisions before coding

- Persistent storage, not cache-only: Temporary cache files can be evicted by Android. SQLite and AsyncStorage are better choices for data that should survive app restarts.
- Real step data: Health Connect is the initial data source; direct sensor integration is an optional extension. We'll avoid double counting.
- Expo compatibility: We'll use Expo's managed workflow where possible, with a development build and native configuration for integrations that require it.
- APK delivery: We'll configure EAS to generate a directly installable APK and test that exact build on an Android device.

## Next step

The README is our initial specification. The next deliverable should be the actual Expo project foundation: dependency setup, NativeWind configuration, navigation, SQLite initialization, and a first working dashboard.

One choice will affect the implementation significantly: should the app count steps directly from the phone's sensor, or read steps from Android Health Connect?

My recommendation is to start with Health Connect, then add direct sensor tracking if you need independent step counting without relying on another health-data provider.
