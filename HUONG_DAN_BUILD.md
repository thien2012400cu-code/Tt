# 🚀 Hướng dẫn Build APK bằng GitHub Actions (không cần Android Studio)

## Bước 1 — Tạo repo GitHub

1. Vào https://github.com/new
2. Đặt tên repo: `VPNBlocker`
3. Chọn **Private** (nếu không muốn public)
4. Bấm **Create repository**

---

## Bước 2 — Upload code lên GitHub

### Cách A: Dùng giao diện web (dễ nhất, không cần cài gì)

Tạo từng file bằng cách bấm **"Add file" → "Create new file"** trên GitHub:

```
Cấu trúc cần tạo:
├── .github/
│   └── workflows/
│       └── build.yml          ← copy nội dung từ file build.yml
├── app/
│   ├── build.gradle           ← copy từ app/build.gradle
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── kotlin/com/vpnblocker/
│       │   ├── BlockerVpnService.kt
│       │   └── MainActivity.kt
│       └── res/
│           ├── layout/activity_main.xml
│           └── values/
│               ├── colors.xml
│               ├── strings.xml
│               └── themes.xml
├── build.gradle
├── gradle.properties
└── settings.gradle
```

**Để tạo file trong thư mục:**
- Gõ tên file có đường dẫn luôn, ví dụ: `.github/workflows/build.yml`
- GitHub tự tạo folder

---

### Cách B: Dùng GitHub Desktop (kéo thả)

1. Tải GitHub Desktop: https://desktop.github.com
2. Clone repo vừa tạo về máy
3. Copy toàn bộ file từ ZIP vào thư mục repo
4. Bấm **Commit** → **Push**

---

### Cách C: Git CLI

```bash
cd VPNBlocker
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/TEN_BAN/VPNBlocker.git
git push -u origin main
```

---

## Bước 3 — Thêm file Gradle Wrapper (QUAN TRỌNG)

GitHub Actions cần file `gradlew`. Tạo 2 file này trên GitHub:

### File: `gradle/wrapper/gradle-wrapper.properties`
```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.2-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

### File: `gradlew` (script shell)
```bash
#!/bin/sh
# Gradle start up script for UN*X
CLASSPATH=$APP_HOME/gradle/wrapper/gradle-wrapper.jar

# ...
# Tải file đầy đủ tại:
# https://raw.githubusercontent.com/gradle/gradle/v8.2.1/gradlew
```

**Cách nhanh nhất:** Vào link sau, copy toàn bộ nội dung, tạo file `gradlew` trên GitHub:
```
https://raw.githubusercontent.com/nicokruger/random-stuff/master/gradlew
```

Sau đó cần thêm `gradle-wrapper.jar` — file binary, không thể tạo tay.

---

## ✅ Cách đơn giản nhất: Dùng template có sẵn

Thay vì tạo từ đầu, fork một Android project mẫu đã có gradle wrapper:

1. Vào: https://github.com/android/nowinandroid (hoặc bất kỳ Android repo nào)
2. Bấm **Fork**
3. Xóa code cũ, thêm code VPNBlocker vào
4. Giữ nguyên `gradlew`, `gradle/wrapper/gradle-wrapper.jar`

---

## Bước 4 — Chạy GitHub Actions

Sau khi push code:

1. Vào tab **Actions** của repo
2. Chọn workflow **"Build APK"**
3. Bấm **"Run workflow"** → **"Run workflow"**
4. Đợi ~3-5 phút

---

## Bước 5 — Tải APK

Sau khi build xong (✅ xanh):

1. Bấm vào run vừa chạy
2. Kéo xuống phần **Artifacts**
3. Bấm **"VPNBlocker-debug"** → tải về ZIP
4. Giải nén → có file `app-debug.apk`

---

## Cài APK lên điện thoại

1. Chép file `.apk` vào điện thoại (USB / Google Drive / Telegram)
2. Vào **Settings → Security → Unknown sources** → Bật
3. Mở file manager → tìm file `.apk` → cài đặt

---

## ⚡ Nếu muốn nhanh hơn: Dùng Replit

1. Vào https://replit.com
2. Tạo project loại **Android**
3. Copy code vào
4. Bấm Run — Replit có thể build APK online

Hoặc **Gitpod**: https://gitpod.io — mở VS Code trên browser, có thể build luôn.
