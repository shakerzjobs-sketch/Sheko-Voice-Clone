# Sheko Voice Clone - APK Build Guide

## Prerequisites

Before building the APK, ensure you have the following installed:

- **Android Studio** (Latest version)
- **Java Development Kit (JDK)** 11 or higher
- **Android SDK** (API 30+)
- **Gradle** (Usually bundled with Android Studio)
- **Git** (for version control)

## Project Structure

```
Sheko-Ready-To-Upload/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/          # Java source files
│   │   │   ├── res/           # Resources (layouts, drawables, values)
│   │   │   │   ├── layout/    # XML layout files
│   │   │   │   ├── raw/       # Raw audio files (reference_voice.wav)
│   │   │   │   └── values/    # String and configuration files
│   │   │   └── AndroidManifest.xml
│   │   ├── test/              # Unit tests
│   │   └── androidTest/       # Instrumented tests
│   ├── build.gradle           # App-level build configuration
│   └── proguard-rules.pro     # ProGuard obfuscation rules
├── build.gradle               # Project-level build configuration
├── settings.gradle            # Gradle settings
└── local.properties           # Local SDK path (create this)
```

## Step 1: Extract the ZIP File

```bash
# Navigate to where the zip file is located
cd /path/to/Sheko-Ready-To-Upload.zip

# Extract the zip file
unzip "Sheko-Ready-To-Upload (1).zip" -d sheko-project
cd sheko-project
```

## Step 2: Configure Local Properties

Create a `local.properties` file in the project root:

```properties
sdk.dir=/path/to/Android/Sdk
```

On different systems:
- **Windows**: `C:\\Users\\YourUsername\\AppData\\Local\\Android\\Sdk`
- **macOS**: `/Users/YourUsername/Library/Android/sdk`
- **Linux**: `/home/YourUsername/Android/Sdk`

## Step 3: Build Variants

### Debug Build (For Testing)

```bash
# Using Gradle wrapper
./gradlew assembleDebug

# Or using Android Studio
# Build > Build Bundle(s) / APK(s) > Build APK(s)
```

Output: `app/build/outputs/apk/debug/app-debug.apk`

### Release Build (For Production)

Before building release, you need a signing key:

```bash
# Create a keystore (do this once)
keytool -genkey -v -keystore my-release-key.keystore \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias my-key-alias

# Build release APK
./gradlew assembleRelease
```

Output: `app/build/outputs/apk/release/app-release.apk`

## Step 4: Build Using Android Studio GUI

1. Open Android Studio
2. File → Open → Select the project folder
3. Wait for Gradle sync to complete
4. Build → Build Bundle(s) / APK(s) → Build APK(s)
5. The APK will be generated in `app/build/outputs/apk/`

## Step 5: Build Features

### Check Build Status
```bash
./gradlew build --info
```

### Clean Build
```bash
./gradlew clean build
```

### Build with Specific Module
```bash
./gradlew :app:assembleDebug
```

### Generate APK with Signing
```bash
./gradlew bundleRelease -Pandroid.injected.signing.store.file=/path/to/keystore \
  -Pandroid.injected.signing.store.password=your_password \
  -Pandroid.injected.signing.key.alias=your_alias \
  -Pandroid.injected.signing.key.password=key_password
```

## Step 6: Install on Device/Emulator

```bash
# List connected devices
adb devices

# Install debug APK
adb install app/build/outputs/apk/debug/app-debug.apk

# Install and run
adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell am start -n com.example.sheko/.MainActivity
```

## Step 7: Testing

### Run Unit Tests
```bash
./gradlew test
```

### Run Instrumented Tests
```bash
./gradlew connectedAndroidTest
```

### Check Code Quality
```bash
./gradlew lint
```

## Troubleshooting

### Issue: Gradle Sync Failed
**Solution:**
```bash
./gradlew clean
./gradlew sync
```

### Issue: SDK Not Found
**Solution:** Ensure `local.properties` has correct SDK path
```bash
echo "sdk.dir=/path/to/Android/Sdk" > local.properties
```

### Issue: Build Cache Issues
**Solution:**
```bash
./gradlew clean
rm -rf .gradle
./gradlew build
```

### Issue: Java Compatibility
**Solution:** Update Java compatibility in `build.gradle`:
```gradle
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_11
        targetCompatibility JavaVersion.VERSION_11
    }
}
```

### Issue: Memory Error During Build
**Solution:** Increase Gradle heap size:
```properties
# In gradle.properties
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m
```

## Build Configuration

### Key Configuration Files

**build.gradle (Project)**
```gradle
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.x.x'
    }
}
```

**build.gradle (App)**
```gradle
android {
    compileSdkVersion 34
    
    defaultConfig {
        applicationId "com.example.sheko"
        minSdkVersion 24
        targetSdkVersion 34
        versionCode 1
        versionName "1.0"
    }
    
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

## Performance Optimization

### Optimize Build Time
```bash
# Enable parallel build
./gradlew assembleDebug --parallel

# Use build cache
./gradlew assembleDebug --build-cache

# Offline mode (if dependencies cached)
./gradlew assembleDebug --offline
```

### Reduce APK Size
```bash
# Generate bundle instead of APK (smaller)
./gradlew bundleRelease

# Enable ProGuard/R8 minification in release builds
```

## Continuous Integration Setup

### GitHub Actions Example
Create `.github/workflows/build.yml`:
```yaml
name: Build APK
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: '11'
      - run: ./gradlew assembleDebug
      - uses: actions/upload-artifact@v3
        with:
          name: app-debug.apk
          path: app/build/outputs/apk/debug/app-debug.apk
```

## APK Output Locations

- **Debug APK**: `app/build/outputs/apk/debug/app-debug.apk`
- **Release APK**: `app/build/outputs/apk/release/app-release.apk`
- **Bundle**: `app/build/outputs/bundle/release/app-release.aab`
- **Mapping File**: `app/build/outputs/mapping/release/mapping.txt`

## Additional Resources

- [Android Developers - Build APK](https://developer.android.com/studio/build/building-cmdline)
- [Gradle Documentation](https://gradle.org/docs/)
- [Android Studio Guide](https://developer.android.com/studio/intro)

## Quick Start Commands

```bash
# Complete setup and build
./gradlew clean build

# Build and install on device
./gradlew installDebug

# Build release signed APK
./gradlew assembleRelease

# Check build configuration
./gradlew projects

# View all available tasks
./gradlew tasks
```

---

**Last Updated**: 2024
**Status**: Ready for Build
