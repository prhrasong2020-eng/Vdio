
name: Build Android APK

on:
  workflow_dispatch:
  push:
    paths:
      - "*.zip"
      - ".github/workflows/build-apk.yml"

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Find and extract starter ZIP
        shell: bash
        run: |
          ZIP_FILE="$(find . -maxdepth 1 -type f -iname '*.zip' -print -quit)"
          if [ -z "$ZIP_FILE" ]; then
            echo "ไม่พบไฟล์ ZIP ที่โฟลเดอร์หลักของ Repository"
            exit 1
          fi
          mkdir -p /tmp/starter
          unzip -q "$ZIP_FILE" -d /tmp/starter
          APP_DIR="$(find /tmp/starter -type f -path '*/android_app/pubspec.yaml' -print -quit | xargs -r dirname)"
          if [ -z "$APP_DIR" ]; then
            echo "ไม่พบโปรเจกต์ Flutter ใน ZIP"
            exit 1
          fi
          mkdir -p app
          cp -R "$APP_DIR"/. app/

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable

      - name: Generate Android project files
        working-directory: app
        run: flutter create . --platforms=android --project-name=ai_short_film_studio

      - name: Install packages
        working-directory: app
        run: flutter pub get

      - name: Build release APK
        working-directory: app
        run: flutter build apk --release

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: AI-Short-Film-Studio-APK
          path: app/build/app/outputs/flutter-apk/app-release.apk
