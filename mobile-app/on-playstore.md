# How to deploy Mobile Application on PlayStore ?

Deploying an Android application on Google Play requires signing your app with a digital certificate known as a keystore. This certificate ensures the authenticity and integrity of your application. This guide walks you through the process of generating a keystore, creating a key.properties file to securely manage your keystore information, and configuring your project to use these credentials for signing your app.

## Prerequisites

Before proceeding with the deployment process, ensure you have the following:

- Android Application: Have a completed Android application ready for deployment.
- Java Development Kit (JDK): Ensure JDK is installed on your development machine.
- Android Studio: Have Android Studio installed for building and signing your application.

## Step 1: Generate Key and Create `key.properties` File

1. Generate Keystore: Open a terminal or command prompt and run the following command to generate a keystore:

```sh
keytool -genkeypair -v -keystore your-key-name.keystore -alias your-alias-name -keyalg RSA -keysize 2048 -validity 10000
```

Replace `your-key-name.keystore` with your desired keystore file name and `your-alias-name` with your preferred alias. Follow the prompts to enter the required information.

2. Create key.properties File: After generating the keystore, create a key.properties file in the root directory of your Android project (next to build.gradle files).

Add the following lines to key.properties and fill in the values:

```ini
keyAlias=your-alias-name
keyPassword=your-key-password
storeFile=/path/to/your/keystore/your-key-name.keystore
storePassword=your-keystore-password
```

- keyAlias: Alias name you used when generating the keystore.
- keyPassword: Password for the alias in the keystore.
- storeFile: Path to the keystore file.
- storePassword: Password for the keystore file itself.

Make sure to `replace your-alias-name`, `your-key-password`, `your-key-name`.`keystore`, and `your-keystore-password` with your actual values.

## Step 2: Configure build.gradle

In your `app/build.gradle` file, add the following code snippet to read the `key.properties` file and use it for signing your APK during the build process:

```groovy
android {
  ...
  // Load keystore properties from key.properties file
  def keystoreProperties = new Properties()
  def keystorePropertiesFile = rootProject.file('key.properties')
  if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
  }

  signingConfigs {
    release {
      keyAlias keystoreProperties['keyAlias']
      keyPassword keystoreProperties['keyPassword']
      storeFile file(keystoreProperties['storeFile'])
      storePassword keystoreProperties['storePassword']
    }
  }

  buildTypes {
    release {
      signingConfig signingConfigs.release
      ...
    }
  }
}
```

## Step 3: Secure `key.properties` and Ensure It's Ignored in Git

1. Add `key.properties` to `.gitignore`: Open or create a .gitignore file in the root of your project if it doesn't exist.

Add the line:

```
key.properties
```

This ensures that key.properties is not tracked by Git and thus not exposed in your version control system.

2. Secure `key.properties`: Since `key.properties` contains sensitive information (passwords), make sure to keep it secure locally. Avoid sharing or committing key.properties to repositories or sharing it insecurely.

## Step 4: Generating APK or AAB

1. Flutter

```sh
flutter build apk --release # For APK
flutter build appbundle --release # For AAB
```

Output Location: APK (app-release.apk) or AAB (app-release.aab) is located in build/app/outputs directory.

2. React Native:

Navigate to your React Native project's android directory and to generate APK runs.

```sh
./gradlew assembleRelease
```

Output Location: APK (app-release.apk) is found in android/app/build/outputs/apk/release.

AAB generation is currently not supported directly via React Native CLI. You would typically use Android Studio to generate an AAB.

3. Kotlin (Native Android):

Navigate to your Kotlin-based Android project directory and run ./gradlew assembleRelease.

```sh
./gradlew assembleRelease # for APK
./gradlew bundleRelease # For AAB
```

Output Location: APK (app-release.apk) or AAB (app-release.aab) is located in app/build/outputs/apk/release or app/build/outputs/bundle/release, respectively.
