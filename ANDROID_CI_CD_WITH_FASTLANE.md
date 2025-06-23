# 📱 React Native Android CI/CD Setup with GitHub Actions & Fastlane

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

## 🔧 Add GitHub Secrets (Only For assemble and bundle release)

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

## 🚀 Fastlane Integration Setup

```sh
brew install fastlane
```

```sh
cd android
fastlane init
```

```sh
brew update
brew install rbenv ruby-build
rbenv global 3.2.0
nano ~/.zshrc
```

Add this to your `~/.zshrc`:

```sh
if command -v rbenv > /dev/null; then
  eval "$(rbenv init -)"
fi
```

Then run:

```sh
source ~/.zshrc
gem install bundler
bundle install
```

To build the release with Fastlane (run inside the `android` directory):

```sh
bundle exec fastlane android build_release
```

If needed, update your bundler platform with:

```sh
bundle lock --add-platform x86_64-linux
```

---

## 🛠️ Full Android CI Workflow (`android-ci.yml`)

```yml
name: Android CI Build All

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    name: Build Debug, Release APK, and AAB
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

    - name: Build Debug APK
      run: cd android && ./gradlew assembleDebug

    - name: Build Release APK
      run: cd android && ./gradlew assembleRelease

    - name: Build Release AAB (Bundle)
      run: cd android && ./gradlew bundleRelease

    - name: Upload Debug APK
      uses: actions/upload-artifact@v4
      with:
        name: app-debug.apk
        path: android/app/build/outputs/apk/debug/app-debug.apk

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

## 🧪 Debug-Only Workflow

```yml
name: Android Debug Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    name: Build Debug APK
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

    - name: Build Debug APK
      run: cd android && ./gradlew assembleDebug

    - name: Upload Debug APK
      uses: actions/upload-artifact@v4
      with:
        name: app-debug.apk
        path: android/app/build/outputs/apk/debug/app-debug.apk
```

---

## 📦 Release APK Only Workflow

```yml
name: Android Release APK

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    name: Build Release APK
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

    - name: Upload Release APK
      uses: actions/upload-artifact@v4
      with:
        name: app-release.apk
        path: android/app/build/outputs/apk/release/app-release.apk
```

---

## 📦 Bundle AAB Only Workflow

```yml
name: Android Release AAB

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    name: Build Release AAB
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

    - name: Build Release AAB (Bundle)
      run: cd android && ./gradlew bundleRelease

    - name: Upload Release AAB
      uses: actions/upload-artifact@v4
      with:
        name: app-release.aab
        path: android/app/build/outputs/bundle/release/app-release.aab
```
