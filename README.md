Stride — Cross-Device Step Tracker

Stride is an offline-first fitness application designed for Android phones and future smartwatch support.

The phone application uses React Native, Expo, TypeScript, and NativeWind. A compatible smartwatch will run an independent native tracking application capable of recording and storing steps without a phone connection.

Project Vision

Build a privacy-focused activity-tracking system with:

Daily step counts and configurable goals.
Local activity history and progress charts.
Independent phone and smartwatch tracking.
Reliable synchronization after connectivity is restored.
Local-first storage with no mandatory account or cloud backend.
Installable Android APK and a separate watch installation package where supported.
Platform Strategy
Phone: React Native + Expo + TypeScript + NativeWind.
Phone storage: SQLite for durable records; AsyncStorage for preferences.
Phone health integration: Android Health Connect, where supported.
Smartwatch: Platform-specific native application selected after evaluating compatible devices.
Wear OS option: Kotlin + Jetpack Compose + supported wearable health APIs.
Synchronization: A versioned, device-to-device protocol with stable record identifiers, retry handling, and duplicate prevention.

The phone and watch must remain independently functional. Neither device should require an active connection to the other to record and save its own activity.

Extended Development Roadmap
Establish the Android application foundation.
Implement and test real step-data integration.
Add local storage, goals, and activity history.
Define shared domain models and synchronization contracts.
Select smartwatch hardware and implement independent tracking.
Implement offline-first synchronization and source reconciliation.
Perform device, battery, permission, and recovery testing.
Generate release builds and document installation.
Cross-Device Data Requirements

Every activity record must identify its source, device, time interval, and stable record ID.

Synchronization must be idempotent, preserve locally saved records, and recover after disconnection.

The application must not blindly add phone and watch totals because their measurements can overlap.

Daily totals must follow a documented source-of-truth and reconciliation policy.

Release Targets
Phone MVP
Verified real step data.
Configurable daily goals.
Persistent activity history.
Offline operation.
Installable Android APK.
Phone and Watch Release
Independent step recording on the selected watch.
Local persistence on both devices.
Reliable synchronization and conflict handling.
Verified behavior when the phone is unavailable.
Tested phone and watch installation packages.

Current status: Architecture and roadmap defined. Implementation and hardware validation remain pending.
