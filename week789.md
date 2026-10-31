# 📋 KẾ HOẠCH CHI TIẾT TUẦN 7 – TUẦN 8 – TUẦN 9: HỆ THỐNG ĐẤU GIÁ TRỰC TUYẾN
## Bản Đặc Tả Kỹ Thuật Dựa Trên Phân Tích Mã Nguồn Hiện Tại & Kế Hoạch Tuần 1, Tuần 2

> **Môn học:** Lập trình nâng cao (2627I_UET.CS2043_6)  
> **Dự án:** Hệ thống Đấu giá trực tuyến (Online Auction System) — Nhóm 1  
> **Thời gian nước rút:** 01/11/2026 – 07/11/2026 (7 ngày)  
> **Cột mốc chấm tiến độ của Giảng viên:** Thứ Bảy, ngày 07/11/2026  
> **Cơ sở phân tích:**  
> 1. Mã nguồn hiện tại trên nhánh `dev` và nhánh làm việc (`Entity`, `Auction`, `AuctionManager`, `Bid`, `User`, `Item`, `ItemFactory`, `ClientApp`).  
> 2. Kế hoạch Tuần 1 ([plan/week1.md](file:///E:/Dai-hoc/HK3/Lap-trinh-nang-cao/Bai-tap-lon/Lap-trinh-nang-cao-BTL-nhom-1/plan/week1.md)) — Nền tảng OOP & Domain Model.  
> 3. Kế hoạch Tuần 2 ([plan/week2.md](file:///E:/Dai-hoc/HK3/Lap-trinh-nang-cao/Bai-tap-lon/Lap-trinh-nang-cao-BTL-nhom-1/plan/week2.md)) — 5 Trục Chuyên Môn Độc Lập đang thực hiện.

---

# 🔍 TỔNG HỢP HIỆN TRẠNG & BÀN ĐẠP TỪ TUẦN 1 VÀ TUẦN 2

### 1. Thành quả đã chốt từ Tuần 1 (`plan/week1.md` - Tuần 6 HK):
- Đã thiết lập Maven Multi-Module (`auction-server`, `auction-client`), Java 21, JUnit 5.10.2, JavaFX 21.0.2, Gson 2.10.1.
- `Entity.java`: Lớp cơ sở chứa `id` (UUID), `createdAt`, cơ chế fallback null-safe.
- Cây kế thừa `User`: `User` (abstract) $\rightarrow$ `Bidder`, `Seller`, `Admin` có validate email, username, phân quyền.
- Cây kế thừa `Item`: `Item` (abstract) $\rightarrow$ `Electronics`, `Art`, `Vehicle`.
- `ItemFactory.java`: Khởi tạo đối tượng linh hoạt qua mẫu Factory Method.
- `AuctionManager.java`: Quản lý phiên đấu giá duy nhất theo mẫu Singleton, dùng `ConcurrentHashMap`.
- `Bid.java`: Đối tượng đặt giá bất biến (`final`), an toàn dữ liệu.
- Bộ 5 file Unit Test cơ bản: `AuctionManagerTest`, `AuctionTest`, `BidTest`, `ItemFactoryTest`, `UserTest`.

### 2. Sự tiếp nối từ Tuần 2 (`plan/week2.md` - Tuần 7 HK):
Nhóm đã phân công 5 thành viên theo **5 Trục Chuyên Môn (Horizontal Tracks)** trên các nhánh `feature/w7-...`. Kế hoạch này **kế thừa 100% các nhánh và mảng công việc của Tuần 2**, không thay đổi vai trò của ai, nâng cấp liên tục qua Tuần 7 $\rightarrow$ Tuần 8 $\rightarrow$ Tuần 9:
- 🎨 **Mảng 1 (Hùng):** `feature/w7-gui-base` $\rightarrow$ Nền tảng Giao diện JavaFX, Theme CSS, Form quản lý.
- ⚡ **Mảng 2 (Tuấn):** `feature/w7-realtime-bidding-room` $\rightarrow$ Realtime Observer & Giao diện Phòng đấu giá.
- 🌐 **Mảng 3 (Sang):** `feature/w7-network-protocol` $\rightarrow$ Hạ tầng Mạng Socket, Gói tin JSON, CI/CD Checkstyle.
- 💾 **Mảng 4 (Minh):** `feature/w7-persistence-auth` $\rightarrow$ Custom Exceptions, Bảo mật SHA-256 + Salt, Lưu trữ DAO/JSON.
- ⚙️ **Mảng 5 (Quân):** `feature/w7-concurrency-engine` $\rightarrow$ Khóa an toàn luồng `ReentrantLock`, Timer tự đóng phiên, Test chịu tải.

---

# 🏛️ PHẦN 1. TUẦN 7 — ĐA LUỒNG, OBSERVER PATTERN & BẮT ĐẦU JAVAFX

> **Mục tiêu:** Hoàn thiện trọn vẹn các cam kết của Tuần 2 (`plan/week2.md`): Khóa `ReentrantLock` công bằng cho `Auction`, Interface `AuctionObserver`, và bộ giao diện FXML cơ bản nạp qua `SceneManager`.

## 1.1. Bộ câu hỏi vấn đáp miệng Tuần 7 (Ôn tập nội bộ)
1. *Race Condition và Lost Update trong bài toán đấu giá là gì?*  
   $\rightarrow$ Khi 2 Bidder cùng bấm đặt giá ở cùng 1 mili-giây, nếu không khóa đồng bộ, cả 2 cùng đọc giá cũ và cùng ghi đè $\rightarrow$ mất lượt đặt giá (*lost update*) hoặc thứ tự giá bị đảo lộn (*race condition*).
2. *Tại sao phải dùng `new ReentrantLock(true)` (Fair Lock) thay vì từ khóa `synchronized`?*  
   $\rightarrow$ `synchronized` không cam kết thứ tự luồng (có thể gây starvation). `ReentrantLock(true)` là Fair Lock, xếp hàng luồng theo đúng thứ tự thời gian bấm nút (FIFO), hoàn toàn công bằng cho người tham gia đấu giá.
3. *Mô hình Observer Pattern trong hệ thống hoạt động như thế nào?*  
   $\rightarrow$ `Auction` là Subject lưu trạng thái giá. `AuctionObserver` là Interface lắng nghe. Khi có bid mới hoặc đổi trạng thái, `Auction` tự duyệt danh sách và gọi callback thông báo tới giao diện mà không cần Polling F5.

---

## 1.2. Cây thư mục mã nguồn Tuần 7
```text
online-auction-system/
├── auction-server/
│   └── src/
│       ├── main/java/com/auction/server/domain/model/auction/
│       │   ├── Auction.java                 # [Quân] Thay synchronized bằng ReentrantLock(true), thêm observers
│       │   ├── AuctionStatus.java           # [Minh] Vòng đời OPEN -> RUNNING -> FINISHED -> PAID/CANCELED
│       │   ├── AuctionManager.java          # [Sang] Quản lý phiên hoạt động & hủy phiên
│       │   └── observer/
│       │       └── AuctionObserver.java     # [Tuấn] Interface Observer chuẩn
│       └── test/java/com/auction/server/domain/model/auction/
│           ├── AuctionTest.java             # [Quân] Test đặt giá đa luồng
│           └── AuctionObserverTest.java     # [Tuấn] Test callback Observer
└── auction-client/
    └── src/main/
        ├── java/com/auction/client/
        │   ├── ClientApp.java               # [Sang] Nạp theme.css và liên kết SceneManager
        │   ├── util/
        │   │   └── SceneManager.java        # [Hùng] Quản lý chuyển đổi Scene JavaFX
        │   └── controller/
        │       └── BiddingRoomController.java # [Tuấn] Controller phòng đấu giá (Mock Observer)
        └── resources/
            ├── css/
            │   └── theme.css                # [Hùng] Bộ Dark Theme CSS hiện đại
            └── fxml/
                └── BiddingRoomView.fxml     # [Tuấn] Layout phòng đấu giá trực tiếp
```

---

## 1.3. Đặc tả nhiệm vụ chi tiết từng thành viên trong Tuần 7

### 👤 1. TRẦN MẠNH QUÂN — Mảng 5: Concurrency Fair Lock & Observer trong `Auction`
* **Nhánh làm việc:** `feature/w7-concurrency-engine`
* **Hiện trạng trong code:** `Auction.java` đang dùng `public synchronized boolean placeBid()`.
* **Nhiệm vụ chi tiết:**
  1. Khai báo khóa công bằng: `private final ReentrantLock lock = new ReentrantLock(true);`
  2. Khai báo danh sách Observer: `private final List<AuctionObserver> observers = new CopyOnWriteArrayList<>();`
  3. Bổ sung các phương thức quản lý Observer:
     - `public void attachObserver(AuctionObserver observer)`
     - `public void detachObserver(AuctionObserver observer)`
     - `private void notifyNewBid(Bid bid)`: duyệt `observers` gọi `o.onNewBidPlaced(this, bid)`.
     - `public void notifyStatusChanged(AuctionStatus newStatus)`: duyệt `observers` gọi `o.onAuctionStatusChanged(this, newStatus)`.
  4. Cập nhật hàm `placeBid(Bidder bidder, double amount)`: Toàn bộ thân hàm bọc trong `lock.lock()` và `finally { lock.unlock(); }`. Khi bid thành công, gọi `notifyNewBid(newBid)`.
* **Viết Unit Test:** Cập nhật `AuctionTest.java`, bổ sung test 20 luồng đồng thời gọi `placeBid()`.
* **Lệnh kiểm tra:** `mvn test -Dtest="AuctionTest"` $\rightarrow$ **BUILD SUCCESS**.

---

### 👤 2. LÊ ANH TUẤN — Mảng 2: Observer Interface & Giao Diện Phòng Đấu Giá (`BiddingRoom`)
* **Nhánh làm việc:** `feature/w7-realtime-bidding-room`
* **Nhiệm vụ chi tiết:**
  1. Tạo file `com.auction.server.domain.model.auction.observer.AuctionObserver.java`:
     ```java
     public interface AuctionObserver {
         void onNewBidPlaced(Auction auction, Bid newBid);
         void onAuctionStatusChanged(Auction auction, AuctionStatus newStatus);
     }
     ```
  2. Tạo layout `resources/fxml/BiddingRoomView.fxml`: Gồm nhãn giá cao nhất to 32px `#lblCurrentPrice`, người dẫn đầu `#lblCurrentWinner`, đồng hồ `#lblCountdown`, bảng `TableView` lịch sử bid `#tblBidHistory`, các nút đặt giá nhanh `+1 Bước giá`, `+2 Bước giá`, nút `ĐẶT GIÁ`.
  3. Viết `BiddingRoomController.java` implements `AuctionObserver`: Cài đặt callback `@Override void onNewBidPlaced(...)` cập nhật Label và thêm 1 dòng vào TableView. Tạo nút "Đặt giá thử" trên giao diện để test nhảy số realtime.
* **Viết Unit Test:** Tạo `AuctionObserverTest.java` kiểm tra đăng ký và nhận sự kiện Observer.
* **Lệnh kiểm tra:** `mvn test -Dtest="AuctionObserverTest"` $\rightarrow$ **PASS 100%**.

---

### 👤 3. ĐỖ MẠNH HÙNG — Mảng 1: Hệ Thống CSS Dark Theme & Điều Hướng `SceneManager`
* **Nhánh làm việc:** `feature/w7-gui-base`
* **Nhiệm vụ chi tiết:**
  1. Tạo file `resources/css/theme.css`: Thiết kế Dark Theme hiện đại với nền tối `#1e1e2f`, `#27293d`, chữ trắng `#ffffff`. Định dạng `.button` bo góc `8px`, hiệu ứng `:hover` sáng cyan, `:pressed` thụt nhẹ; style cho `.text-field` và `.table-view`.
  2. Tạo lớp `com.auction.client.util.SceneManager.java` (Singleton):
     - `public static synchronized SceneManager getInstance()`
     - `public void setPrimaryStage(Stage stage)`
     - `public void switchScene(String fxmlPath, String title)`: nạp FXML bằng `FXMLLoader`, gán `theme.css`, chuyển đổi scene mượt mà.
* **Kiểm tra đầu ra:** Khởi chạy `ClientApp`, mở cửa sổ giao diện 1000x700 với Dark Theme sắc nét.

---

### 👤 4. HOÀNG THẾ CÔNG MINH — Mảng 4: Nghiệp Vụ Validate & Luân Chuyển Trạng Thái
* **Nhánh làm việc:** `feature/w7-persistence-auth`
* **Nhiệm vụ chi tiết:**
  1. Bổ sung hàm luân chuyển trạng thái trong `Auction.java`:
     - `public boolean updateStatus(AuctionStatus newStatus)`: Kiểm tra chuyển đổi hợp lệ (`OPEN` $\rightarrow$ `RUNNING` $\rightarrow$ `FINISHED` $\rightarrow$ `PAID`), kích hoạt `notifyStatusChanged(newStatus)`.
  2. Tăng cường validate trong `placeBid()`: Kiểm tra chặt chẽ `bidder.getBalance() >= amount` và cấm Seller tự đấu giá sản phẩm của chính mình.

---

### 👤 5. LÊ ĐỨC THIỆN SANG — Mảng 3: Quản Lý Phiên `AuctionManager` & Cấu Hình Khởi Động
* **Nhánh làm việc:** `feature/w7-network-protocol`
* **Nhiệm vụ chi tiết:**
  1. Nâng cấp `AuctionManager.java`: Bổ sung `cancelAuction(String auctionId, User requester)` kiểm tra nếu `requester` là `Admin` hoặc là `Seller` sở hữu phiên thì mới cho phép hủy.
  2. Cập nhật `ClientApp.java`: Khởi tạo `SceneManager`, nạp `BiddingRoomView.fxml` làm màn hình mặc định ban đầu để cả nhóm kiểm tra giao diện.

---

## 1.4. Ma trận Review chéo & Checklist Tuần 7

| PR Author | Nhánh làm việc | Reviewer 1 (Chính) | Reviewer 2 (Phụ) |
| :--- | :--- | :---: | :---: |
| **Quân** | `feature/w7-concurrency-engine` | Sang | Tuấn |
| **Tuấn** | `feature/w7-realtime-bidding-room` | Hùng | Quân |
| **Hùng** | `feature/w7-gui-base` | Tuấn | Minh |
| **Minh** | `feature/w7-persistence-auth` | Quân | Sang |
| **Sang** | `feature/w7-network-protocol` | Minh | Hùng |

### Checklist nghiệm thu Tuần 7:
- [ ] `Auction.placeBid()` được khóa an toàn bằng `ReentrantLock(true)`.
- [ ] Interface `AuctionObserver` định nghĩa đúng chuẩn 2 hàm callback.
- [ ] File `theme.css` Dark theme hoàn chỉnh.
- [ ] Mở giao diện `BiddingRoomView.fxml`, bấm đặt giá thử thì nhãn giá tự động nhảy số tức thì.

---
---

# 🛡️ TUẦN 8 — KIỂM THỬ, NGOẠI LỆ (CUSTOM EXCEPTIONS) & GUI MVC NÂNG CAO

> **Mục tiêu:** Xây dựng hệ thống ngoại lệ chuyên biệt (`exception`), thay thế các lệnh trả về `return false` bằng Custom Exceptions, viết bộ Unit Test JUnit 5 (`assertThrows`), tự động hóa đóng phiên (`AuctionScheduler`), và hoàn thiện các màn hình FXML theo chuẩn MVC.

## 2.1. Bộ câu hỏi vấn đáp miệng Tuần 8 (Ôn tập nội bộ)
1. *Tại sao cần dùng Custom Exceptions thay vì ném RuntimeException chung chung?*  
   $\rightarrow$ Giúp phân loại chính xác lỗi nghiệp vụ (như đặt giá sai, phiên đóng, sai mật khẩu), giúp Controller bắt đúng loại lỗi để hiển thị thông báo thân thiện tới người dùng.
2. *Trong JUnit 5, hàm `assertThrows()` hoạt động như thế nào?*  
   $\rightarrow$ `assertThrows(ExpectedException.class, () -> executable)` thực thi đoạn mã; nếu ném đúng class ngoại lệ mong muốn thì test PASS, nếu không ném hoặc ném sai loại thì test FAIL.
3. *Mô hình MVC trong JavaFX phân tách như thế nào?*  
   $\rightarrow$ **View** là file FXML; **Controller** bắt sự kiện và gọi Model/Service; **Model** chứa nghiệp vụ thuần túy, hoàn toàn không phụ thuộc vào UI.

---

## 2.2. Cây thư mục mã nguồn Tuần 8
```text
online-auction-system/
├── auction-server/
│   └── src/
│       ├── main/java/com/auction/server/
│       │   ├── exception/                       # [Minh - Mới] Package Custom Exceptions
│       │   │   ├── AuctionException.java        # Exception gốc
│       │   │   ├── InvalidBidException.java     # Lỗi giá đặt không hợp lệ
│       │   │   ├── AuctionClosedException.java  # Lỗi đấu giá khi phiên đã kết thúc
│       │   │   └── AuthenticationException.java # Lỗi đăng nhập thất bại
│       │   ├── domain/model/auction/
│       │   │   ├── Auction.java                 # [Quân - Sửa đổi] Ném InvalidBidException & AuctionClosedException
│       │   │   └── AuctionScheduler.java        # [Quân - Mới] Timer đếm ngược tự đóng phiên
│       │   └── service/
│       │       └── UserService.java             # [Minh - Mới] Dịch vụ xác thực tài khoản
│       └── test/java/com/auction/server/
│           ├── exception/
│           │   └── ExceptionTest.java           # [Sang - Mới] Bộ test JUnit 5 cho ngoại lệ
│           └── auction/
│               └── AuctionSchedulerTest.java    # [Quân - Mới] Test lập lịch đóng phiên
└── auction-client/
    └── src/main/
        ├── java/com/auction/client/controller/
        │   ├── LoginController.java             # [Hùng - Mới] Controller Đăng nhập
        │   ├── RegisterController.java          # [Hùng - Mới] Controller Đăng ký
        │   ├── SellerDashboardController.java   # [Hùng - Mới] Controller Quản lý kho hàng Seller
        │   └── BiddingRoomController.java       # [Tuấn - Sửa đổi] Bắt ngoại lệ hiển thị Alert popup
        └── resources/fxml/
            ├── LoginView.fxml                   # [Hùng - Mới] Form đăng nhập
            ├── RegisterView.fxml                # [Hùng - Mới] Form đăng ký tài khoản
            ├── SellerDashboard.fxml             # [Hùng - Mới] Bảng điều khiển người bán
            └── AddItemDialog.fxml               # [Hùng - Mới] Hộp thoại tạo sản phẩm mới
```

---

## 2.3. Đặc tả nhiệm vụ chi tiết từng thành viên trong Tuần 8

### 👤 1. HOÀNG THẾ CÔNG MINH — Mảng 4: Cụm Custom Exceptions & Dịch Vụ `UserService`
* **Nhánh làm việc:** `feature/w8-minh/custom-exceptions-auth`
* **Nhiệm vụ chi tiết:**
  1. Tạo package `com.auction.server.exception` gồm 4 lớp:
     - `AuctionException.java` (`extends RuntimeException`): Lớp cha chung.
     - `InvalidBidException.java`: Ném ra khi giá đặt nhỏ hơn giá sàn, nhỏ hơn bước giá, hoặc số dư không đủ.
     - `AuctionClosedException.java`: Ném ra khi đặt giá vào phiên không ở trạng thái `RUNNING`.
     - `AuthenticationException.java`: Ném ra khi đăng nhập thất bại.
  2. Tạo lớp `com.auction.server.service.UserService.java`:
     - `register(username, password, email, role)`: Validate thông tin, ném `AuctionException("Username đã tồn tại")` nếu trùng.
     - `login(username, password)`: Ném `AuthenticationException("Sai tên đăng nhập hoặc mật khẩu")` nếu thông tin không khớp.

---

### 👤 2. TRẦN MẠNH QUÂN — Mảng 5: Refactor Ném Ngoại Lệ & Bộ Đếm Thời Gian `AuctionScheduler`
* **Nhánh làm việc:** `feature/w8-quan/exception-refactor-scheduler`
* **Nhiệm vụ chi tiết:**
  1. Refactor hàm `placeBid()` trong `Auction.java`:
     - Thay thế việc `return false` bằng việc ném Custom Exceptions:
       - Nếu phiên không `OPEN`/`RUNNING` $\rightarrow$ ném `AuctionClosedException`.
       - Nếu số tiền không hợp lệ hoặc số dư không đủ $\rightarrow$ ném `InvalidBidException`.
  2. Tạo lớp `com.auction.server.domain.model.auction.AuctionScheduler.java`:
     - Sử dụng `private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);`
     - Phương thức `public void scheduleAuctionEnd(Auction auction)`: Tính số mili-giây còn lại đến `auction.getEndTime()`. Khi hết giờ $\rightarrow$ tự động gọi `auction.updateStatus(AuctionStatus.FINISHED)`, kích hoạt `auction.determineWinner()`, và thông báo tới các Observer.
     - Phương thức `public void shutdown()`: dừng an toàn luồng đếm thời gian.

---

### 👤 3. LÊ ĐỨC THIỆN SANG — Mảng 3: Bộ Kiểm Thử Ngoại Lệ JUnit 5 & Rà Soát SOLID
* **Nhánh làm việc:** `feature/w8-sang/exception-test-solid`
* **Nhiệm vụ chi tiết:**
  1. Tạo file `com.auction.server.exception.ExceptionTest.java`:
     - Viết tối thiểu 8 ca kiểm thử bắt buộc sử dụng `assertThrows`:
       - Test ném `InvalidBidException` khi đặt giá nhỏ hơn bước giá tối thiểu.
       - Test ném `InvalidBidException` khi số dư tài khoản của Bidder nhỏ hơn mức giá đặt.
       - Test ném `AuctionClosedException` khi đặt giá vào phiên đã `FINISHED` hoặc `CANCELED`.
       - Test ném `AuthenticationException` khi đăng nhập với mật khẩu sai.
  2. Rà soát chuẩn hóa SOLID: Đảm bảo toàn bộ thuộc tính trong các Entity đều là `private`, loại bỏ mã nguồn thừa (*code smells*).
* **Lệnh kiểm tra:** `mvn test -Dtest="ExceptionTest"` $\rightarrow$ **PASS 100%**.

---

### 👤 4. ĐỖ MẠNH HÙNG — Mảng 1: Hoàn Thiện Các Form FXML Chuẩn MVC
* **Nhánh làm việc:** `feature/w8-hung/mvc-form-views`
* **Nhiệm vụ chi tiết:**
  1. Tạo `resources/fxml/LoginView.fxml` + `LoginController.java`: Ô nhập username, password, nút Đăng nhập. Nhãn `#lblError` hiển thị chữ đỏ khi bắt ngoại lệ `AuthenticationException`. Bấm Đăng nhập thành công $\rightarrow$ chuyển sang `SellerDashboard.fxml`.
  2. Tạo `resources/fxml/RegisterView.fxml` + `RegisterController.java`: Form đăng ký đầy đủ thông tin.
  3. Tạo `resources/fxml/SellerDashboard.fxml` + `SellerDashboardController.java`: Bảng `TableView` danh sách sản phẩm đăng bán, nút mở hộp thoại popup `AddItemDialog.fxml` để thêm sản phẩm mới.

---

### 👤 5. LÊ ANH TUẤN — Mảng 2: Xử Lý Ngoại Lệ & Đồng Hồ Đếm Ngược Trên UI Phòng Đấu Giá
* **Nhánh làm việc:** `feature/w8-tuan/mvc-bidding-room-controller`
* **Nhiệm vụ chi tiết:**
  1. Cập nhật `BiddingRoomController.java`:
     - Khi bấm nút `ĐẶT GIÁ`: Bọc trong khối `try-catch`, bắt `InvalidBidException` hoặc `AuctionClosedException` và mở popup `Alert(AlertType.WARNING)` hiển thị thông báo lỗi thân thiện thay vì làm sập ứng dụng.
  2. Xây dựng đồng hồ đếm ngược trực quan: Sử dụng `javafx.animation.Timeline` chạy mỗi 1 giây cập nhật thời gian còn lại lên `#lblCountdown`. Khi còn dưới 60 giây, đổi chữ sang màu đỏ cảnh báo.

---

## 2.4. Ma trận Review chéo & Checklist Tuần 8

| PR Author | Nhánh làm việc | Reviewer 1 (Chính) | Reviewer 2 (Phụ) |
| :--- | :--- | :---: | :---: |
| **Minh** | `feature/w8-minh/custom-exceptions-auth` | Sang | Hùng |
| **Quân** | `feature/w8-quan/exception-refactor-scheduler` | Minh | Tuấn |
| **Sang** | `feature/w8-sang/exception-test-solid` | Quân | Tuấn |
| **Hùng** | `feature/w8-hung/mvc-form-views` | Tuấn | Sang |
| **Tuấn** | `feature/w8-tuan/mvc-bidding-room-controller` | Hùng | Quân |

### Checklist nghiệm thu Tuần 8:
- [ ] 4 lớp Custom Exception đã được tạo đúng package.
- [ ] Hàm `Auction.placeBid()` ném đúng ngoại lệ thay vì return false.
- [ ] Bộ test `ExceptionTest.java` có tối thiểu 8 ca test PASS 100%.
- [ ] Giao diện bắt ngoại lệ hiển thị popup hoặc label đỏ, không bị văng ứng dụng.
- [ ] Kiến trúc JavaFX tuân thủ nghiêm ngặt MVC (FXML tách rời Controller).

---
---

# 🌐 TUẦN 9 — TÍCH HỢP HỆ THỐNG, MẠNG SOCKET & LƯU TRỮ DỮ LIỆU

> **Mục tiêu:** Xây dựng **Hạ tầng Mạng Socket Client–Server đa luồng**, chuẩn hóa gói tin **JSON (Gson)**, lưu trữ dữ liệu ra file bằng **DAO & JsonStorage**, băm mật khẩu **SHA-256 + Salt**, và tích hợp **Checkstyle Google Java Style** vào Maven + GitHub Actions.

## 3.1. Bộ câu hỏi vấn đáp miệng Tuần 9 (Ôn tập nội bộ)
1. *Cơ chế hoạt động của ServerSocket và Socket trong Java?*  
   $\rightarrow$ `ServerSocket.accept()` lắng nghe tại cổng 8888. Khi có Client kết nối, Server bàn giao Socket đó cho 1 `ClientHandler` chạy trên Thread Pool để phục vụ song song, không làm nghẽn luồng tiếp nhận chính.
2. *Tại sao khi cập nhật giao diện JavaFX từ luồng Socket bắt buộc phải dùng `Platform.runLater()`?*  
   $\rightarrow$ JavaFX tuân thủ nguyên tắc *Single-Threaded UI Rule*. Mọi thao tác vẽ hoặc thay đổi component bắt buộc phải diễn ra trên luồng chính *JavaFX Application Thread*. Dữ liệu mạng chạy trên luồng ngầm, do đó phải đưa tác vụ vào hàng đợi của luồng UI qua `Platform.runLater()`.
3. *Tại sao phải thêm chuỗi Salt ngẫu nhiên khi băm mật khẩu bằng SHA-256?*  
   $\rightarrow$ Thêm Salt ngẫu nhiên 16 bytes giúp vô hiệu hóa kỹ thuật tấn công *Rainbow Table* (bảng tra cứu mật khẩu dựng sẵn) và đảm bảo hai tài khoản có cùng mật khẩu vẫn có chuỗi băm hoàn toàn khác nhau.

---

## 3.2. Cây thư mục mã nguồn Tuần 9
```text
online-auction-system/
├── pom.xml                                      # [Sang] Thêm maven-checkstyle-plugin
├── .github/workflows/
│   └── maven-ci.yml                             # [Sang] CI kiểm tra cả test và checkstyle
├── auction-server/
│   ├── data/                                    # [Minh] Thư mục chứa CSDL JSON
│   │   ├── users.json
│   │   └── auctions.json
│   └── src/
│       ├── main/java/com/auction/server/
│       │   ├── security/
│       │   │   └── PasswordHasher.java          # [Minh] Băm mật khẩu SHA-256 + Salt
│       │   ├── persistence/
│       │   │   ├── GenericDAO.java              # [Minh] Interface Generic DAO
│       │   │   ├── UserDAOImpl.java             # [Minh] Đọc ghi data/users.json
│       │   │   ├── AuctionDAOImpl.java          # [Minh] Đọc ghi data/auctions.json
│       │   │   └── JsonStorage.java             # [Minh] Tiện ích File I/O Gson Pretty
│       │   └── network/
│       │       ├── protocol/                    # [Sang] Gói tin chuẩn JSON
│       │       │   ├── ActionType.java          # Enum các hành động mạng
│       │       │   ├── ResponseStatus.java      # Enum mã phản hồi (OK, ERROR...)
│       │       │   ├── Request.java             # Object gói tin Client gửi
│       │       │   ├── Response.java            # Object gói tin Server trả
│       │       │   └── MessageSerializer.java   # Chuyển đổi JSON bằng Gson
│       │       └── server/
│       │           ├── AuctionServer.java       # [Sang] ServerSocket cổng 8888 đa luồng
│       │           └── ClientHandler.java       # [Sang] Phục vụ từng Client riêng biệt
│       └── test/java/com/auction/server/
│           ├── concurrency/
│           │   └── AuctionConcurrencyTest.java  # [Quân] Test 50 luồng đồng thời (CountDownLatch)
│           └── persistence/
│               └── UserDAOTest.java             # [Minh] Test đọc ghi file JSON
└── auction-client/
    └── src/main/java/com/auction/client/
        ├── network/
        │   └── AuctionClient.java               # [Tuấn] Socket Client chạy ngầm
        └── controller/
            └── (Các Controller kết nối với AuctionClient thay vì mock data) # [Hùng & Tuấn]
```

---

## 3.3. Đặc tả nhiệm vụ chi tiết từng thành viên trong Tuần 9

### 👤 1. LÊ ĐỨC THIỆN SANG — Mảng 3: Hạ Tầng Socket Server, Giao Thức JSON & Checkstyle
* **Nhánh làm việc:** `feature/w9-sang/socket-server-checkstyle`
* **Nhiệm vụ chi tiết:**
  1. Thiết kế Giao thức Gói tin chuẩn JSON (`com.auction.server.network.protocol`):
     - `ActionType`: `LOGIN`, `REGISTER`, `GET_AUCTIONS`, `PLACE_BID`, `CREATE_ITEM`, `NEW_BID_NOTIFICATION`, `PING`.
     - `ResponseStatus`: `OK`, `ERROR`, `UNAUTHORIZED`, `NOT_FOUND`.
     - `Request.java`: `action: ActionType`, `data: Map<String, Object>`, `token: String`.
     - `Response.java`: `status: ResponseStatus`, `message: String`, `data: Object`.
     - `MessageSerializer.java`: Chuyển đổi Gson giữa `Request`/`Response` và chuỗi JSON.
  2. Hiện thực `AuctionServer.java` & `ClientHandler.java`:
     - Mở `ServerSocket` cổng `8888`.
     - Quản lý luồng Client bằng `ExecutorService threadPool = Executors.newCachedThreadPool();`.
     - Quản lý danh sách kết nối: `List<ClientHandler> clients = new CopyOnWriteArrayList<>();`.
     - Hàm `broadcast(Response res)`: Duyệt `clients` gửi gói tin cập nhật tới tất cả Client đang kết nối.
  3. Cấu hình `maven-checkstyle-plugin` (Google Java Style) vào `pom.xml` và cập nhật file CI `.github/workflows/maven-ci.yml`.
* **Lệnh kiểm tra:** `mvn checkstyle:check` $\rightarrow$ **BUILD SUCCESS (0 lỗi format)**.

---

### 👤 2. LÊ ANH TUẤN — Mảng 2: Socket Client & Realtime Push Cập Nhật Giao Diện
* **Nhánh làm việc:** `feature/w9-tuan/socket-client-realtime`
* **Nhiệm vụ chi tiết:**
  1. Tạo lớp `com.auction.client.network.AuctionClient.java`:
     - Mở kết nối Socket tới Server cổng `8888`.
     - Khởi chạy một luồng chạy ngầm liên tục lắng nghe các gói tin JSON push từ Server.
     - Hàm gửi dữ liệu: `public void sendRequest(Request request)`.
  2. Tích hợp Realtime Update vào `BiddingRoomController`:
     - Khi nhận gói tin `NEW_BID_NOTIFICATION`:
       ```java
       Platform.runLater(() -> {
           lblCurrentPrice.setText(String.format("%,.0f VNĐ", newBid.getAmount()));
           lblCurrentWinner.setText("Dẫn đầu: " + newBid.getBidderUsername());
           tblBidHistory.getItems().add(0, newBid);
       });
       ```
     - Khi bấm nút `ĐẶT GIÁ`: Tạo `Request(ActionType.PLACE_BID)` gửi qua Socket lên Server.

---

### 👤 3. HOÀNG THẾ CÔNG MINH — Mảng 4: Bảo Mật Mật Khẩu (SHA-256 + Salt) & Tầng Lưu Trữ DAO
* **Nhánh làm việc:** `feature/w9-minh/persistence-security-dao`
* **Nhiệm vụ chi tiết:**
  1. Tạo lớp `com.auction.server.security.PasswordHasher.java`:
     - `public static String generateSalt()`: Sinh chuỗi ngẫu nhiên 16 bytes bằng `SecureRandom`.
     - `public static String hashPassword(String password, String salt)`: Băm bằng `SHA-256`.
     - `public static boolean verifyPassword(String inputPassword, String storedHash, String storedSalt)`.
  2. Xây dựng Tầng Lưu Trữ File JSON (`persistence`):
     - `GenericDAO<T extends Entity>`: Interface cung cấp 5 hàm CRUD: `save()`, `findById()`, `findAll()`, `update()`, `delete()`.
     - `JsonStorage.java`: Tiện ích đọc và ghi danh sách đối tượng Java ra file `.json` bằng Gson pretty printing.
     - `UserDAOImpl.java`: Đọc/ghi danh sách tài khoản vào file `data/users.json`. Mật khẩu lưu ở dạng băm kèm salt.
     - `AuctionDAOImpl.java`: Đọc/ghi danh sách phiên đấu giá vào file `data/auctions.json`. Khi khởi động Server, tự động tải dữ liệu từ file vào bộ nhớ RAM.

---

### 👤 4. TRẦN MẠNH QUÂN — Mảng 5: Bộ Kiểm Thử Concurrency Chuyên Sâu (50 Threads)
* **Nhánh làm việc:** `feature/w9-quan/concurrency-stress-test`
* **Nhiệm vụ chi tiết:**
  1. Viết `AuctionConcurrencyTest.java` chuyên sâu:
     - Tạo 1 phiên đấu giá với giá khởi điểm `1,000.0`, bước giá `50.0`.
     - Tạo **50 đối tượng `Bidder`** cùng số dư lớn.
     - Sử dụng `CountDownLatch readyLatch = new CountDownLatch(50);` và `CountDownLatch startLatch = new CountDownLatch(1);` để ép 50 luồng xuất phát tại cùng một mili-giây.
     - Dùng `ExecutorService executor = Executors.newFixedThreadPool(50);`.
  2. Assertions kiểm tra:
     - Không phát sinh ngoại lệ tranh chấp luồng.
     - Danh sách `bidHistory` có giá tăng dần đơn điệu, không có hiện tượng nhảy lùi giá (*rollback*).
     - Giá cao nhất cuối cùng `currentHighestBid` bằng chính xác giá của lượt bid cuối cùng trong lịch sử.
* **Lệnh kiểm tra:** `mvn test -Dtest="AuctionConcurrencyTest"` $\rightarrow$ **PASS 100%**.

---

### 👤 5. ĐỖ MẠNH HÙNG — Mảng 1: Tích Hợp GUI Với Hạ Tầng Mạng Socket
* **Nhánh làm việc:** `feature/w9-hung/gui-socket-integration`
* **Nhiệm vụ chi tiết:**
  1. Chuyển đổi các Controller (`LoginController`, `RegisterController`, `SellerDashboardController`) sang gửi `Request` qua `AuctionClient`:
     - Khi bấm Đăng nhập $\rightarrow$ gửi `Request(ActionType.LOGIN)` lên Server, đợi phản hồi `Response(status=OK)` mới chuyển màn hình.
     - Khi tạo sản phẩm mới $\rightarrow$ gửi `Request(ActionType.CREATE_ITEM)` lên Server để lưu vào file CSDL JSON.
  2. Xử lý lỗi ngắt kết nối: Hiển thị hộp thoại cảnh báo nếu Server chưa bật hoặc bị mất mạng đột ngột.

---

## 3.4. Ma trận Review chéo & Checklist Tuần 9

| PR Author | Nhánh làm việc | Reviewer 1 (Chính) | Reviewer 2 (Phụ) |
| :--- | :--- | :---: | :---: |
| **Sang** | `feature/w9-sang/socket-server-checkstyle` | Minh | Quân |
| **Tuấn** | `feature/w9-tuan/socket-client-realtime` | Quân | Hùng |
| **Minh** | `feature/w9-minh/persistence-security-dao` | Sang | Hùng |
| **Quân** | `feature/w9-quan/concurrency-stress-test` | Sang | Tuấn |
| **Hùng** | `feature/w9-hung/gui-socket-integration` | Tuấn | Minh |

### Checklist nghiệm thu Tuần 9:
- [ ] Lệnh `mvn checkstyle:check` đạt chuẩn Google Java Style.
- [ ] Khởi động `AuctionServer` cổng 8888 chạy nền ổn định.
- [ ] Dữ liệu người dùng được băm SHA-256 + Salt và lưu ra file `data/users.json`.
- [ ] Mở 2 Client đặt giá: Giá nhảy tức thì trên cả 2 màn hình qua Socket và `Platform.runLater()`.
- [ ] Tắt Server rồi bật lại: Toàn bộ phiên và lịch sử bid vẫn còn nguyên vẹn.
- [ ] Test `AuctionConcurrencyTest` 50 luồng PASS 100%.
