name: Dr Maseer Baryal Safi Official App

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "17"

      - name: Install Android SDK
        run: |
          SDKMANAGER="$(find "$ANDROID_HOME/cmdline-tools" -type f -name sdkmanager | head -n 1)"
          yes | "$SDKMANAGER" --licenses || true
          "$SDKMANAGER" --install "platform-tools" "platforms;android-35" "build-tools;35.0.0"

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: "8.9"

      - name: Extract Nangarhar project
        run: |
          mkdir -p extracted
          unzip -q Nangarhar.zip -d extracted

      - name: Find Android project
        shell: bash
        run: |
          FILE=$(find extracted -type f \( -name settings.gradle.kts -o -name settings.gradle \) -print -quit)
          if [ -z "$FILE" ]; then
            echo "Gradle project not found"
            exit 1
          fi
          echo "PROJECT_DIR=$(dirname "$FILE")" >> "$GITHUB_ENV"

      - name: Align Java and Kotlin targets
        run: |
          python3 - <<'PY'
          from pathlib import Path
          import re

          for p in list(Path("extracted").rglob("build.gradle")) + list(Path("extracted").rglob("build.gradle.kts")):
              s = p.read_text()
              s = s.replace("VERSION_1_8", "VERSION_17")
              s = s.replace("JVM_1_8", "JVM_17")
              s = re.sub(r'((?:sourceCompatibility|targetCompatibility|jvmTarget)\s*=\s*["\'])1\.8(["\'])', r'\g<1>17\2', s)
              p.write_text(s)
          PY

      - name: Build APK
        run: |
          cd "$PROJECT_DIR"
          if [ -f gradlew ]; then
            chmod +x gradlew
            ./gradlew assembleDebug
          else
            gradle assembleDebug
          fi

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Dr-Maseer-Baryal-Safi-Official-App
          path: extracted/**/build/outputs/apk/**/*.apk
          if-no-files-found: error
