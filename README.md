# Hệ thống Thuyết minh Tự động
Nền tảng SaaS cung cấp trải nghiệm âm thanh đa ngôn ngữ dựa trên vị trí (Geofencing). Tự động phát thuyết minh về văn hóa, lịch sử và ẩm thực khi du khách dạo bước qua các điểm dừng chân.

**Công nghệ & Tính năng chính:**
* **.NET:** MAUI & ASP.NET Core
* **GPS:** Offline Geofencing
* **5+:** Languages Supported

---

## Mục lục
1. [System Overview (Tổng quan hệ thống)](#1-system-overview-tổng-quan-hệ-thống)
2. [MVP Features (Chức năng chính)](#2-mvp-features-chức-năng-chính)
3. [System Flow (Luồng hệ thống / Logic cốt lõi)](#3-system-flow-luồng-hệ-thống--logic-cốt-lõi)
4. [Architecture (Kiến trúc hệ thống)](#4-architecture-kiến-trúc-hệ-thống)
5. [Diagrams (Sơ đồ kiến trúc)](#5-diagrams-sơ-đồ-kiến-trúc)
6. [Use Cases (Ca sử dụng)](#6-use-cases-ca-sử-dụng)
7. [Database Schema (Cấu trúc cơ sở dữ liệu)](#7-database-schema-cấu-trúc-cơ-sở-dữ-liệu)
8. [API Design (Thiết kế API)](#8-api-design-thiết-kế-api)
9. [Core Components (Thành phần cốt lõi)](#9-core-components-thành-phần-cốt-lõi)
10. [CMS & Analytics (Quản trị & Phân tích)](#10-cms--analytics-quản-trị--phân-tích)
11. [UI/UX (Giao diện & Trải nghiệm)](#11-uiux-giao-diện--trải-nghiệm)
12. [Detailed Features (Chi tiết tính năng)](#12-detailed-features-chi-tiết-tính-năng)
13. [User Stories (Câu chuyện người dùng)](#13-user-stories-câu-chuyện-người-dùng)
14. [Non-Functional (Yêu cầu phi chức năng)](#14-non-functional-yêu-cầu-phi-chức-năng)
15. [Conclusion (Tổng kết & Hướng phát triển)](#15-conclusion-tổng-kết--hướng-phát-triển)

---

## 1. System Overview (Tổng quan hệ thống)

### Mô tả hệ thống
Hệ thống cung cấp trải nghiệm thuyết minh tự động (Auto-narration) qua thiết bị di động cho du khách khi tham quan phố ẩm thực Vĩnh Khánh. Dựa trên vị trí GPS (Geofencing), ứng dụng sẽ tự động phát các đoạn audio giới thiệu về lịch sử, văn hóa, và các món ăn đặc trưng của từng quán/khu vực khi du khách đi ngang qua.

### Problem (Vấn đề)
* Phố Vĩnh Khánh ồn ào, đông đúc, du khách nước ngoài khó tiếp cận thông tin văn hóa ẩm thực.
* Thiếu hướng dẫn viên tại chỗ hoặc rào cản ngôn ngữ với chủ quán.
* Trải nghiệm xem bản đồ truyền thống khiến du khách mất tập trung vào cảnh quan thực tế.

### Goal (Mục tiêu)
* Tạo ra một "Virtual Tour Guide" rảnh tay (Hands-free experience).
* Cung cấp nội dung đa ngôn ngữ (Anh, Việt, Hàn, Nhật, Trung).
* Hoạt động ổn định ngay cả khi kết nối mạng kém (Offline-first approach).

---

## 2. MVP Features (Chức năng chính)

### 📱 Mobile App (Client)
* `[Core]` GPS tracking ngầm.
* `[Core]` Geofencing trigger audio tự động.
* `[Feature]` Scan QR Code để phát audio thủ công.
* `[Arch]` Cơ chế Offline-first (Tải package trước khi tour bắt đầu).

### ⚙️ CMS (Admin/Shop)
* `[Data]` Quản lý POI (Point of Interest) & Geofence Polygon.
* `[Media]` Upload & Quản lý Audio files.
* `[Content]` Quản lý bản dịch đa ngôn ngữ.
* `[Logic]` Thiết lập Tour/Route logic.

### 📊 Analytics
* `[Report]` Heatmap: Khu vực khách dừng chân lâu nhất.
* `[Data]` Tracking user movement (Dữ liệu ẩn danh định danh qua SessionID).
* `[Metrics]` Top POI được nghe nhiều nhất.
* `[Metrics]` Thời lượng nghe trung bình (Listening time).

---

## 3. System Flow (Luồng hệ thống / Logic cốt lõi)
Luồng xử lý từ lúc khởi động app đến khi phát audio tự động. Yêu cầu khắt khe về việc xử lý trùng lặp và ưu tiên phát.

**Bước 1: App Initialization & Data Sync**
App khởi động. Gọi API load toàn bộ metadata POI và Geofence data từ Server. Lưu trữ cục bộ vào **SQLite**. User chọn "Download Tour Audio" để tải file .mp3 về local storage.

**Bước 2: Background GPS Tracking**
Hệ thống kích hoạt Service chạy ngầm. Thu thập tọa độ GPS mỗi 3-5 giây. Sử dụng thuật toán *Haversine* hoặc Native Geofencing API để tính khoảng cách đến các điểm POI.

**Bước 3: Geofence Trigger Event**
Tọa độ GPS rơi vào bán kính Geofence (VD: 15m xung quanh quán Ốc Oanh). Event `OnGeofenceEnter` được kích hoạt.
```text
// Pseudocode Trigger Logic
IF (Distance(User, POI) <= POI.Radius) THEN
    TriggerEvent(POI_ID)
END IF
