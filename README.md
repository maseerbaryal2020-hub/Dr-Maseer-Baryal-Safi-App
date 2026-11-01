name: Build Nangarhar APK

on:
  workflow_dispatch:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v5

      - name: Extract ZIP
        run: unzip -o Nangarhar.zip -d extracted

      - name: Setup Java
        uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: '17'

      - name: Install Android SDK
        run: |
          SDK="$ANDROID_HOME"
          SDKMANAGER=$(find "$SDK/cmdline-tools" -name sdkmanager | head -1)
          yes | "$SDKMANAGER" --licenses || true
          "$SDKMANAGER" "platform-tools" "platforms;android-35" "build-tools;35.0.0"

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.9'

      - name: Find Project
        id: project
        run: |
          FILE=$(find extracted -name settings.gradle.kts -o -name settings.gradle | head -1)
          if [ -z "$FILE" ]; then
            echo "Gradle project not found"
            exit 1
          fi
          echo "dir=$(dirname "$FILE")" >> "$GITHUB_OUTPUT"

      - name: Build APK
        run: |
          cd "${{ steps.project.outputs.dir }}"
          gradle assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Nangarhar-APK
          path: extracted/**/build/outputs/apk/**/*.apk
          if-no-files-found: error
