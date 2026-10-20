name: Build tvhome APK

on:
  workflow_dispatch:
  push:
    paths:
      - '.github/workflows/build.yml'
      - 'tvhome-main.zip'

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'

      - name: Setup Android SDK
        uses: android-actions/setup-android@v4

      - name: Extract project
        run: |
          unzip -q tvhome-main.zip
          cd tvhome-main
          printf 'sdk.dir=%s\n' "$ANDROID_HOME" > local.properties
          chmod +x gradlew

      - name: Build APK
        run: |
          cd tvhome-main
          ./gradlew assembleDebug --stacktrace

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: tvhome-debug-apk
          path: tvhome-main/**/build/outputs/apk/debug/*.apk
          if-no-files-found: error
