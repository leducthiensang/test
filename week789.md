# 📋 KẾ HOẠCH CHI TIẾT TUẦN 7 – TUẦN 8 – TUẦN 9: HỆ THỐNG ĐẤU GIÁ TRỰC TUYẾN
## Bản Đặc Tả Kỹ Thuật Kế Thừa 100% Tuần 2 Hiện Tại (`plan/week2.md`) & Mở Rộng Sang Tuần 8, 9

> **Môn học:** Lập trình nâng cao (2627I_UET.CS2043_6)  
> **Dự án:** Hệ thống Đấu giá trực tuyến (Online Auction System) — Nhóm 1  
> **Thời gian nước rút:** 01/11/2026 – 07/11/2026 (Đúng 7 ngày)  
> **Cột mốc chấm tiến độ của Giảng viên:** Thứ Bảy, ngày 07/11/2026  
> **Nguyên tắc kế thừa tối cao:**  
> - **TUẦN 7 trong bản kế hoạch này chính là TUẦN 2 mà nhóm đang làm dở ([plan/week2.md](file:///E:/Dai-hoc/HK3/Lap-trinh-nang-cao/Bai-tap-lon/Lap-trinh-nang-cao-BTL-nhom-1/plan/week2.md)).** Toàn bộ phân công 5 Mảng Chuyên Môn, tên nhánh `feature/w7-...`, class, method và giao kèo kỹ thuật (Contracts) của Tuần 2 được **giữ nguyên 100%**, không ai phải đổi mảng hay làm lại từ đầu!  
> - **TUẦN 8** là bước nâng cấp tự nhiên: Bổ sung bộ Custom Exceptions (`InvalidBidException`, `AuctionClosedException`...), viết Unit Test ngoại lệ (`assertThrows`), hoàn thiện FXML theo mô hình MVC.  
> - **TUẦN 9** là bước tích hợp mạng và hoàn thiện: Cắm Socket Client–Server End-to-End, Broadcast Realtime qua Socket (`Platform.runLater`), lưu trữ file JSON bền vững, và tích hợp Checkstyle Google Style vào Maven/CI.

---

# 🔍 TỔNG HỢP HIỆN TRẠNG & BÀN ĐẠP TỪ TUẦN 1 VÀ TUẦN 2

### 1. Nền tảng đã xong từ Tuần 1 (`plan/week1.md` - Tuần 6 HK):
- Maven Multi-Module (`auction-server`, `auction-client`), Java 21 LTS, JUnit 5.10.2, JavaFX 21.0.2, Gson 2.10.1.
- `Entity.java`: Lớp cơ sở chứa `id` (UUID), `createdAt`, fallback null-safe.
- Cây kế thừa `User`: `User` (abstract) $\rightarrow$ `Bidder`, `Seller`, `Admin`.
- Cây kế thừa `Item`: `Item` (abstract) $\rightarrow$ `Electronics`, `Art`, `Vehicle`.
- `ItemFactory.java`: Factory Method tạo sản phẩm tập trung.
- `AuctionManager.java`: Singleton quản lý phiên, dùng `ConcurrentHashMap`.
- `Bid.java`: Đối tượng đặt giá bất biến (`final`).
- Bộ 5 file test JUnit 5 cơ bản: `AuctionManagerTest`, `AuctionTest`, `BidTest`, `ItemFactoryTest`, `UserTest`.

### 2. Tiến độ đang làm dở của Tuần 2 (`plan/week2.md` - Tuần 7 HK):
- Đã thêm thư viện `Gson 2.10.1` và liên kết `auction-server` vào `auction-client` trong `pom.xml` (Commit `77869dc`).
- Cả 5 thành viên đang triển khai trên 5 nhánh cá nhân độc lập của Tuần 2:
  * 🎨 **Mảng 1: Đỗ Mạnh Hùng** (`feature/w7-gui-base`) $\rightarrow$ GUI Base & CSS Theme.
  * ⚡ **Mảng 2: Lê Anh Tuấn** (`feature/w7-realtime-bidding-room`) $\rightarrow$ Realtime UI Room & Observer.
  * 🌐 **Mảng 3: Lê Đức Thiện Sang** (`feature/w7-network-protocol`) $\rightarrow$ Network Protocol & Server Socket.
  * 💾 **Mảng 4: Hoàng Thế Công Minh** (`feature/w7-persistence-auth`) $\rightarrow$ Persistence & Auth Core.
  * ⚙️ **Mảng 5: Trần Mạnh Quân** (`feature/w7-concurrency-engine`) $\rightarrow$ Core Engine & Concurrency.

---

# 🏛️ PHẦN 1. TUẦN 7 — BỔ SUNG NỀN MÓNG & XÂY DỰNG 5 TRỤC ĐỘC LẬP (TUẦN 2 HIỆN TẠI)

> **Mục tiêu:** Kế thừa nguyên văn các cam kết của [plan/week2.md](file:///E:/Dai-hoc/HK3/Lap-trinh-nang-cao/Bai-tap-lon/Lap-trinh-nang-cao-BTL-nhom-1/plan/week2.md). Mỗi thành viên sở hữu 1 khối mã nguồn **tự chạy được, tự test được** trên nhánh riêng bằng Mock data, chưa cần nối mạng với nhau để triệt tiêu hoàn toàn sự phụ thuộc.

## 1.1. Các Giao Kèo Hợp Đồng (Contracts) Tuần 7 (Từ `plan/week2.md`)

### Contract 1: Chuẩn gói tin Mạng (Mảng 3 sở hữu)
```json
// Gói tin gửi đi (Request):
{
  "action": "LOGIN",
  "data": { "username": "tuan", "password": "123" },
  "token": "uuid-neu-da-dang-nhap"
}

// Gói tin phản hồi (Response):
{
  "status": "OK",
  "message": "Đăng nhập thành công",
  "data": { "userId": "...", "username": "tuan", "role": "BIDDER" }
}
```

### Contract 2: Tầng Lưu Trữ & DAO (Mảng 4 sở hữu)
```java
public interface GenericDAO<T extends Entity> {
    void save(T entity);
    T findById(String id);
    List<T> findAll();
    void update(T entity);
    void delete(String id);
}

public interface UserDAO extends GenericDAO<User> {
    User findByUsername(String username);
}
```

### Contract 3: Sự kiện Realtime Observer (Mảng 2 & Mảng 5 phối hợp)
```java
public interface AuctionObserver {
    void onNewBidPlaced(Auction auction, Bid newBid);
    void onAuctionStatusChanged(Auction auction, AuctionStatus newStatus);
}
```

---

## 1.2. Cây thư mục mã nguồn Tuần 7 (Từ `plan/week2.md`)
```text
online-auction-system/
├── .github/
│   ├── pull_request_template.md              # [Mảng 3] Template mở PR
│   └── ISSUE_TEMPLATE/
│       └── bug_report.md                     # [Mảng 3] Template báo lỗi
├── docs/
│   ├── STYLE_GUIDE.md                        # [Mảng 4] Quy chuẩn code, Javadoc, AAA test
│   └── javafx-guide.md                       # [Mảng 1] Hướng dẫn SceneBuilder & FXML
│
├── auction-server/
│   └── src/
│       ├── main/java/com/auction/server/
│       │   ├── ServerApp.java                # [Mảng 3] Khởi chạy Server cổng 8888
│       │   ├── domain/model/
│       │   │   ├── auction/
│       │   │   │   ├── Auction.java          # [Mảng 5 DUY NHẤT SỬA] Lock đa luồng & Notify Observer
│       │   │   │   ├── AuctionScheduler.java # [Mảng 5] Bộ đếm ngược đóng phiên tự động
│       │   │   │   └── observer/
│       │   │   │       └── AuctionObserver.java # [Mảng 2 TẠO INTERFACE]
│       │   │   └── user/
│       │   │       └── User.java             # [Mảng 4] Thêm salt (Constructor Overloading)
│       │   ├── network/                      # [Mảng 3]
│       │   │   ├── protocol/ (ActionType, ResponseStatus, Request, Response, MessageSerializer)
│       │   │   └── server/ (AuctionServer, ClientHandler)
│       │   ├── service/                      # [Mảng 4] UserService
│       │   ├── security/                     # [Mảng 4] PasswordHasher (SHA-256 + Salt)
│       │   └── persistence/                  # [Mảng 4] GenericDAO, UserDAO, UserDAOImpl, JsonStorage
│       └── test/java/com/auction/server/
│           ├── concurrency/                  # [Mảng 5] AuctionConcurrencyTest
│           ├── security/                     # [Mảng 4] PasswordHasherTest
│           ├── persistence/                  # [Mảng 4] UserDAOTest, JsonStorageTest
│           ├── network/                      # [Mảng 3] MessageSerializerTest, EchoClientTest
│           └── observer/                     # [Mảng 2] AuctionObserverTest
│
└── auction-client/
    └── src/main/
        ├── java/com/auction/client/
        │   ├── ClientApp.java                # [Mảng 1] Nạp LoginView.fxml bằng FXMLLoader
        │   ├── util/
        │   │   └── SceneManager.java         # [Mảng 1] Chuyển đổi qua lại các màn hình
        │   └── controller/
        │       ├── LoginController.java      # [Mảng 1]
        │       ├── RegisterController.java   # [Mảng 1]
        │       ├── SellerDashboardController.java # [Mảng 1]
        │       └── BiddingRoomController.java # [Mảng 2]
        └── resources/
            ├── css/
            │   └── theme.css                 # [Mảng 1] Dark theme hiện đại dùng chung
            └── fxml/
                ├── LoginView.fxml            # [Mảng 1]
                ├── RegisterView.fxml         # [Mảng 1]
                ├── SellerDashboard.fxml      # [Mảng 1]
                ├── AddItemDialog.fxml        # [Mảng 1]
                └── BiddingRoomView.fxml      # [Mảng 2]
```

---

## 1.3. Đặc tả nhiệm vụ chi tiết 5 thành viên trong Tuần 7 (Từ `plan/week2.md`)

### 👤 1. ĐỖ MẠNH HÙNG — Mảng 1: Client GUI & UX Base (Giao Diện Form & Quản Lý)
* **Nhánh làm việc:** `feature/w7-gui-base`
* **Công việc cụ thể:**
  1. Viết tài liệu `docs/javafx-guide.md` hướng dẫn SceneBuilder, đặt `fx:id`, kết nối Controller.
  2. Tạo `resources/css/theme.css`: Dark Theme hiện đại (màu nền `#1e1e2f`, `#27293d`, chữ trắng, nút bấm bo góc 8px hover cyan, style TableView và TextField).
  3. Viết `SceneManager.java` (Singleton) quản lý `switchScene(String fxmlPath)` trên Stage chính.
  4. Sửa `ClientApp.java` nạp `LoginView.fxml` làm màn hình mặc định.
  5. Dựng FXML và Controller:
     - `LoginView.fxml` + `LoginController.java`: Ô nhập username, password, nút Login, link chuyển Register.
     - `RegisterView.fxml` + `RegisterController.java`: Form đăng ký đầy đủ, ComboBox chọn vai trò.
     - `SellerDashboard.fxml` + `SellerDashboardController.java`: Bảng TableView sản phẩm của Seller, nút mở `AddItemDialog.fxml`.
* **Tiêu chí nghiệm thu:** Chạy `cd auction-client && mvn javafx:run` mở giao diện Dark Theme sắc nét, chuyển cảnh không lỗi FXML.

---

### 👤 2. LÊ ANH TUẤN — Mảng 2: Realtime UI & Observer (Phòng Đấu Giá Trực Tiếp)
* **Nhánh làm việc:** `feature/w7-realtime-bidding-room`
* **Quy tắc chống conflict:** CHỈ TẠO FILE MỚI, KHÔNG sửa trực tiếp vào `Auction.java`.
* **Công việc cụ thể:**
  1. Tạo `domain.model.auction.observer.AuctionObserver.java` gồm 2 hàm callback (`onNewBidPlaced`, `onAuctionStatusChanged`).
  2. Thiết kế `BiddingRoomView.fxml`: Thông tin sản phẩm, nhãn giá cao nhất to 32px `#lblCurrentPrice`, người dẫn đầu `#lblCurrentWinner`, đồng hồ đếm ngược `#lblCountdown`, bảng `TableView` lịch sử bid `#tblBidHistory`, các nút đặt giá nhanh `+1 Bước giá`, `+2 Bước giá`, nút `ĐẶT GIÁ NGAY`.
  3. Hiện thực `BiddingRoomController.java` implements `AuctionObserver`: Cài đặt callback nhận sự kiện tự động cập nhật Label giá và thêm dòng vào TableView. Bấm nút "Đặt giá thử" tự sinh bid mẫu để kiểm chứng nhảy số realtime.
  4. Viết `AuctionObserverTest.java`: Tạo mock observer, kiểm tra nhận đúng sự kiện khi callback được gọi.
* **Tiêu chí nghiệm thu:** `mvn test -Dtest="AuctionObserverTest"` PASS 100%.

---

### 👤 3. LÊ ĐỨC THIỆN SANG (Nhóm trưởng) — Mảng 3: Network & Protocol Core (Hạ Tầng Mạng & Server Socket)
* **Nhánh làm việc:** `feature/w7-network-protocol`
* **Công việc cụ thể:**
  1. Tạo `.github/pull_request_template.md` và `.github/ISSUE_TEMPLATE/bug_report.md`.
  2. Tạo `ServerApp.java` chứa `main()` in log chào mừng và gọi `AuctionServer.start()`.
  3. Thiết kế giao thức gói tin JSON: Enum `ActionType` (`LOGIN`, `REGISTER`, `GET_ACTIVE_AUCTIONS`, `PLACE_BID`, `CREATE_ITEM`, `PING`), Enum `ResponseStatus` (`OK`, `ERROR`, `UNAUTHORIZED`, `NOT_FOUND`), Class `Request`, Class `Response`, Class `MessageSerializer` (Gson).
  4. Hiện thực Socket Server đa luồng: `AuctionServer.java` (lắng nghe cổng 8888, CachedThreadPool), `ClientHandler.java` (implements Runnable, đọc JSON, gửi trả PONG an toàn luồng `synchronized sendMessage`).
  5. Viết kịch bản kiểm tra `EchoClientTest` hoặc console test: Kết nối cổng 8888 gửi PING nhận PONG.
* **Tiêu chí nghiệm thu:** Chạy Server cổng 8888, client console kết nối và nhận phản hồi JSON PONG hợp lệ.

---

### 👤 4. HOÀNG THẾ CÔNG MINH — Mảng 4: Persistence & Auth Core (Lưu Trữ Dữ Liệu, Mã Hóa & User Service)
* **Nhánh làm việc:** `feature/w7-persistence-auth`
* **Quy tắc bảo vệ code:** Bổ sung `salt` vào `User.java` bằng Constructor Overloading (`this.salt = "";`) để không phá vỡ test Tuần 1.
* **Công việc cụ thể:**
  1. Viết tài liệu quy chuẩn lập trình `docs/STYLE_GUIDE.md` (Naming, Formatting 4 spaces, Javadoc, Pattern AAA).
  2. Xây dựng `security.PasswordHasher`: `generateSalt()` sinh 16 bytes ngẫu nhiên, `hashPassword()` băm SHA-256, `verifyPassword()`.
  3. Xây dựng tầng CSDL: Interface `GenericDAO<T>`, `JsonStorage` đọc/ghi JSON bằng Gson pretty printing, `UserDAOImpl` lưu danh sách User vào file `auction-server/data/users.json`.
  4. Hiện thực `service.UserService`: `register()` kiểm tra trùng lặp, sinh salt, băm mật khẩu và lưu DAO; `login()` tìm user, verify mật khẩu băm.
  5. Viết Unit Test: `PasswordHasherTest.java`, `UserDAOTest.java`, `UserServiceTest.java`.
* **Tiêu chí nghiệm thu:** `mvn test -Dtest="PasswordHasherTest,UserDAOTest,UserServiceTest,UserTest"` PASS 100%; file `data/users.json` được tạo ra chứa mật khẩu băm.

---

### 👤 5. TRẦN MẠNH QUÂN — Mảng 5: Core Engine & Concurrency (Động Cơ Đấu Giá & Đa Luồng)
* **Nhánh làm việc:** `feature/w7-concurrency-engine`
* **Công việc cụ thể:**
  1. Nâng cấp An toàn Đa luồng cho `Auction.java`: Dùng `private final ReentrantLock lock = new ReentrantLock(true);` (Fair lock) bọc toàn bộ quá trình kiểm tra giá sàn, bước giá, số dư và ghi nhận `bidHistory`.
  2. Tích hợp Observer vào `Auction.java`: Khai báo `private final List<AuctionObserver> observers = new CopyOnWriteArrayList<>();`, các hàm `attachObserver`, `detachObserver`, `notifyNewBid`. Trong `placeBid()`, bid thành công lập tức gọi `notifyNewBid(newBid)`.
  3. Xây dựng `AuctionScheduler.java`: Dùng `ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);`. Hàm `scheduleAuctionEnd(Auction auction)` đếm ngược đến `endTime`, tự chuyển trạng thái `FINISHED` và kích hoạt `determineWinner()`.
  4. Viết `AuctionConcurrencyTest.java`: Dùng `ExecutorService` và `CountDownLatch`, cho 30–50 luồng đồng thời gọi `placeBid()` tại cùng một mili-giây, kiểm tra tuyệt đối không có lost update hay rollback giá.
* **Tiêu chí nghiệm thu:** `mvn test -Dtest="AuctionConcurrencyTest,AuctionTest"` PASS 100%.

---

## 1.4. Checklist & Review chéo Tuần 7 (Từ `plan/week2.md`)

| PR Author | Nhánh làm việc | Reviewer 1 (Chính) | Reviewer 2 (Phụ) |
| :--- | :--- | :---: | :---: |
| **Hùng** | `feature/w7-gui-base` | Tuấn | Minh |
| **Tuấn** | `feature/w7-realtime-bidding-room` | Quân | Hùng |
| **Sang** | `feature/w7-network-protocol` | Minh | Quân |
| **Minh** | `feature/w7-persistence-auth` | Sang | Hùng |
| **Quân** | `feature/w7-concurrency-engine` | Sang | Tuấn |

---
---

# 🛡️ TUẦN 8 — KIỂM THỬ, NGOẠI LỆ (CUSTOM EXCEPTIONS) & GUI MVC NÂNG CAO

> **Mục tiêu:** Kế thừa thành quả của 5 nhánh Tuần 7 đã độc lập chạy được, Tuần 8 tập trung chuẩn hóa hệ thống ngoại lệ chuyên biệt (**Custom Exceptions**), thay thế các lệnh trả về `return false` bằng Custom Exceptions, viết bộ **Unit Test JUnit 5** (`assertThrows`), và hoàn thiện kết nối MVC giữa FXML và Controller.

## 2.1. Bộ câu hỏi vấn đáp miệng Tuần 8
1. *Tại sao trong hệ thống thực tế cần định nghĩa Custom Exceptions thay vì ném RuntimeException chung chung?*  
   $\rightarrow$ Giúp phân tách chính xác lỗi nghiệp vụ (như đặt giá sai bước giá, phiên đã đóng) với lỗi hệ thống, giúp Controller bắt đúng loại lỗi để hiển thị thông báo Alert thân thiện thay vì làm sập ứng dụng.
2. *Hàm `assertThrows()` trong JUnit 5 hoạt động ra sao?*  
   $\rightarrow$ `assertThrows(ExpectedException.class, () -> executable)` thực thi đoạn mã; nếu ném đúng class ngoại lệ mong muốn thì test PASS, nếu không ném hoặc ném sai loại thì test FAIL.
3. *Mô hình MVC trong ứng dụng JavaFX được tổ chức thế nào?*  
   $\rightarrow$ View là file FXML; Controller bắt sự kiện và gọi Model/Service; Model chứa dữ liệu và nghiệp vụ thuần túy, hoàn toàn không dính mã giao diện.

---

## 2.2. Cây thư mục mã nguồn phát triển mới trong Tuần 8
```text
online-auction-system/
├── auction-server/
│   └── src/
│       ├── main/java/com/auction/server/
│       │   ├── exception/                       # [Minh] Package Custom Exceptions
│       │   │   ├── AuctionException.java        # Exception gốc
│       │   │   ├── InvalidBidException.java     # Lỗi giá đặt không hợp lệ
│       │   │   ├── AuctionClosedException.java  # Lỗi đấu giá khi phiên đã kết thúc
│       │   │   └── AuthenticationException.java # Lỗi đăng nhập thất bại
│       │   ├── domain/model/auction/
│       │   │   └── Auction.java                 # [Quân] Ném InvalidBidException & AuctionClosedException
│       │   └── service/
│       │       └── UserService.java             # [Minh] Ném AuthenticationException khi sai mật khẩu
│       └── test/java/com/auction/server/
│           └── exception/
│               └── ExceptionTest.java           # [Sang] Bộ test JUnit 5 assertThrows cho ngoại lệ
└── auction-client/
    └── src/main/java/com/auction/client/controller/
        ├── LoginController.java                 # [Hùng] Bắt AuthenticationException hiển thị label đỏ
        └── BiddingRoomController.java           # [Tuấn] Bắt InvalidBidException hiển thị popup Alert
```

---

## 2.3. Đặc tả nhiệm vụ chi tiết 5 thành viên trong Tuần 8

### 👤 1. HOÀNG THẾ CÔNG MINH — Cụm Custom Exceptions & Cập Nhật `UserService`
* **Nhánh làm việc:** `feature/w8-minh/custom-exceptions-auth`
* **Công việc cụ thể:**
  1. Tạo package `com.auction.server.exception` gồm 4 lớp ngoại lệ:
     - `AuctionException.java` (`extends RuntimeException`): Lớp cha gốc.
     - `InvalidBidException.java` (`extends AuctionException`): Ném ra khi giá đặt nhỏ hơn giá sàn, nhỏ hơn bước giá, hoặc số dư không đủ.
     - `AuctionClosedException.java` (`extends AuctionException`): Ném ra khi đặt giá vào phiên không ở trạng thái `RUNNING`.
     - `AuthenticationException.java` (`extends AuctionException`): Ném ra khi đăng nhập thất bại.
  2. Cập nhật `UserService.java`:
     - Khi đăng nhập sai mật khẩu hoặc không tìm thấy user $\rightarrow$ ném `AuthenticationException("Sai tên đăng nhập hoặc mật khẩu!")`.
     - Khi đăng ký trùng username $\rightarrow$ ném `AuctionException("Tên đăng nhập đã tồn tại!")`.

---

### 👤 2. TRẦN MẠNH QUÂN — Refactor Ném Ngoại Lệ Trong `Auction.java` & Củng Cố Scheduler
* **Nhánh làm việc:** `feature/w8-quan/exception-refactor-scheduler`
* **Công việc cụ thể:**
  1. Refactor hàm `placeBid()` trong `Auction.java`:
     - Thay thế việc `return false` bằng cách ném Custom Exceptions:
       - Nếu phiên không `RUNNING` $\rightarrow$ ném `AuctionClosedException("Phiên đấu giá đang đóng!")`.
       - Nếu số tiền nhỏ hơn bước giá hoặc số dư không đủ $\rightarrow$ ném `InvalidBidException("Giá đặt không hợp lệ hoặc số dư không đủ!")`.
  2. Củng cố `AuctionScheduler.java`: Đảm bảo khi đếm ngược hết giờ, phiên chuyển trạng thái sang `FINISHED`, kích hoạt `determineWinner()`, và nếu có ai cố đặt giá sau thời điểm này sẽ bị ném ngay `AuctionClosedException`.

---

### 👤 3. LÊ ĐỨC THIỆN SANG — Bộ Kiểm Thử Ngoại Lệ JUnit 5 & Tái Cấu Trúc SOLID
* **Nhánh làm việc:** `feature/w8-sang/exception-test-solid`
* **Công việc cụ thể:**
  1. Tạo file `com.auction.server.exception.ExceptionTest.java`:
     - Viết tối thiểu 8 ca kiểm thử bắt buộc sử dụng `assertThrows`:
       - `assertThrows(InvalidBidException.class, () -> auction.placeBid(bidder, 50.0));`
       - `assertThrows(AuctionClosedException.class, () -> auction.placeBid(bidder, 200.0));`
       - `assertThrows(AuthenticationException.class, () -> userService.login("tuan", "wrongpass"));`
  2. Rà soát chuẩn hóa SOLID theo tài liệu `docs/STYLE_GUIDE.md` của Minh: Đảm bảo toàn bộ thuộc tính trong các Entity đều là `private`, loại bỏ mã nguồn thừa.
* **Lệnh kiểm tra:** `mvn test -Dtest="ExceptionTest"` $\rightarrow$ **PASS 100%**.

---

### 👤 4. ĐỖ MẠNH HÙNG — Hoàn Thiện MVC Cho Các Form Giao Diện
* **Nhánh làm việc:** `feature/w8-hung/mvc-form-views`
* **Công việc cụ thể:**
  1. Hoàn thiện `LoginController.java`: Khi gọi `UserService.login()` thất bại, bắt `AuthenticationException` và hiển thị chữ đỏ thông báo lỗi lên `#lblError`. Đăng nhập thành công chuyển sang `SellerDashboard.fxml`.
  2. Hoàn thiện `RegisterController.java`: Kiểm tra form đăng ký, bắt ngoại lệ nếu username đã tồn tại.
  3. Hoàn thiện `SellerDashboardController.java`: Đổ danh sách sản phẩm vào TableView, mở hộp thoại `AddItemDialog.fxml` để thêm sản phẩm mới tương ứng với `ItemFactory`.

---

### 👤 5. LÊ ANH TUẤN — Xử Lý Ngoại Lệ & Đồng Hồ Đếm Ngược Trên UI Phòng Đấu Giá
* **Nhánh làm việc:** `feature/w8-tuan/mvc-bidding-room-controller`
* **Công việc cụ thể:**
  1. Cập nhật `BiddingRoomController.java`:
     - Bắt ngoại lệ từ Model khi bấm nút `ĐẶT GIÁ`:
       ```java
       try {
           double amount = Double.parseDouble(txtBidAmount.getText().trim());
           auction.placeBid(currentUser, amount);
       } catch (NumberFormatException e) {
           showAlert("Lỗi nhập liệu", "Vui lòng nhập một số tiền hợp lệ!");
       } catch (InvalidBidException | AuctionClosedException e) {
           showAlert("Từ chối đặt giá", e.getMessage());
       }
       ```
     - Hàm `showAlert(String title, String content)`: Mở cửa sổ popup `Alert(AlertType.WARNING)`.
  2. Đồng hồ đếm ngược trực quan: Sử dụng `javafx.animation.Timeline` chạy mỗi 1 giây cập nhật thời gian còn lại lên `#lblCountdown`. Khi còn dưới 60 giây, đổi chữ sang màu đỏ cảnh báo.

---

## 2.4. Checklist & Review chéo Tuần 8

| PR Author | Nhánh làm việc | Reviewer 1 (Chính) | Reviewer 2 (Phụ) |
| :--- | :--- | :---: | :---: |
| **Minh** | `feature/w8-minh/custom-exceptions-auth` | Sang | Hùng |
| **Quân** | `feature/w8-quan/exception-refactor-scheduler` | Minh | Tuấn |
| **Sang** | `feature/w8-sang/exception-test-solid` | Quân | Tuấn |
| **Hùng** | `feature/w8-hung/mvc-form-views` | Tuấn | Sang |
| **Tuấn** | `feature/w8-tuan/mvc-bidding-room-controller` | Hùng | Quân |

---
---

# 🌐 TUẦN 9 — TÍCH HỢP HỆ THỐNG, MẠNG SOCKET & LƯU TRỮ DỮ LIỆU

> **Mục tiêu:** Kết nối hoàn chỉnh hạ tầng mạng Socket Client–Server End-to-End, triển khai cơ chế **Broadcast Realtime qua Socket** (`Platform.runLater`), lưu trữ CSDL phiên đấu giá ra file JSON, và tích hợp **Checkstyle Google Java Style** vào Maven + GitHub Actions.

## 3.1. Bộ câu hỏi vấn đáp miệng Tuần 9
1. *Cơ chế hoạt động của Socket Client-Server đa luồng?*  
   $\rightarrow$ `ServerSocket.accept()` lắng nghe tại cổng 8888. Khi có Client kết nối, Server bàn giao Socket đó cho 1 `ClientHandler` chạy trên Thread Pool để phục vụ song song, không làm nghẽn luồng tiếp nhận chính.
2. *Tại sao khi cập nhật giao diện JavaFX từ luồng Socket bắt buộc phải dùng `Platform.runLater()`?*  
   $\rightarrow$ JavaFX tuân thủ nguyên tắc *Single-Threaded UI Rule*. Mọi thao tác vẽ hoặc thay đổi component bắt buộc phải diễn ra trên luồng chính *JavaFX Application Thread*. Dữ liệu mạng chạy trên luồng ngầm, do đó phải đưa tác vụ vào hàng đợi của luồng UI qua `Platform.runLater()`.

---

## 3.2. Cây thư mục mã nguồn phát triển mới trong Tuần 9
```text
online-auction-system/
├── pom.xml                                      # [Sang] Thêm maven-checkstyle-plugin
├── .github/workflows/
│   └── maven-ci.yml                             # [Sang] CI kiểm tra cả test và checkstyle
├── auction-server/
│   ├── data/                                    # [Minh] Thư mục chứa CSDL JSON
│   │   ├── users.json                           # Đã có từ Tuần 7
│   │   └── auctions.json                        # [Minh - Mới] Danh sách phiên và lịch sử bid
│   └── src/
│       ├── main/java/com/auction/server/
│       │   ├── persistence/
│       │   │   └── AuctionDAOImpl.java          # [Minh - Mới] Đọc ghi data/auctions.json
│       │   └── network/server/
│       │       └── AuctionServer.java           # [Sang - Hoàn thiện] Thêm hàm broadcast() push realtime
│       └── test/java/com/auction/server/
│           └── concurrency/
│               └── AuctionConcurrencyTest.java  # [Quân - Nâng cấp] Test 50 luồng CountDownLatch
└── auction-client/
    └── src/main/java/com/auction/client/
        ├── network/
        │   └── AuctionClient.java               # [Tuấn - Mới] Socket Client chạy ngầm nhận push
        └── controller/
            └── (Các Controller kết nối với AuctionClient gửi Request thật qua Socket) # [Hùng & Tuấn]
```

---

## 3.3. Đặc tả nhiệm vụ chi tiết 5 thành viên trong Tuần 9

### 👤 1. LÊ ĐỨC THIỆN SANG — Mảng 3: Hoàn Thiện Socket Server, Broadcast & Checkstyle
* **Nhánh làm việc:** `feature/w9-sang/socket-server-checkstyle`
* **Công việc cụ thể:**
  1. Nâng cấp `AuctionServer.java`:
     - Quản lý danh sách kết nối: `List<ClientHandler> clients = new CopyOnWriteArrayList<>();`.
     - Hàm `broadcast(Response res)`: Duyệt `clients` gửi gói tin cập nhật tới tất cả Client đang kết nối.
  2. Nâng cấp `ClientHandler.java`: Tiếp nhận `Request(ActionType.PLACE_BID)`, gọi `Auction.placeBid()`, và kích hoạt `server.broadcast(newBidResponse)`.
  3. Cấu hình `maven-checkstyle-plugin` (Google Java Style) vào `pom.xml` và cập nhật file CI `.github/workflows/maven-ci.yml`.
* **Lệnh kiểm tra:** `mvn checkstyle:check` $\rightarrow$ **BUILD SUCCESS (0 lỗi format)**.

---

### 👤 2. LÊ ANH TUẤN — Mảng 2: Socket Client & Realtime Push Cập Nhật Giao Diện
* **Nhánh làm việc:** `feature/w9-tuan/socket-client-realtime`
* **Công việc cụ thể:**
  1. Tạo lớp `com.auction.client.network.AuctionClient.java`:
     - Mở kết nối Socket tới Server cổng 8888.
     - Luồng ngầm liên tục đọc gói tin push từ Server.
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

### 👤 3. HOÀNG THẾ CÔNG MINH — Mảng 4: Mở Rộng Lưu Trữ CSDL `auctions.json`
* **Nhánh làm việc:** `feature/w9-minh/persistence-security-dao`
* **Công việc cụ thể:**
  1. Hiện thực `AuctionDAOImpl.java`:
     - Đọc/ghi danh sách phiên đấu giá và `bidHistory` vào file `auction-server/data/auctions.json`.
     - Khi khởi động Server, tự động tải dữ liệu từ file vào bộ nhớ `AuctionManager`.
* **Lệnh kiểm tra:** `mvn test -Dtest="UserDAOTest"` PASS 100%.

---

### 👤 4. TRẦN MẠNH QUÂN — Mảng 5: Bộ Kiểm Thử Concurrency Áp Lực Cao (50 Threads)
* **Nhánh làm việc:** `feature/w9-quan/concurrency-stress-test`
* **Công việc cụ thể:**
  1. Nâng cấp `AuctionConcurrencyTest.java`:
     - Tạo 1 phiên đấu giá, 50 Bidder cùng số dư lớn.
     - Dùng `CountDownLatch(50)` kích hoạt 50 luồng xuất phát tại cùng một mili-giây.
     - Assertions kiểm tra: Không race condition, không rollback giá, giá cao nhất cuối cùng khớp với lịch sử.
* **Lệnh kiểm tra:** `mvn test -Dtest="AuctionConcurrencyTest"` $\rightarrow$ **PASS 100%**.

---

### 👤 5. ĐỖ MẠNH HÙNG — Mảng 1: Kết Nối Toàn Bộ GUI Với Socket Client
* **Nhánh làm việc:** `feature/w9-hung/gui-socket-integration`
* **Công việc cụ thể:**
  1. Chuyển đổi các Controller (`LoginController`, `RegisterController`, `SellerDashboardController`) sang gửi `Request` qua `AuctionClient`:
     - Bấm Đăng nhập $\rightarrow$ gửi `Request(ActionType.LOGIN)` lên Server, đợi `Response(status=OK)` mới chuyển màn hình.
     - Tạo sản phẩm mới $\rightarrow$ gửi `Request(ActionType.CREATE_ITEM)` lên Server để lưu vào file JSON.
  2. Xử lý lỗi ngắt kết nối: Hiển thị hộp thoại cảnh báo nếu Server chưa bật hoặc bị mất mạng đột ngột.

---

## 3.4. Checklist & Review chéo Tuần 9

| PR Author | Nhánh làm việc | Reviewer 1 (Chính) | Reviewer 2 (Phụ) |
| :--- | :--- | :---: | :---: |
| **Sang** | `feature/w9-sang/socket-server-checkstyle` | Minh | Quân |
| **Tuấn** | `feature/w9-tuan/socket-client-realtime` | Quân | Hùng |
| **Minh** | `feature/w9-minh/persistence-security-dao` | Sang | Hùng |
| **Quân** | `feature/w9-quan/concurrency-stress-test` | Sang | Tuấn |
| **Hùng** | `feature/w9-hung/gui-socket-integration` | Tuấn | Minh |
