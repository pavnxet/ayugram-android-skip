# Jumpcutter Android (ExoPlayer Skip Silence) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking. This plan specifically **DOES NOT** commit changes; files are modified locally for Github Actions to run.

**Goal:** Integrate Jumpcutter-like silence skipping functionality into the Telegram Android video player and provide a GitHub Action to directly build the APK cleanly.

**Architecture:** We will expose `ExoPlayer`'s built-in `setSkipSilenceEnabled` toggle through `VideoPlayer.java`, add a UI toggle to `PhotoViewer.java` (the media player UI). Finally, we'll configure a GitHub Actions CI file `.github/workflows/build-android.yml` that overrides the API keys locally and compiles the build without needing a manual dev-environment map setup.

**Tech Stack:** Java, ExoPlayer, Android, GitHub Actions

---

### Task 1: Expose `skipSilence` toggle in `VideoPlayer`

**Files:**
- Modify: `TMessagesProj/src/main/java/org/telegram/ui/Components/VideoPlayer.java`

- [ ] **Step 1: Write explicit boolean enabler**
Inside `VideoPlayer.java` class, add:
```java
    public void setSkipSilenceEnabled(boolean skipSilenceEnabled) {
        if (player != null) {
            player.setSkipSilenceEnabled(skipSilenceEnabled);
        }
        if (audioPlayer != null) {
            audioPlayer.setSkipSilenceEnabled(skipSilenceEnabled);
        }
    }
```
*(No commits will be made)*

---

### Task 2: Add UI Toggle to PhotoViewer/Speed Menu

**Files:**
- Modify: `TMessagesProj/src/main/java/org/telegram/ui/PhotoViewer.java`

- [ ] **Step 1: Add Skip Silence Option**
Locate the speed/playback settings in `PhotoViewer.java`. Add a toggle for "Skip Silence".
- [ ] **Step 2: Route the Option to the VideoPlayer**
When selected, set `videoPlayer.setSkipSilenceEnabled(enabled)`.
*(No commits will be made)*

---

### Task 3: Create GitHub Actions Workflow for Android Build

**Files:**
- Create: `.github/workflows/build-android.yml`

- [ ] **Step 1: Write Action file**
Create `.github/workflows/build-android.yml` with the following content:

```yaml
name: Build Android APK

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-22.04

    steps:
    - uses: actions/checkout@v4
      with:
        submodules: recursive

    - name: Set up JDK 11
      uses: actions/setup-java@v4
      with:
        java-version: '11'
        distribution: 'temurin'
        cache: gradle

    - name: Setup dummy API keys
      run: |
        echo "APP_ID=123" > API_KEYS
        echo "APP_HASH=\"dummy\"" >> API_KEYS
        echo "MAPS_V2_API=\"dummy\"" >> API_KEYS
        echo "SIGNING_KEY_STORE_PASSWORD=dummy" >> API_KEYS
        echo "SIGNING_KEY_ALIAS=dummy" >> API_KEYS
        echo "SIGNING_KEY_PASSWORD=dummy" >> API_KEYS

    - name: Disable signing config manually (for CI Debug Build)
      run: |
        sed -i 's/signingConfig signingConfigs.release//g' TMessagesProj/build.gradle

    - name: Grant execute permissions
      run: chmod +x gradlew

    - name: Build Beta Release APK (Unsigned)
      run: ./gradlew assembleBetaRelease --stacktrace

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: ayugram-android-unsigned-apk
        path: TMessagesProj/build/outputs/apk/beta/release/*.apk
```
