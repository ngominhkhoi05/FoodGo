# FoodGo — Hệ thống Giao Đồ Ăn Trực Tuyến

> Hệ thống giao đồ ăn trực tuyến đa nền tảng với 4 vai trò người dùng: Khách hàng, Tài xế, Người bán, Quản trị viên. Backend bằng Spring Boot + Firebase Firestore, Frontend bằng Flutter.

---

## Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Kiến trúc hệ thống](#2-kiến-trúc-hệ-thống)
3. [Công nghệ sử dụng](#3-công-nghệ-sử-dụng)
4. [Phiên bản](#4-phiên-bản)
5. [Cấu trúc dự án](#5-cấu-trúc-dự-án)
6. [Cài đặt Backend](#6-cài-đặt-backend)
7. [Cài đặt Frontend](#7-cài-đặt-frontend)
8. [Tài khoản test](#8-tài-khoản-test)
9. [Cấu trúc API](#9-cấu-trúc-api)
10. [Các lưu ý quan trọng](#10-các-lưu-ý-quan-trọng)

---

## 1. Tổng quan

FoodGo là hệ thống giao đồ ăn trực tuyến được xây dựng với kiến trúc microservices (Frontend), bao gồm:

| Module | Mô tả |
|---|---|
| `be-foodgo` | Backend REST API (Spring Boot + Firebase Firestore) |
| `fe_foodgo_customers` | Ứng dụng Khách hàng (Flutter) — đặt món, theo dõi đơn, quản lý tài khoản |
| `fe_foodgo_admin` | Ứng dụng Quản trị viên (Flutter) — cấu hình hệ thống, đối soát tài chính |
| `fe_food_go_portal` | Ứng dụng Người bán/Merchant (Flutter) — quản lý cửa hàng, thực đơn, doanh thu |
| `fe_food_go_driver` | Ứng dụng Tài xế (Flutter) — nhận đơn, giao hàng, GPS tracking real-time |

### Vai trò người dùng

| Vai trò | Mã | Mô tả |
|---|---|---|
| Khách hàng | 1 | Tìm kiếm quán, đặt hàng, theo dõi đơn, đánh giá |
| Tài xế | 2 | Nhận đơn, giao hàng, GPS tracking real-time |
| Người bán | 3 | Quản lý cửa hàng, thực đơn, xử lý đơn hàng |
| Quản trị viên | 4 | Cấu hình hệ thống, đối soát tài chính |

---

## 2. Kiến trúc hệ thống

```
                        ┌─────────────────────┐
                        │   Firebase Cloud    │
                        │  Firestore, RTDB,   │
                        │  Storage, FCM, Auth│
                        └──────────┬──────────┘
                                   │
           ┌───────────────────────┼───────────────────────┐
           │                       │                       │
    ┌──────▼──────┐         ┌──────▼──────┐         ┌──────▼──────┐
    │    BE       │         │    FE       │         │    FE       │
    │  Spring     │◄────────│  Customer   │         │  Admin      │
    │  Boot API   │  REST   │  (Flutter)  │         │  (Flutter)  │
    │  :8086      │         └─────────────┘         └─────────────┘
    │             │◄───────────────────────────────────────────┐
    │  WebSocket  │         ┌─────────────┐         ┌──────────▼──┐
    │  (STOMP)   │◄────────│  FE Driver  │         │  FE Portal  │
    │             │  WS     │  (Flutter)  │         │  (Flutter)  │
    └─────────────┘         └─────────────┘         └─────────────┘
```

---

## 3. Công nghệ sử dụng

### 3.1. Backend (`be-foodgo`)

| Thành phần | Công nghệ |
|---|---|
| Ngôn ngữ | Java 21 |
| Framework | Spring Boot 4.0.6 |
| Database | Firebase Firestore |
| Realtime Database | Firebase Realtime Database |
| Authentication | Firebase Admin SDK + JWT |
| Storage | Firebase Cloud Storage, Cloudinary |
| Real-time | Spring WebSocket (STOMP) |
| Push Notification | Firebase Cloud Messaging (FCM) |
| API Documentation | SpringDoc OpenAPI |
| Build tool | Maven |
| Email | Spring Boot Mail (Gmail SMTP) |

### 3.2. Frontend (tất cả ứng dụng Flutter)

| Thành phần | Công nghệ |
|---|---|
| Framework | Flutter |
| Ngôn ngữ | Dart |
| State Management | flutter_bloc (Driver), StatefulWidget (Customers/Admin/Portal) |
| Database | Firebase Firestore, Firebase Realtime Database |
| Authentication | Firebase Authentication |
| Maps | flutter_map (OpenStreetMap) |
| Location | Geolocator |
| Real-time Communication | STOMP (WebSocket) |
| Push Notification | Firebase Cloud Messaging (FCM) |
| Dependency Injection | get_it (Driver) |

---

## 4. Phiên bản

### Backend

| Thành phần | Phiên bản |
|---|---|
| Java | 21 |
| Spring Boot | 4.0.6 |
| Maven | 3.6+ |
| Firebase Admin SDK | 9.2.0 |
| google-cloud-storage | 2.47.0 |
| JJWT | 0.12.6 |
| SpringDoc OpenAPI | 2.8.4 |

### Frontend (tất cả 4 app)

| Thành phần | Phiên bản |
|---|---|
| Flutter SDK | ^3.11.1 |
| Dart SDK | ^3.11.1 |
| Android Gradle Plugin | 8.x |
| Kotlin | 1.9.x |
| iOS Deployment Target | 12.0+ |

---

## 5. Cấu trúc dự án

```
FoodGo/
│
├── be-foodgo/                     # Backend Spring Boot
│   ├── src/main/java/com/example/be_foodgo/
│   │   ├── config/               # Firebase, Security, CORS, WebSocket
│   │   ├── controller/           # REST API endpoints
│   │   ├── dto/                  # Data Transfer Object
│   │   ├── model/               # Firestore document mapping
│   │   ├── repository/           # Truy vấn Firestore
│   │   ├── seeder/              # Khởi tạo dữ liệu mẫu
│   │   ├── service/             # Logic nghiệp vụ
│   │   ├── exception/           # Xử lý lỗi tập trung
│   │   ├── constant/            # Enum và hằng số
│   │   └── websocket/           # Cấu hình WebSocket
│   ├── src/main/resources/
│   │   ├── application.properties
│   │   └── firebase-service-account.json
│   └── pom.xml
│
├── fe_foodgo_customers/           # Ứng dụng Khách hàng (Flutter)
│   ├── lib/
│   │   ├── core/                 # Network, Utils, Localization
│   │   └── features/            # Auth, Home, Store, Cart, Order,
│   │                            #   Profile, Payment, Rewards, ...
│   ├── android/
│   ├── ios/
│   └── pubspec.yaml
│
├── fe_foodgo_admin/               # Ứng dụng Quản trị viên (Flutter)
│   ├── lib/
│   │   └── data/services/        # API constants, network
│   ├── android/
│   ├── ios/
│   └── pubspec.yaml
│
├── fe_food_go_portal/             # Ứng dụng Người bán/Merchant (Flutter)
│   ├── lib/
│   ├── android/
│   ├── ios/
│   └── pubspec.yaml
│
├── fe_food_go_driver/             # Ứng dụng Tài xế (Flutter)
│   ├── lib/
│   │   ├── core/                 # Cấu hình chung, API, constants, theme
│   │   └── features/             # Auth, Home, Orders, Profile
│   ├── android/
│   ├── ios/
│   └── pubspec.yaml
│
├── google-services.json           # Firebase config cho Android (tất cả app)
├── firestore.rules                # Firestore Security Rules
├── firebase-service-account.json  # Firebase Admin SDK service account
├── .firebaserc                   # Firebase project config
└── README.md                     # Tài liệu tổng hợp dự án
```

---

## 6. Cài đặt Backend

### 6.1. Yêu cầu

- **JDK 21** trở lên
- **Maven 3.6+** (hoặc dùng Maven wrapper đi kèm)
- **Tài khoản Firebase** với Firestore, Realtime Database, Storage
- **firebase-service-account.json** (lấy từ Firebase Console)

### 6.2. Cấu hình Firebase

1. Truy cập [Firebase Console](https://console.firebase.google.com/)
2. Tạo project mới hoặc chọn project hiện có
3. Kích hoạt: **Firestore Database**, **Realtime Database**, **Firebase Storage**
4. Vào **Project Settings → Service accounts → Generate new private key**
5. Đổi tên file JSON thành `firebase-service-account.json`
6. Đặt file vào `src/main/resources/`

> **Lưu ý bảo mật:** File `firebase-service-account.json` chứa credentials nhạy cảm. **Không được commit lên Git.** File này đã được thêm vào `.gitignore`.

### 6.3. Cấu hình `application.properties`

File `src/main/resources/application.properties` chứa các cấu hình chính:

```properties
spring.application.name=be-foodgo
server.port=8086

# Firebase
firebase.database.url=https://food-go-17a5d-default-rtdb.asia-southeast1.firebasedatabase.app/
firebase.storage.bucket=food-go-17a5d.appspot.com

# JWT
jwt.secret=FoodGoJwtSecretKey2026Nam3Ki2VeryLongAndSecure256BitSecretKeyForSigningTokens
jwt.expiration=10800000

# Email (Gmail SMTP)
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=foldershop83@gmail.com
spring.mail.password=<app-password>
app.email.enabled=true

# Cloudinary
cloudinary.cloud_name=dd51afnue
cloudinary.api_key=867855984214359
cloudinary.api_secret=<api-secret>

# Upload
spring.servlet.multipart.max-file-size=5MB
spring.servlet.multipart.max-request-size=10MB
```

> **Lưu ý:** Để gửi email thực, cần tạo **App Password** từ tài khoản Google:
> 1. Vào [Google Account](https://myaccount.google.com) → **Security** → **2-Step Verification** → **App passwords**
> 2. Tạo app password mới và thay vào `spring.mail.password`

### 6.4. Chạy Backend

```bash
# macOS / Linux
./mvnw spring-boot:run

# Windows
mvnw.cmd spring-boot:run

# Hoặc build JAR rồi chạy
./mvnw clean package -DskipTests
java -jar target/be-foodgo-0.0.1-SNAPSHOT.jar
```

Backend chạy tại: `http://localhost:8086`

Swagger UI (API Documentation): `http://localhost:8086/swagger-ui.html`

### 6.4. Firebase Configuration Details

Dự án sử dụng chung một Firebase project (`food-go-17a5d`) cho cả Backend và tất cả ứng dụng Frontend.

#### 6.4.1. Firebase Project Info

| Thông tin | Giá trị |
|---|---|
| Project ID | `food-go-17a5d` |
| Project Number | `628420378856` |
| Firestore URL | `https://food-go-17a5d-default-rtdb.asia-southeast1.firebasedatabase.app` |
| Storage Bucket | `food-go-17a5d.firebasestorage.app` |

#### 6.4.2. Các ứng dụng Android đã đăng ký

| Package Name | App ID |
|---|---|
| `com.example.fe_food_go_driver` | `1:628420378856:android:af12af71853f7011fcc95d` |
| `com.example.fe_food_go_portal` | `1:628420378856:android:bf74e9666cae29b1fcc95d` |
| `com.example.fe_foodgo_customers` | `1:628420378856:android:53ef385ebabaccfcfcc95d` |

#### 6.4.3. Firestore Security Rules

Xem file [`firestore.rules`](./firestore.rules) tại thư mục gốc dự án. Các quy tắc chính:

| Collection | Quyền đọc | Quyền ghi | Ghi chú |
|---|---|---|---|
| `users/*` | Công khai | Công khai | Thông tin người dùng |
| `customer_profiles/*` | Công khai | Công khai | Profile, địa chỉ, giỏ hàng, voucher |
| `orders/*` | Công khai | Công khai | Đơn hàng |
| `driver_profiles/*` | Công khai | Công khai | Profile tài xế |
| `merchant_profiles/*` | Công khai | Công khai | Profile người bán |
| `stores/*` | Công khai | **Chỉ backend** | Thông tin cửa hàng (backend quản lý) |
| `products` (trong stores) | Công khai | **Chỉ backend** | Sản phẩm (backend quản lý) |
| `categories/*` | Công khai | **Chỉ backend** | Danh mục |
| `vouchers/*` | Công khai | **Chỉ backend** | Voucher |
| `banners/*` | Công khai | **Chỉ backend** | Banner quảng cáo |
| `order_requests/{driverId}` | Chỉ driver sở hữu | **Chỉ backend** | Đơn giao hàng cho tài xế |

> **Lưu ý bảo mật:** Hiện tại hệ thống sử dụng **backend API login (JWT)** thay vì Firebase Auth trực tiếp, nên các collection chính đều cho phép read/write công khai. Trong môi trường production, nên bổ sung Firebase Admin SDK để xác thực JWT từ backend và giới hạn quyền truy cập Firestore theo người dùng.

#### 6.4.4. Service Account (Backend)

Backend sử dụng service account để kết nối Firebase Admin SDK:

```json
{
  "type": "service_account",
  "project_id": "food-go-17a5d",
  "client_email": "firebase-adminsdk-fbsvc@food-go-17a5d.iam.gserviceaccount.com",
  ...
}
```

- File: `src/main/resources/firebase-service-account.json`
- **Không commit file này lên Git.** Nếu mất file, vào **Firebase Console → Project Settings → Service accounts → Generate new private key**.

---

## 7. Cài đặt Frontend

### 7.1. Yêu cầu hệ thống chung

- **Flutter SDK:** >= 3.11.1
- **Dart SDK:** >= 3.11.1
- **Android SDK** (nếu build Android)
- **Xcode + CocoaPods** (nếu build iOS, chỉ trên macOS)
- **Git**

### 7.2. Cài đặt chung cho tất cả ứng dụng Flutter

**1. Cài đặt dependencies:**

```bash
cd <ten-thu-muc-ung-dung>
flutter pub get
```

**2. Cấu hình Firebase:**

**2. Cấu hình Firebase (`google-services.json`):**

File `google-services.json` đã có sẵn tại thư mục gốc của dự án, chứa cấu hình cho cả 3 ứng dụng Android. Mỗi ứng dụng Flutter cần đặt bản sao tương ứng vào thư mục `android/app/` của nó:

| Ứng dụng | Package Name | Vị trí đặt file |
|---|---|---|
| Customers | `com.example.fe_foodgo_customers` | `fe_foodgo_customers/android/app/google-services.json` |
| Admin | *(chưa đăng ký riêng)* | Cấu hình Firebase thông qua `firebase_options.dart` |
| Portal | `com.example.fe_food_go_portal` | `fe_food_go_portal/android/app/google-services.json` |
| Driver | `com.example.fe_food_go_driver` | `fe_food_go_driver/android/app/google-services.json` |

- Lấy file từ Firebase Console → Project Settings → Your apps → Android app
- iOS: tải `GoogleService-Info.plist` → đặt vào `ios/Runner/GoogleService-Info.plist` của từng app

Kích hoạt dịch vụ trong Firebase Console:
- **Authentication** → Sign-in method → bật **Email/Password**
- **Firestore Database** → Create database
- **Realtime Database** → Create database
- **Storage** → Create storage
- **Cloud Messaging** → Lấy Server key để gửi push notification

**3. Cấu hình Android:**

`android/build.gradle` (project-level):
```groovy
plugins {
    id 'com.android.application' version '8.1.0' apply false
    id 'com.google.gms.google-services' version '4.4.2' apply false
}
```

`android/app/build.gradle` (app-level):
```groovy
plugins {
    id 'com.google.gms.google-services'
}

defaultConfig {
    minSdkVersion 21
}
```

**4. Thêm quyền Android:**

`android/app/src/main/AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
```

**5. Cấu hình iOS:**

`ios/Runner/Info.plist`:
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>App có thể xác định vị trí của bạn để nhận đơn giao hàng</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>App có thể xác định vị trí khi chạy nền để cập nhật vị trí liên tục</string>
<key>UIBackgroundModes</key>
<array>
    <string>location</string>
    <string>remote-notification</string>
</array>
```

```bash
cd ios && pod install --repo-update && cd ..
```

**6. Chạy ứng dụng:**

```bash
# Chế độ debug
flutter run

# Build APK debug
flutter build apk --debug

# Build APK release
flutter build apk --release

# Build iOS (chỉ macOS)
flutter build ios
```

### 7.3. Cấu hình API URL cho từng app

#### `fe_foodgo_customers` (`lib/core/network/api_client.dart`)

```dart
static const String _baseUrl = 'http://<dia-chi-IP>:8086/api';
```

#### `fe_foodgo_admin` (`lib/data/services/api_constants.dart`)

```dart
static const String baseUrl = 'http://<dia-chi-IP>:8086/api';
```

#### `fe_food_go_portal` (`lib/data/services/api_constants.dart`)

```dart
static const String baseUrl = 'https://be-foodgo.canluaz.io.vn/api';
```

#### `fe_food_go_driver` (`lib/core/api/api_constants.dart` hoặc `lib/injection_container.dart`)

> Cấu hình địa chỉ server Backend trong file `api_constants.dart` hoặc `injection_container.dart` tương ứng.

### 7.4. Lưu ý theo nền tảng chạy

| Nền tảng | baseURL |
|---|---|
| Android Emulator | `http://10.0.2.2:8086/api` |
| iOS Simulator | `http://localhost:8086/api` |
| Thiết bị thật | `http://<IPv4-may-tinh>:8086/api` |
| Web | `https://be-foodgo.canluaz.io.vn/api` |

> **Lưu ý:** Điện thoại và máy tính server **phải kết nối cùng một mạng Wi-Fi** và cùng lớp mạng (cùng subnet). Lấy địa chỉ IP trên Windows bằng lệnh `ipconfig` → tìm **IPv4 Address** của adapter Wi-Fi.

### 7.5. Chạy Web Portal với CORS

```bash
flutter run -d chrome --web-browser-flag "--disable-web-security"
```

### 7.6. Danh sách Package / Dependency

#### 7.6.1. Backend (`be-foodgo` / `pom.xml`)

| Package | Phiên bản | Mục đích |
|---|---|---|
| spring-boot-starter-webmvc | (parent) | REST API |
| spring-boot-starter-security | (parent) | Security |
| spring-boot-starter-websocket | (parent) | WebSocket |
| spring-boot-starter-validation | (parent) | Validation |
| spring-boot-starter-mail | (parent) | Email |
| firebase-admin | 9.2.0 | Firebase SDK |
| google-cloud-storage | 2.47.0 | Cloud Storage |
| jjwt-api / jjwt-impl / jjwt-jackson | 0.12.6 | JWT authentication |
| springdoc-openapi-starter-webmvc-ui | 2.8.4 | Swagger UI |
| cloudinary-http5 | 2.3.2 | Cloudinary image hosting |
| commons-csv | 1.10.0 | CSV export |
| lombok | (parent) | Giảm boilerplate |

#### 7.6.2. `fe_foodgo_customers` / `fe_foodgo_admin` / `fe_food_go_portal` (`pubspec.yaml`)

| Package | Phiên bản | Mục đích |
|---|---|---|
| flutter_bloc | ^8.1.6 | State management |
| fl_chart | ^0.66.0 / ^0.69.2 | Biểu đồ thống kê |
| intl | ^0.19.0 / ^0.20.2 | Định dạng ngày tháng |
| firebase_core | ^4.6.0 / ^4.9.0 | Firebase core |
| firebase_auth | ^6.3.0 / ^6.5.1 | Xác thực |
| cloud_firestore | ^6.2.0 | Cloud Firestore |
| dio | ^5.4.0 / ^5.9.2 | HTTP client |
| shared_preferences | ^2.3.3 / ^2.5.5 | Lưu trữ cục bộ |
| image_picker | ^1.1.2 / ^1.2.2 | Chọn hình |
| file_picker | ^8.1.2 | Chọn file |
| firebase_storage | ^13.4.2 | Firebase Storage (portal) |
| flutter_map | ^7.0.2 / ^8.3.0 | Bản đồ |
| latlong2 | ^0.9.1 | Tọa độ địa lý |
| geolocator | ^13.0.2 / ^14.0.2 | Lấy vị trí |
| easy_localization | ^3.0.7 | Đa ngôn ngữ (portal) |

#### 7.6.3. `fe_food_go_driver` (`pubspec.yaml`)

| Package | Phiên bản | Mục đích |
|---|---|---|
| flutter_bloc | ^8.1.6 | State management |
| equatable | ^2.0.5 | So sánh đối tượng |
| dartz | ^0.10.1 | Functional programming |
| rxdart | ^0.28.0 | Reactive extensions |
| get_it | ^8.0.3 | Dependency injection |
| shared_preferences | ^2.3.5 | Lưu trữ cục bộ |
| flutter_secure_storage | ^9.2.4 | Lưu trữ an toàn |
| intl | ^0.20.2 | Định dạng ngày tháng |
| http | ^1.2.0 | HTTP client |
| stomp_dart_client | ^3.0.1 | WebSocket STOMP |
| geolocator | ^13.0.2 | Lấy vị trí |
| permission_handler | ^11.3.1 | Xử lý quyền |
| flutter_tts | ^4.2.0 | Text-to-speech |
| flutter_map | ^7.0.2 | Bản đồ |
| latlong2 | ^0.9.1 | Tọa độ địa lý |
| google_fonts | ^6.2.1 | Phông chữ |
| shimmer | ^3.0.0 | Hiệu ứng loading |
| pinput | ^5.0.0 | Nhập OTP |
| url_launcher | ^6.3.1 | Mở URL |
| firebase_core | ^3.12.1 | Firebase core |
| cloud_firestore | ^5.6.6 | Cloud Firestore |
| firebase_database | ^11.3.5 | Realtime Database |
| firebase_auth | ^5.5.3 | Xác thực |
| firebase_messaging | ^15.2.4 | Push notification |
| flutter_local_notifications | ^21.0.0 | Thông báo cục bộ |

---

## 8. Tài khoản test

### 8.1. Backend (dữ liệu seed tự động)

`FirebaseDataSeeder` tự động tạo các tài khoản sau trong Firestore khi khởi động:

| Email | Mật khẩu | Vai trò |
|---|---|---|
| `khachhang@gmail.com` | `Khoi123@` | Khách hàng (1) |
| `admin@foodgo.com` | `Admin123@` | Quản trị viên (4) |
| `trangkimdatst2005@gmail.com` | `Kimdat@123` | Tài xế (2) |
| `luudinhnghia30012005@gmail.com` | `Nghia123@` | Người bán (3) |
| `taixe2@gmail.com` | `Taixe123@` | Tài xế (2) |
| `taixe3@gmail.com` | `Taixe123@` | Tài xế (2) |

### 8.2. Firebase Authentication

Nếu đăng nhập bằng Firebase Authentication (khách hàng, tài xế), cần tạo tài khoản tay trong **Firebase Console → Authentication → Users → Add user**.

### 8.3. Firestore Collections đã seed

Dữ liệu mẫu đã được seed vào: `users`, `wallets`, `transactions`, `system_configs`, `categories`, `stores`, `products`, `banners`, `vouchers`, `reviews`, `orders`, `customer_profiles`, `driver_profiles`, `merchant_profiles`, `admin_profiles`.

---

## 9. Cấu trúc API

### 9.1. Base URL

```
http://localhost:8086/api
```

### 9.2. Cấu trúc chung response

```json
{
  "success": true,
  "message": "Thành công",
  "data": { ... }
}
```

### 9.3. Authentication

| Endpoint | Phương thức | Mô tả |
|---|---|---|
| `/api/auth/login` | POST | Đăng nhập → trả về JWT token |
| `/api/auth/register` | POST | Đăng ký |
| `/api/auth/refresh` | POST | Refresh token |
| `/api/auth/logout` | POST | Đăng xuất |

JWT token gửi kèm trong header:
```
Authorization: Bearer <token>
```

### 9.4. Các endpoint chính

| Nhóm | Prefix | Mô tả |
|---|---|---|
| Auth | `/api/auth/*` | Đăng nhập, đăng ký, refresh token |
| User | `/api/users/*` | Thông tin người dùng |
| Store | `/api/stores/*` | Quản lý cửa hàng |
| Product | `/api/products/*` | Quản lý sản phẩm |
| Order | `/api/orders/*` | Xử lý đơn hàng |
| Voucher | `/api/vouchers/*` | Quản lý voucher |
| Payment | `/api/payments/*` | Thanh toán |
| Wallet | `/api/wallets/*` | Ví tiền |
| Review | `/api/reviews/*` | Đánh giá |
| Category | `/api/categories/*` | Danh mục |
| Banner | `/api/banners/*` | Banner quảng cáo |
| Profile | `/api/profiles/*` | Profile người dùng |
| Stats | `/api/stats/*` | Thống kê |
| WebSocket | `/ws` | Kênh real-time (STOMP) |

### 9.5. Trạng thái đơn hàng

| Giá trị | Trạng thái |
|---|---|
| 0 | Chờ xác nhận |
| 1 | Đang chuẩn bị |
| 2 | Đang giao |
| 3 | Hoàn thành |
| 4 | Đã hủy |

### 9.6. Phương thức thanh toán

| Giá trị | Mô tả |
|---|---|
| `cash` | Tiền mặt (COD) |
| `momo` | Ví MoMo |
| `zalo` | ZaloPay |
| `vnpay` | VNPay |
| `card` | Thẻ ngân hàng |

---

## 10. Các lưu ý quan trọng

### 10.1. Firebase Configuration

#### File cấu hình Firebase

| File | Vị trí | Mục đích |
|---|---|---|
| `google-services.json` | Thư mục gốc dự án | Cấu hình chung cho cả 3 app Android |
| `firebase-service-account.json` | `be-foodgo/src/main/resources/` | Kết nối Firebase Admin SDK (Backend) |
| `firestore.rules` | Thư mục gốc dự án | Firestore Security Rules |
| `GoogleService-Info.plist` | `ios/Runner/` (mỗi app) | Cấu hình Firebase cho iOS |

- File `google-services.json` và `GoogleService-Info.plist` (iOS) **bắt buộc** phải có tại đúng vị trí. Nếu thiếu, ứng dụng sẽ crash khi khởi động.
- File `firebase-service-account.json` chỉ cần cho Backend; **không commit lên Git**.
- Các dịch vụ Firebase cần kích hoạt: Authentication (Email/Password), Firestore Database, Realtime Database, Storage, Cloud Messaging.
- Nếu mất `firebase-service-account.json`: **Firebase Console → Project Settings → Service accounts → Generate new private key**.
- API Key hiện tại: `AIzaSyCr9eK2rGlAGrKwWLYSIvdmrzWJp74ebQo` (Android).

### 10.2. Backend

- **Port mặc định:** `8086`. Đổi port bằng cách sửa `server.port` trong `application.properties`.
- **JWT Secret:** Chuỗi hiện tại là chuỗi test. Trong môi trường production, cần đổi thành chuỗi ngẫu nhiên 256-bit.
- **Firebase Service Account:** Không commit `firebase-service-account.json` lên Git. Nếu mất file, vào Firebase Console → Project Settings → Service accounts → Generate new private key.

### 10.3. Frontend (Flutter)

- **Quyền Vị trí:** Tất cả app đều cần quyền `ACCESS_FINE_LOCATION` và `ACCESS_COARSE_LOCATION`. Nếu không cấp quyền, một số chức năng sẽ không hoạt động.
- **Android minSdkVersion:** Phải >= 21.
- **WebSocket/STOMP:** Ứng dụng Driver kết nối đến Backend qua WebSocket (STOMP) để nhận đơn giao hàng real-time. Đảm bảo endpoint WebSocket đã được cấu hình đúng.
- **Foreground Service (Driver):** Ứng dụng Tài xế sử dụng Foreground Service để cập nhật vị trí khi app ở nền.
- **Push Notification (Driver):** Khi app ở nền, push notification được gửi qua FCM. Cần cấu hình FCM server key phía Backend.

### 10.4. Các lỗi thường gặp

| Lỗi | Giải pháp |
|---|---|
| `Unable to find git` | Cài đặt git và thêm vào PATH |
| `google-services.json` không hợp lệ | Tải lại từ Firebase Console, kiểm tra tên package |
| `minSdkVersion too low` | Tăng `minSdkVersion` trong `build.gradle` lên 21+ |
| `pod install` thất bại (iOS) | Chạy `cd ios && pod install --repo-update` |
| Không gửi được email | Tạo App Password từ Google, kiểm tra `app.email.enabled=true` |
| WebSocket không kết nối | Kiểm tra Backend đã chạy, kiểm tra URL API trong Flutter |
| Firestore permission denied | Kiểm tra rules Firestore trong Firebase Console |
| Lỗi `XMLHttpRequest` trên Web | Chạy `flutter run -d chrome --web-browser-flag "--disable-web-security"` |

---

## Lệnh hữu ích

```bash
# Backend
./mvnw install                              # Cài đặt Maven dependencies
./mvnw spring-boot:run                      # Chạy Backend
./mvnw clean package -DskipTests            # Build JAR

# Flutter
flutter clean && flutter pub get             # Xóa cache và cài lại dependencies
flutter run                                  # Chạy debug
flutter build apk --release                  # Build APK release
flutter build web --release                  # Build Web release
flutter build ios                            # Build iOS (chỉ macOS)
```

---

**Phiên bản:** 0.0.1-SNAPSHOT
