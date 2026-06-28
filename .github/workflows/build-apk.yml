name: Build APK

on:
  push:
    branches: [ main, master ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Estrai il progetto dallo zip
        run: |
          set -e
          ZIP=$(ls *.zip | head -n1)
          echo "Uso lo zip: $ZIP"
          mkdir -p src
          unzip -o "$ZIP" -d src >/dev/null
          if [ ! -f src/settings.gradle ]; then
            inner=$(find src -maxdepth 4 -name settings.gradle | head -n1)
            dir=$(dirname "$inner")
            echo "Progetto trovato in: $dir"
            shopt -s dotglob
            mv "$dir"/* src/ 2>/dev/null || true
          fi
          ls -la src

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Set up Android SDK
        uses: android-actions/setup-android@v3

      - name: Install SDK packages
        run: yes | sdkmanager "platforms;android-34" "build-tools;34.0.0" "platform-tools" >/dev/null || true

      - name: Compila l'APK
        working-directory: src
        env:
          KEYSTORE_BASE64: ${{ secrets.KEYSTORE_BASE64 }}
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
        run: |
          set -e
          : "${KEYSTORE_PASSWORD:=farmaci-store-pw}"
          : "${KEY_ALIAS:=farmaci}"
          : "${KEY_PASSWORD:=farmaci-key-pw}"
          if [ -n "$KEYSTORE_BASE64" ]; then
            echo "$KEYSTORE_BASE64" | base64 -d > release.keystore
          else
            keytool -genkeypair -v -keystore release.keystore -alias "$KEY_ALIAS" \
              -keyalg RSA -keysize 2048 -validity 10000 \
              -storepass "$KEYSTORE_PASSWORD" -keypass "$KEY_PASSWORD" \
              -dname "CN=Gestione Farmaci,O=Personale,C=IT"
          fi
          export KEYSTORE_FILE="$PWD/release.keystore"
          export KEYSTORE_PASSWORD KEY_ALIAS KEY_PASSWORD
          gradle wrapper --gradle-version 8.7
          ./gradlew assembleRelease --no-daemon --stacktrace

      - name: Raccogli l'APK
        run: |
          mkdir -p out
          cp src/app/build/outputs/apk/release/*.apk out/GestioneFarmaci.apk

      - name: Carica l'APK
        uses: actions/upload-artifact@v4
        with:
          name: GestioneFarmaci-APK
          path: out/GestioneFarmaci.apk
          if-no-files-found: error
