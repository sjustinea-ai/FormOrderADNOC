name: Build APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Extract Android project
        run: |
          mkdir -p android_project
          ZIP_FILE=$(find . -maxdepth 1 -type f -iname "*.zip" | head -n 1)

          if [ -z "$ZIP_FILE" ]; then
            echo "ZIP Android project tidak ditemukan"
            exit 1
          fi

          unzip -q "$ZIP_FILE" -d android_project

      - name: Find Android project
        run: |
          PROJECT_DIR=$(find android_project -type f \( -name "settings.gradle" -o -name "settings.gradle.kts" \) -exec dirname {} \; | head -n 1)

          if [ -z "$PROJECT_DIR" ]; then
            echo "Project Android tidak ditemukan"
            echo "Isi ZIP:"
            find android_project -maxdepth 4 -type f | head -100
            exit 1
          fi

          echo "PROJECT_DIR=$PROJECT_DIR" >> $GITHUB_ENV
          echo "Project ditemukan di: $PROJECT_DIR"

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'

      - name: Build APK
        run: |
          cd "$PROJECT_DIR"
          gradle assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: FormOrderADNOC-APK
          path: |
            **/build/outputs/apk/debug/*.apk
