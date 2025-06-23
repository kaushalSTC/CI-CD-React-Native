
# 📱 React Native Android CI/CD Setup with GitHub Actions and Fastlane

## 🚀 Project Initialization

```bash
npx @react-native-community/cli@latest init YourAppName --package-name=com.orgination.appname

cd YourAppName
git init
git remote add origin yourrepourl.git
mkdir -p .github/workflows
touch .github/workflows/android-ci.yml
```

---

## 🔐 Generate Keystore File

```sh
keytool -genkeypair -v -keystore <release>.keystore -alias <androidreleasekey> -keyalg RSA -keysize 2048 -validity 10000
```

---

## 🔁 Convert Keystore to Base64

```sh
base64 -i <release>.keystore -o keystore_base64.txt
```

---

## 🔧 Add GitHub Secrets

Go to your GitHub repository:

> Settings → Secrets and variables → Actions → New repository secret

### Required Secrets

| Secret Name              | Description                              |
|--------------------------|------------------------------------------|
| `ANDROID_KEYSTORE_BASE64` | Base64 string of your `.keystore` file   |
| `KEY_ALIAS`              | Your key alias                           |
| `KEYSTORE_PASSWORD`      | Your keystore password                   |
| `KEY_PASSWORD`           | Your key password (usually same)         |

---

## 🛠️ Android CI Workflow (`android-ci.yml`)

```yml
name: Android CI Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    name: Build Release APK and AAB
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'

    - name: Set up Java
      uses: actions/setup-java@v3
      with:
        distribution: 'temurin'
        java-version: '17'

    - name: Install Yarn
      run: npm install -g yarn

    - name: Install dependencies
      run: yarn install

    - name: Decode keystore from secret
      run: |
        echo "${{ secrets.ANDROID_KEYSTORE_BASE64 }}" | base64 -d > android/app/my-release-key.keystore

    - name: Set up keystore properties
      run: |
        echo "MYAPP_UPLOAD_STORE_FILE=my-release-key.keystore" >> android/gradle.properties
        echo "MYAPP_UPLOAD_KEY_ALIAS=${{ secrets.KEY_ALIAS }}" >> android/gradle.properties
        echo "MYAPP_UPLOAD_STORE_PASSWORD=${{ secrets.KEYSTORE_PASSWORD }}" >> android/gradle.properties
        echo "MYAPP_UPLOAD_KEY_PASSWORD=${{ secrets.KEY_PASSWORD }}" >> android/gradle.properties

    - name: Build Release APK
      run: cd android && ./gradlew assembleRelease

    - name: Build Release AAB (Bundle)
      run: cd android && ./gradlew bundleRelease

    - name: Upload Release APK
      uses: actions/upload-artifact@v4
      with:
        name: app-release.apk
        path: android/app/build/outputs/apk/release/app-release.apk

    - name: Upload Release AAB
      uses: actions/upload-artifact@v4
      with:
        name: app-release.aab
        path: android/app/build/outputs/bundle/release/app-release.aab
```

---

## 🚀 Fastlane Integration

### 📦 Install Fastlane and Setup

```sh
brew install fastlane
```

### 📁 Setup Fastlane in Android Directory

```sh
cd android
fastlane init
```

If you encounter Ruby version issues:

```sh
brew update
brew install rbenv ruby-build
rbenv global 3.2.0
```

Add to `~/.zshrc`:

```sh
if command -v rbenv > /dev/null; then
  eval "$(rbenv init -)"
fi
```

```sh
source ~/.zshrc
gem install bundler
bundle install
```

### ✅ Build with Fastlane

Run inside the `android` directory:

```sh
bundle exec fastlane android build_release
```

If using on GitHub Actions:

```sh
bundle lock --add-platform x86_64-linux
```
