name: Build Nangarhar APK

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v5

      - name: Extract ZIP
        run: unzip Nangarhar.zip -d extracted

      - name: Setup Java
        uses: actions/setup-java@v5
        with:
          distribution: temurin
          java-version: '17'

      - name: Setup Android
        uses: android-actions/setup-android@v3
        with:
          packages: |
            platforms;android-35
            build-tools;35.0.0

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.9'

      - name: Build APK
        run: |
          PROJECT_DIR=$(dirname "$(find extracted -name settings.gradle.kts -print -quit)")
          cd "$PROJECT_DIR"
          gradle assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Nangarhar-APK
          path: extracted/**/app/build/outputs/apk/debug/*.apk
