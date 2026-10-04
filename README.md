# MaestroStarter

Automated login tests using Maestro, organized by platform.

```text
test/user_login/
|-- web/      Implemented browser login flows
|-- android/  Android app flows
`-- ios/      iOS app flows
```

Flow web yang tersedia saat ini menguji situs. Pengujian aplikasi native Android/iOS juga memerlukan aplikasi yang bisa dipasang dan flow Maestro yang sesuai.

## Instalasi

### Prasyarat

- macOS dan terminal VS Code
- Node.js dan npm
- Maestro CLI, tersedia di `PATH`
- Koneksi internet untuk mengunduh komponen SDK atau Chromium saat pertama kali digunakan
- Xcode diperlukan hanya untuk iOS Simulator

### Pasang dependency proyek

Dari direktori proyek, jalankan:

```bash
npm install
```

Proyek menyertakan dependency npm `chromedriver`. Flow web Maestro menggunakan Chromium yang dikelola dan diunduh Maestro; Google Chrome atau Chromium tidak perlu dipasang manual.

## Android

### Menyiapkan emulator tanpa Android Studio

Android SDK Command-line Tools dan VS Code cukup untuk mengunduh komponen SDK, membuat emulator, dan menjalankannya. Android Studio tidak diperlukan. Langkah berikut menggunakan Apple Silicon (misalnya Mac M1/M2/M3); untuk Mac Intel, ganti system image `arm64-v8a` dengan `x86_64`.

#### 1. Pasang Java dan Android Command-line Tools

Pasang JDK 17 atau lebih baru, lalu pastikan Java tersedia:

```bash
java -version
```

Unduh **Command line tools only** untuk macOS dari [halaman unduhan Android Studio](https://developer.android.com/studio#command-line-tools-only). Buat direktori SDK:

```bash
mkdir -p "$HOME/Android/Sdk/cmdline-tools/latest"
```

Ekstrak ZIP yang diunduh. Pindahkan **isi** folder `cmdline-tools` hasil ekstraksi (termasuk `bin`, `lib`, dan file lainnya) ke `"$HOME/Android/Sdk/cmdline-tools/latest"`. Pastikan file berikut tersedia:

```text
~/Android/Sdk/cmdline-tools/latest/bin/sdkmanager
```

#### 2. Atur environment di terminal VS Code

Tambahkan konfigurasi berikut ke `~/.zshrc`:

```bash
export ANDROID_HOME="$HOME/Android/Sdk"
export PATH="$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$PATH"
```

Terapkan perubahan dan verifikasi:

```bash
source ~/.zshrc
sdkmanager --version
```

Jika terminal VS Code masih tidak mengenali perintah, tutup dan buka kembali terminal tersebut.

#### 3. Pasang komponen SDK dan system image

Terima lisensi SDK, lalu pasang emulator, ADB, platform Android, dan system image Google APIs ARM64:

```bash
sdkmanager --licenses
sdkmanager "platform-tools" "emulator" "platforms;android-35" "system-images;android-35;google_apis;arm64-v8a"
```

#### 4. Buat dan jalankan virtual device

Pastikan profil Pixel 8 tersedia, lalu buat AVD bernama `MyEmu`:

```bash
avdmanager list device
avdmanager create avd -n MyEmu -k "system-images;android-35;google_apis;arm64-v8a" --device "pixel_8"
```

Jalankan emulator dari terminal VS Code:

```bash
emulator -avd MyEmu
```

Setelah emulator selesai boot, pastikan ADB mendeteksinya dengan status `device`:

```bash
adb devices
```

Untuk membuka emulator tanpa menjalankan YAML, cukup jalankan `emulator -avd MyEmu`. Extension VS Code seperti **Android iOS Emulator** bisa dipakai sebagai alternatif untuk memilih dan membuka AVD, tetapi tidak wajib.

### Menggunakan ulang langkah membuka situs

Flow Android memakai subflow `test/user_login/android/open_saucedemo.yaml` untuk membuka Chrome dan menavigasi ke SauceDemo. Panggil subflow tersebut dari flow lain dengan:

```yaml
- runFlow: "open_saucedemo.yaml"
```

Karena file pemanggil dan subflow berada di direktori yang sama, langkah pembukaan browser tidak perlu disalin ke setiap flow.

### Menjalankan test Android

Isi variabel `PROD_URL`, `VALID_USERNAME`, dan `UNIVERSAL_PASSWORD` pada file `.env` seperti dijelaskan di bagian [konfigurasi web](#konfigurasi). Pastikan emulator sedang berjalan, lalu jalankan seluruh enam kasus login Android:

```bash
npm run maestro-android
```

Suite Android menjalankan kasus yang sama dengan suite web: kredensial valid, username salah, password salah, username kosong, password kosong, dan kedua field kosong.

### Mengetahui `appId` Android

Buka aplikasi yang ingin diuji secara manual di emulator, lalu jalankan:

```bash
adb shell dumpsys window | grep -E 'mCurrentFocus|mFocusedApp'
```

Output biasanya memuat nama paket dan activity, misalnya `com.example.app/.MainActivity`. Bagian sebelum `/` (`com.example.app`) adalah `appId`. Jika aplikasi aktif tidak muncul, coba:

```bash
adb shell dumpsys activity activities | grep mResumedActivity
```

Nama paket aplikasi pihak ketiga yang terpasang dapat dilihat dengan `adb shell pm list packages -3`. Nama pada ikon aplikasi belum tentu sama dengan `appId`.

Jika aplikasi sudah dibuka manual, flow dapat langsung menjalankan langkah seperti `tapOn` tanpa `appId` dan tanpa `launchApp`. Maestro akan berinteraksi dengan aplikasi yang sedang tampil. Untuk memakai `launchApp`, flow perlu mengetahui aplikasi yang hendak dibuka.

## Web (Chromium)

Flow web di repository menggunakan konfigurasi URL Maestro (`url: ${PROD_URL}`). Maestro mengunduh dan mengelola Chromium sendiri pada eksekusi pertama, sehingga tidak perlu memasang emulator, Google Chrome, atau Chromium secara terpisah.

### Konfigurasi

Buat file `.env` di direktori proyek:

```env
PROD_URL=https://your-test-site.example
VALID_USERNAME=your_valid_username
UNIVERSAL_PASSWORD=your_test_password
```

Jaga kerahasiaan `.env`; file ini dikecualikan dari Git.

### Menjalankan flow web

Pastikan Maestro CLI tersedia:

```bash
maestro --version
```

Jalankan seluruh suite dari terminal VS Code:

```bash
npm run maestro
```

Script npm memuat `.env` lalu menjalankan `test/user_login/web/user_login.yaml`, yang memanggil flow berikut secara berurutan:

1. `01.login_valid_credential.yaml` - kredensial valid
2. `02.login_invalid_username.yaml` - username tidak dikenal
3. `03.login_invalid_password.yaml` - password salah
4. `04.login_empty_username.yaml` - username kosong
5. `05.login_empty_password.yaml` - password kosong
6. `06.login_empty_field.yaml` - kedua field kosong

Koneksi internet diperlukan saat pertama kali Maestro mengunduh Chromium. Flow order ditentukan oleh entri `runFlow` di orchestrator, bukan urutan nama file.

### Menambahkan flow web

1. Tambahkan file YAML bernomor di `test/user_login/web/`.
2. Tambahkan entri `runFlow` di `test/user_login/web/user_login.yaml` pada posisi yang diinginkan.
3. Jalankan `npm run maestro`.

Contoh:

```yaml
- runFlow: "07.login_locked_user.yaml"
```

## iOS

iOS Simulator membutuhkan Xcode. Android Studio tidak diperlukan, tetapi **Command Line Tools saja tidak cukup** untuk memasang atau menjalankan iOS Simulator.

### Menyiapkan Xcode dan iOS Simulator

Pasang Xcode dari Mac App Store, buka Xcode sekali untuk menyelesaikan setup awal dan menerima lisensi. Pastikan command-line tools menunjuk ke instalasi Xcode:

```bash
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
xcodebuild -version
xcrun simctl list devices available
```

Jika `xcode-select --install` diperlukan untuk tools tambahan, jalankan perintah tersebut. Perintah itu sendiri tidak memasang aplikasi Xcode atau iOS Simulator.

Di Xcode, buka **Settings > Platforms** (atau **Components**, tergantung versi Xcode) dan unduh runtime iOS Simulator. Untuk membuka simulator:

```bash
open -a Simulator
```

Untuk menjalankan device tertentu, lihat nama yang tersedia dengan `xcrun simctl list devices available`, lalu gunakan nama tersebut, misalnya:

```bash
xcrun simctl boot "iPhone 16"
open -a Simulator
```

Nama model bergantung pada runtime dan versi Xcode. Untuk menguji aplikasi native, build aplikasi untuk **iOS Simulator**, lalu pasang file `.app` hasil build ke simulator yang sedang boot:

```bash
xcrun simctl install booted /path/to/YourApp.app
```

### Mengetahui `appId` iOS

`appId` iOS adalah **Bundle Identifier** aplikasi. Di Xcode, temukan nilainya di **Project navigator > pilih project > pilih target aplikasi > General > Identity > Bundle Identifier**. Jika sudah memiliki file `.app` hasil build, Bundle Identifier dapat dibaca dari terminal:

```bash
defaults read /path/to/YourApp.app/Info CFBundleIdentifier
```

Jika aplikasi sudah dibuka manual di iOS Simulator, flow dapat langsung menjalankan langkah seperti `tapOn` tanpa `appId` dan tanpa `launchApp`. Maestro akan berinteraksi dengan aplikasi yang sedang tampil. Untuk menggunakan `launchApp` atau meluncurkan aplikasi tertentu, gunakan Bundle Identifier sebagai `appId`.
