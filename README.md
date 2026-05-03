# 🔒 VPN Blocker — Android App

## Cấu trúc Project

```
VPNBlocker/
├── app/
│   ├── build.gradle
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── kotlin/com/vpnblocker/
│       │   ├── MainActivity.kt          ← UI + xin quyền VPN
│       │   └── BlockerVpnService.kt     ← Core VPN logic
│       └── res/
│           ├── layout/activity_main.xml
│           └── values/{colors,strings,themes}.xml
├── build.gradle
├── settings.gradle
└── gradle.properties
```

---

## 🧠 Cách hoạt động VPNService

### Kiến trúc tổng quan

```
┌─────────────────────────────────────────────────────────┐
│                      DEVICE                              │
│                                                          │
│  App (Chrome, YouTube...)                                │
│       │ gửi packet TCP/UDP                               │
│       ▼                                                  │
│  ┌─────────────────┐                                     │
│  │  TUN Interface  │  ← ảo, do VpnService.Builder tạo   │
│  │  (10.0.0.2/32)  │    Android route TẤT CẢ traffic vào│
│  └────────┬────────┘                                     │
│           │  FileInputStream.read()                      │
│           ▼                                              │
│  ┌─────────────────────────────┐                         │
│  │    BlockerVpnService        │                         │
│  │                             │                         │
│  │  if (blockOutgoing) DROP ──►│── packet bị xóa         │
│  │  else FORWARD ─────────────►│── gửi qua protect()     │
│  │                             │   socket ra internet    │
│  │  Response từ internet       │                         │
│  │  if (blockIncoming) DROP ──►│── không ghi vào TUN     │
│  │  else ghi vào TUN ─────────►│── App nhận được data    │
│  └─────────────────────────────┘                         │
└─────────────────────────────────────────────────────────┘
```

### Chi tiết từng bước

#### 1. `VpnService.prepare(context)`
- Kiểm tra app đã có quyền VPN chưa
- Nếu chưa → trả về Intent để hiển thị dialog hệ thống
- Android chỉ cho 1 VPN app hoạt động cùng lúc

#### 2. `VpnService.Builder.establish()`
- Tạo TUN interface ảo
- `addAddress("10.0.0.2", 32)` → IP ảo của thiết bị
- `addRoute("0.0.0.0", 0)` → route ALL traffic vào tunnel
- `addDisallowedApplication(packageName)` → ứng dụng VPN bypass chính nó (tránh loop)
- Trả về `ParcelFileDescriptor` — file handle đọc/ghi packet

#### 3. Đọc packet (Outgoing)
```kotlin
val input = FileInputStream(vpnInterface.fileDescriptor)
val length = input.read(buffer)
// buffer chứa raw IP packet (IPv4 hoặc IPv6)
```
- Mỗi lần `read()` → 1 gói tin hoàn chỉnh (IP header + payload)
- Có thể parse header để lấy src/dst IP, port, protocol

#### 4. Block Outgoing
```kotlin
if (blockOutgoing) continue  // DROP: không gửi đi
```
- Đơn giản là không làm gì → gói tin bị "nuốt"
- App sẽ timeout hoặc nhận lỗi connection refused

#### 5. Block Incoming  
```kotlin
if (!blockIncoming) {
    output.write(packet)  // Chỉ ghi nếu KHÔNG block
}
```
- Response từ server KHÔNG được ghi vào TUN
- App không nhận được data → request treo/timeout

#### 6. `protect(socket)` — bypass tunnel
```kotlin
val socket = DatagramSocket()
protect(socket)  // socket này đi thẳng ra internet, bypass TUN
```
- Cần thiết khi muốn FORWARD thay vì block
- Tránh vòng lặp: packet gửi ra → vào TUN lại → gửi ra lại...

---

## 🔑 Quyền hạn cần khai báo

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

<service
    android:name=".BlockerVpnService"
    android:permission="android.permission.BIND_VPN_SERVICE">
    <intent-filter>
        <action android:name="android.net.VpnService" />
    </intent-filter>
</service>
```

- `BIND_VPN_SERVICE`: Chỉ system mới bind được service này (bảo mật)
- `intent-filter android.net.VpnService`: Android nhận diện đây là VPN service hợp lệ

---

## 🔨 Hướng dẫn Build APK

### Cách 1: Android Studio (khuyến nghị)

#### Yêu cầu
- Android Studio Hedgehog (2023.1.1) trở lên
- JDK 17
- Android SDK API 34

#### Các bước

```bash
# 1. Clone/copy project vào máy
# 2. Mở Android Studio → File → Open → chọn thư mục VPNBlocker/

# 3. Đợi Gradle sync hoàn tất (~2-5 phút lần đầu)

# 4. Build APK debug:
#    Build → Build Bundle(s) / APK(s) → Build APK(s)

# 5. APK output tại:
#    app/build/outputs/apk/debug/app-debug.apk
```

#### Build Release APK (có ký)

```
Build → Generate Signed Bundle / APK
→ APK
→ Create new keystore (lần đầu) hoặc chọn keystore có sẵn
→ Key alias, password
→ Build Type: release
→ Finish
```

Output: `app/build/outputs/apk/release/app-release.apk`

---

### Cách 2: Command Line (Gradle)

```bash
# macOS / Linux
cd VPNBlocker
./gradlew assembleDebug

# Windows
cd VPNBlocker
gradlew.bat assembleDebug

# APK tại: app/build/outputs/apk/debug/app-debug.apk
```

Build release:
```bash
./gradlew assembleRelease
# Cần cấu hình signing trong app/build.gradle trước
```

---

### Cách 3: Cài trực tiếp lên thiết bị (ADB)

```bash
# Bật Developer Options trên điện thoại:
# Settings → About Phone → tap "Build Number" 7 lần
# Settings → Developer Options → USB Debugging = ON

# Kết nối USB, sau đó:
adb install app/build/outputs/apk/debug/app-debug.apk

# Hoặc build + install cùng lúc (Android Studio):
# Run → Run 'app' (Shift+F10)
```

---

## ⚠️ Lưu ý quan trọng

| Vấn đề | Giải thích |
|--------|-----------|
| **Chỉ 1 VPN cùng lúc** | Android không cho 2 VPN service chạy song song |
| **Cần xác nhận người dùng** | Lần đầu chạy, Android hỏi "Bạn tin tưởng app này không?" |
| **Foreground Service** | API 26+ bắt buộc VPN phải có notification thường trực |
| **Google Play Store** | Ứng dụng VPN phải được Google duyệt riêng (VPN policy) |
| **Không phải firewall thực** | Block ở tầng app-layer, không phải kernel; iptables mạnh hơn nhưng cần root |
| **Battery impact** | Packet processing liên tục tốn pin; nên thêm sleep nếu traffic thấp |

---

## 🚀 Mở rộng thêm

```kotlin
// Chặn theo app cụ thể
builder.addDisallowedApplication("com.facebook.katana")

// Chặn theo IP
builder.addRoute("192.168.1.0", 24)  // chặn subnet

// Chặn theo domain → cần DNS intercept
// Parse DNS query (port 53), lookup domain, trả về NXDOMAIN

// Forward thực sự (không chỉ block)
val socket = DatagramSocket()
protect(socket)
socket.connect(InetAddress.getByName(dstIp), dstPort)
socket.send(DatagramPacket(payload, payload.size))
```
