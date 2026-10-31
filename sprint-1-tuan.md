# 📋 KẾ HOẠCH TỔNG HỢP 1 TUẦN NƯỚC RÚT (01/11 – 07/11/2026)
## Dự Án: Hệ Thống Đấu Giá Trực Tuyến (Online Auction System) — Nhóm 1
### Mục tiêu: Hoàn Tất Tuần 2 Đang Dở Dang $\rightarrow$ Tiến Thẳng Hoàn Thiện Hạng Mục Tuần 8, 9 Trước Giờ Chấm 07/11

> **Môn học:** Lập trình nâng cao (2627I_UET.CS2043_6)  
> **Thời gian tác chiến:** Đúng 7 ngày (Từ Chủ Nhật 01/11/2026 đến Thứ Bảy 07/11/2026)  
> **Điểm xuất phát thực tế:** Nhóm đang thực hiện dở dang kế hoạch Tuần 2 ([plan/week2.md](file:///E:/Dai-hoc/HK3/Lap-trinh-nang-cao/Bai-tap-lon/Lap-trinh-nang-cao-BTL-nhom-1/plan/week2.md)) trên 5 nhánh `feature/w7-...`.  
> **Họp Merge Tuần 2:** Tối Chủ Nhật ngày 01/11/2026 (Chốt hạ nền móng Tuần 7).  
> **Họp Merge Toàn Bộ Tuần 8 & 9:** Tối Thứ Năm ngày 05/11/2026 (Hội tụ toàn hệ thống vào `dev`).  
> **Tổng Duyệt Demo & Vấn Đáp Thử:** Tối Thứ Sáu ngày 06/11/2026.  
> **Thời điểm Giảng viên chấm tiến độ:** Thứ Bảy ngày 07/11/2026.  
> **Chiến lược:** Giữ nguyên 100% 5 Mảng Chuyên Môn Độc Lập đã nhận từ Tuần 2, hoàn thiện theo hình thức cuốn chiếu từng ngày.

---

# 📌 PHẦN 1 — PHẦN CHUNG: TỰ HỌC NỀN TẢNG & VẤN ĐÁP LÝ THUYẾT

Mỗi thành viên phụ trách trục nào phải nghiên cứu sâu mảng đó và nắm chắc các câu trả lời dưới đây để bảo vệ đồ án:

### 1.1. Bảng kỹ năng tự học trọng tâm

| # | Nội dung tự học | Thành viên cần nhất | Đầu ra kiểm tra (Demo khi họp nhóm) |
| :-: | :--- | :---: | :--- |
| **1** | **Gson:** Chuyển đổi `Java Object ↔ JSON` | **Minh, Sang** | Đọc/ghi được `Request`, `Response`, `User`, `Auction` ra chuỗi JSON. |
| **2** | **Java Socket đa luồng:** `ServerSocket` + `Socket` + `ExecutorService` | **Sang, Tuấn** | Server lắng nghe cổng 8888, chấp nhận nhiều Client kết nối song song. |
| **3** | **JavaFX FXML + SceneBuilder + CSS Dark Theme** | **Hùng, Tuấn** | Mở được file FXML bằng SceneBuilder, gán CSS Dark theme bo góc hiện đại. |
| **4** | **Password Hashing:** Băm SHA-256 kết hợp 16 bytes Salt ngẫu nhiên | **Minh** | Hàm `hashPassword()` và `verifyPassword()` chạy test pass 100%. |
| **5** | **Concurrency & Fair Lock:** `ReentrantLock(true)` | **Quân** | Viết test 50 luồng đồng thời gọi `placeBid()` không bị mất dữ liệu hay rollback giá. |
| **6** | **Observer Pattern qua Socket & `Platform.runLater()`** | **Tuấn, Sang** | Khi Server push gói tin giá mới $\rightarrow$ Client tự động nhảy số giao diện tức thời. |
| **7** | **Coding Convention:** Google Java Style Guide & Checkstyle | **Sang, Cả nhóm** | Lệnh `mvn checkstyle:check` chạy sạch lỗi cảnh báo format. |

---

### 1.2. Bộ câu hỏi vấn đáp miệng "Sống còn" (Chuẩn bị sẵn câu trả lời)

1. **Câu hỏi về Concurrency (Quân & Sang):**
   * *Hỏi:* Tại sao trong phương thức `placeBid()` phải sử dụng `new ReentrantLock(true)` thay vì từ khóa `synchronized`?
   * *Trả lời:* `synchronized` là khóa ngầm định của JVM không đảm bảo thứ tự luồng (luồng đến sau có thể chen ngang lấy khóa trước, gây ra hiện tượng luồng bị đói tài nguyên - *starvation*). `new ReentrantLock(true)` là **Fair Lock (Khóa công bằng)**, đảm bảo luồng nào gửi yêu cầu đặt giá trước sẽ được xử lý trước theo hàng đợi FIFO, hoàn toàn công bằng cho người đấu giá và ngăn chặn tuyệt đối lỗi *lost update* hay *rollback* giá.
2. **Câu hỏi về Lập trình Mạng Socket (Sang & Tuấn):**
   * *Hỏi:* Tại sao Server phải dùng `ExecutorService` để phục vụ từng Client trong một luồng riêng (`ClientHandler`)?
   * *Trả lời:* Vì thao tác đọc dữ liệu từ Socket (`in.readLine()`) là thao tác chặn luồng (*blocking I/O*). Nếu xử lý trên một luồng duy nhất, khi 1 Client đang kết nối hoặc bị lag mạng thì toàn bộ các Client khác sẽ bị treo cứng. Việc sử dụng Thread Pool (`ExecutorService`) cho phép Server tiếp nhận và phục vụ đồng thời hàng chục Client độc lập mà không làm nghẽn luồng tiếp nhận `accept()` chính.
3. **Câu hỏi về Giao diện JavaFX (Hùng & Tuấn):**
   * *Hỏi:* Tại sao khi cập nhật giao diện JavaFX từ luồng Socket nhận tin nhắn bắt buộc phải dùng `Platform.runLater()`? Điều gì xảy ra nếu vi phạm?
   * *Trả lời:* JavaFX tuân thủ nguyên tắc *Single-Threaded UI Rule*. Toàn bộ thao tác vẽ hoặc thay đổi component bắt buộc phải diễn ra trên luồng chính *JavaFX Application Thread*. Dữ liệu Socket chạy trên luồng ngầm (*background thread*). Nếu gọi trực tiếp thay đổi UI từ luồng ngầm sẽ gây ra lỗi `IllegalStateException: Not on FX application thread`. Bọc trong `Platform.runLater()` sẽ đưa tác vụ vào hàng đợi của luồng UI thực thi an toàn.
4. **Câu hỏi về Bảo mật & Lưu trữ Dữ liệu (Minh):**
   * *Hỏi:* Tại sao không được lưu mật khẩu dạng thô (plain-text) và tại sao khi băm SHA-256 lại bắt buộc phải thêm chuỗi Salt?
   * *Trả lời:* Lưu mật khẩu dạng thô sẽ khiến tài khoản người dùng bị lộ hoàn toàn nếu file CSDL bị đánh cắp. Băm SHA-256 là hàm một chiều giúp giấu mật khẩu gốc. Tuy nhiên, nếu chỉ băm SHA-256 thuần, kẻ tấn công có thể dùng bảng tra cứu dựng sẵn (*Rainbow Table*) để giải mã ngược. Việc thêm chuỗi Salt (16 bytes ngẫu nhiên từ `SecureRandom`) đảm bảo rằng hai người dùng có cùng mật khẩu thì chuỗi băm lưu trữ vẫn hoàn toàn khác nhau, vô hiệu hóa hoàn toàn Rainbow Table.
5. **Câu hỏi về Design Patterns (Cả nhóm):**
   * *Hỏi:* Hãy chỉ rõ các Design Pattern áp dụng trong dự án và vị trí xuất hiện trong mã nguồn?
   * *Trả lời:*
     - **Singleton Pattern:** `AuctionManager.java` (quản lý danh sách phiên đấu giá tập trung duy nhất) và `SceneManager.java` (quản lý chuyển đổi Stage JavaFX duy nhất).
     - **Factory Method Pattern:** `ItemFactory.java` (khởi tạo linh hoạt các lớp con của `Item`: `Electronics`, `Art`, `Vehicle` dựa trên `ItemType` và Map tham số).
     - **Observer Pattern:** `Auction.java` đóng vai trò là Subject; `AuctionObserver.java` là Interface lắng nghe; khi có lượt đặt giá mới hoặc phiên đóng, Subject duyệt `observers` và gọi `onNewBidPlaced` / `onAuctionStatusChanged`.
     - **MVC (Model-View-Controller):** Phía Client tách rời file FXML (View), Controller JavaFX (Điều khiển), và các lớp Entity/Service (Model).

---

# 🌳 PHẦN 2 — CÂY THƯ MỤC TOÀN DIỆN CỦA DỰ ÁN KHI HOÀN THÀNH

```text
online-auction-system/
├── pom.xml                                      # [Sang] Quản lý Gson, JUnit 5, JavaFX & Checkstyle Plugin
├── .github/workflows/
│   └── maven-ci.yml                             # [Sang] CI tự động: Biên dịch + Checkstyle + Chạy toàn bộ Unit Tests
├── docs/
│   ├── ClassDiagram.png                         # Sơ đồ lớp UML hệ thống
│   ├── STYLE_GUIDE.md                           # [Minh - Tuần 2] Quy chuẩn lập trình
│   └── javafx-guide.md                          # [Hùng - Tuần 2] Hướng dẫn SceneBuilder & FXML
│
├── auction-server/                              # [MODULE BACKEND: NGHIỆP VỤ, MẠNG & CSDL]
│   ├── pom.xml
│   ├── data/                                    # [Minh] Thư mục lưu trữ CSDL JSON bền vững
│   │   ├── users.json                           # Danh sách tài khoản đã băm mật khẩu
│   │   └── auctions.json                        # Danh sách phiên và lịch sử đặt giá
│   └── src/
│       ├── main/java/com/auction/server/
│       │   ├── ServerApp.java                   # [Sang - Tuần 2] Khởi động Server cổng 8888
│       │   ├── domain/model/                    # Tầng thực thể nghiệp vụ (Đã có từ Tuần 1)
│       │   │   ├── Entity.java
│       │   │   ├── user/ (User, Role, Bidder, Seller, Admin)
│       │   │   ├── item/ (Item, ItemType, Electronics, Art, Vehicle, ItemFactory)
│       │   │   └── auction/
│       │   │       ├── Auction.java             # [Quân - Tuần 2 & 8] ReentrantLock & Ném Exceptions
│       │   │       ├── AuctionStatus.java       # [Minh] Trạng thái vòng đời phiên
│       │   │       ├── AuctionManager.java      # [Sang] Singleton quản lý phiên & hủy phiên
│       │   │       ├── Bid.java                 # Đối tượng đặt giá bất biến
│       │   │       ├── AuctionScheduler.java    # [Quân - Tuần 2 & 8] Timer tự động đóng phiên
│       │   │       └── observer/
│       │   │           └── AuctionObserver.java # [Tuấn - Tuần 2] Interface Observer chuẩn
│       │   ├── exception/                       # [Minh - Tuần 8] Package Custom Exceptions
│       │   │   ├── AuctionException.java        # Exception cha gốc
│       │   │   ├── InvalidBidException.java     # Lỗi giá đặt không hợp lệ / nhỏ hơn bước giá
│       │   │   ├── AuctionClosedException.java  # Lỗi đặt giá khi phiên đã kết thúc / bị hủy
│       │   │   └── AuthenticationException.java # Lỗi đăng nhập thất bại
│       │   ├── security/                        # [Minh - Tuần 2]
│       │   │   └── PasswordHasher.java          # Tiện ích băm SHA-256 + 16 bytes Salt
│       │   ├── persistence/                     # [Minh - Tuần 2 & 9] Tầng lưu trữ DAO
│       │   │   ├── GenericDAO.java              # Interface CRUD tổng quát
│       │   │   ├── UserDAOImpl.java             # Đọc/ghi users.json
│       │   │   ├── AuctionDAOImpl.java          # Đọc/ghi auctions.json
│       │   │   └── JsonStorage.java             # Tiện ích đọc/ghi file Gson Pretty
│       │   ├── service/                         # [Minh - Tuần 2 & 8]
│       │   │   └── UserService.java             # Dịch vụ đăng ký / đăng nhập người dùng
│       │   └── network/                         # [Sang - Tuần 2 & 9]
│       │       ├── protocol/
│       │       │   ├── ActionType.java          # LOGIN, REGISTER, PLACE_BID, NEW_BID...
│       │       │   ├── ResponseStatus.java      # OK, ERROR, UNAUTHORIZED, NOT_FOUND
│       │       │   ├── Request.java             # Gói tin Client gửi lên
│       │       │   ├── Response.java            # Gói tin Server trả về
│       │       │   └── MessageSerializer.java   # Tuần tự hóa JSON qua Gson
│       │       └── server/
│       │           ├── AuctionServer.java       # ServerSocket lắng nghe cổng 8888 đa luồng
│       │           └── ClientHandler.java       # Luồng riêng phục vụ từng Client + Broadcast
│       └── test/java/com/auction/server/
│           ├── concurrency/
│           │   └── AuctionConcurrencyTest.java  # [Quân] Test 50 threads đặt giá đồng thời
│           ├── exception/
│           │   └── ExceptionTest.java           # [Sang] Test các kịch bản assertThrows ngoại lệ
│           ├── persistence/
│           │   └── UserDAOTest.java             # [Minh] Test CRUD ghi file JSON
│           ├── security/
│           │   └── PasswordHasherTest.java      # [Minh] Test băm mật khẩu và verify
│           └── network/
│               └── MessageSerializerTest.java   # [Sang] Test đóng gói và giải mã gói tin JSON
│
└── auction-client/                              # [MODULE FRONTEND: GIAO DIỆN JAVAFX & CLIENT SOCKET]
    ├── pom.xml
    └── src/main/
        ├── java/com/auction/client/
        │   ├── Launcher.java                    # Lớp mồi khởi chạy tránh lỗi runtime JavaFX
        │   ├── ClientApp.java                   # Điểm khởi đầu ứng dụng JavaFX, nạp SceneManager
        │   ├── network/                         # [Tuấn - Tuần 9]
        │   │   └── AuctionClient.java           # Luồng Socket Client ngầm lắng nghe Server push
        │   ├── util/                            # [Hùng - Tuần 2]
        │   │   └── SceneManager.java            # Singleton điều hướng chuyển màn hình
        │   └── controller/                      # [Hùng & Tuấn] Các Controller FXML (MVC)
        │       ├── LoginController.java         # [Hùng - Tuần 2 & 8] Điều khiển Đăng nhập
        │       ├── RegisterController.java      # [Hùng - Tuần 2 & 8] Điều khiển Đăng ký
        │       ├── SellerDashboardController.java # [Hùng - Tuần 2 & 8] Điều khiển Quản lý hàng Seller
        │       └── BiddingRoomController.java   # [Tuấn - Tuần 2 & 8 & 9] Điều khiển Phòng đấu giá
        └── resources/
            ├── css/
            │   └── theme.css                    # [Hùng - Tuần 2] Dark Theme CSS hiện đại
            └── fxml/
                ├── LoginView.fxml               # [Hùng - Tuần 2] Giao diện đăng nhập
                ├── RegisterView.fxml            # [Hùng - Tuần 2] Giao diện đăng ký
                ├── SellerDashboard.fxml         # [Hùng - Tuần 2] Giao diện bảng điều khiển Seller
                ├── AddItemDialog.fxml           # [Hùng - Tuần 2] Hộp thoại tạo sản phẩm
                └── BiddingRoomView.fxml         # [Tuấn - Tuần 2] Giao diện phòng đấu giá trực tiếp
```

---

# 🔨 PHẦN 3 — ĐẶC TẢ NHIỆM VỤ CÁ NHÂN CHI TIẾT (5 THÀNH VIÊN)

> [!IMPORTANT]
> Từng thành viên đối chiếu công việc đang làm dở của Tuần 2 (`plan/week2.md`) để chốt hạ vào Chủ Nhật (01/11), sau đó nâng cấp thẳng lên Tuần 8 và Tuần 9 theo đúng lộ trình dưới đây:

---

### 👤 1. TRẦN MẠNH QUÂN — Mảng 5: Concurrency Engine & Auction Lifecycle
* **Nhánh làm việc:** `feature/w7-concurrency-engine`
* **Công việc Tuần 2 đang làm dở (Chốt nộp ngày 01/11):**
  1. Nâng cấp hàm `placeBid()` trong `Auction.java`: Thay `synchronized` bằng `private final ReentrantLock lock = new ReentrantLock(true);` (Fair lock xếp hàng FIFO).
  2. Bổ sung danh sách Observer: `private final List<AuctionObserver> observers = new CopyOnWriteArrayList<>();`, các hàm `attachObserver()`, `detachObserver()`, `notifyNewBid()`.
  3. Viết `AuctionScheduler.java` dùng `ScheduledExecutorService` đếm ngược đến `endTime`.
  4. Viết `AuctionConcurrencyTest.java` (test 20–30 threads).
* **Nâng cấp Tuần 8 & Tuần 9 (Thực hiện từ 02/11 đến 05/11):**
  1. *Tuần 8:* Refactor `Auction.placeBid()`: ném `InvalidBidException` nếu giá đặt nhỏ hơn bước giá hoặc số dư không đủ; ném `AuctionClosedException` nếu phiên không ở trạng thái `RUNNING`.
  2. *Tuần 8:* Củng cố `AuctionScheduler`: khi hết giờ tự chuyển trạng thái `FINISHED`, kích hoạt `determineWinner()`, ai cố đặt giá sau giờ này bị ném `AuctionClosedException`.
  3. *Tuần 9:* Nâng cấp `AuctionConcurrencyTest.java` lên **50 threads đồng thời** chạy qua `CountDownLatch(50)`, kiểm tra giá tăng dần đơn điệu và không lost update.
* **Lệnh kiểm tra đầu ra:**
  ```bash
  mvn test -Dtest="AuctionConcurrencyTest,AuctionTest"
  ```

---

### 👤 2. LÊ ANH TUẤN — Mảng 2: Realtime UI & Socket Client Receiver
* **Nhánh làm việc:** `feature/w7-realtime-bidding-room`
* **Công việc Tuần 2 đang làm dở (Chốt nộp ngày 01/11):**
  1. Tạo interface `AuctionObserver.java` trong `com.auction.server.domain.model.auction.observer`.
  2. Dựng layout `resources/fxml/BiddingRoomView.fxml`: Label giá cao nhất `#lblCurrentPrice` (32px cyan), người dẫn đầu `#lblCurrentWinner`, đồng hồ `#lblCountdown`, bảng `TableView` lịch sử `#tblBidHistory`, các nút đặt giá nhanh, nút `ĐẶT GIÁ NGAY`.
  3. Viết `BiddingRoomController.java` implements `AuctionObserver`: Callback `onNewBidPlaced` cập nhật Label và TableView. Tạo nút "Đặt giá thử" trên UI để test nhảy số.
  4. Viết `AuctionObserverTest.java`.
* **Nâng cấp Tuần 8 & Tuần 9 (Thực hiện từ 02/11 đến 05/11):**
  1. *Tuần 8:* Bắt ngoại lệ từ Model: Khi người dùng nhập sai, Controller bắt `InvalidBidException` hoặc `AuctionClosedException` và mở popup `Alert(AlertType.WARNING)` hiển thị thông báo lỗi thân thiện.
  2. *Tuần 8:* Xây dựng đồng hồ đếm ngược trực quan bằng `javafx.animation.Timeline` chạy mỗi 1 giây; khi còn dưới 60 giây chuyển chữ sang màu đỏ.
  3. *Tuần 9:* Viết lớp `com.auction.client.network.AuctionClient.java`: Luồng ngầm Socket kết nối tới Server cổng 8888, nhận gói tin JSON `NEW_BID_NOTIFICATION` $\rightarrow$ gọi `Platform.runLater()` cập nhật giao diện JavaFX realtime.
* **Lệnh kiểm tra đầu ra:**
  ```bash
  mvn test -Dtest="AuctionObserverTest"
  ```

---

### 👤 3. HOÀNG THẾ CÔNG MINH — Mảng 4: Persistence, Security & Custom Exceptions
* **Nhánh làm việc:** `feature/w7-persistence-auth`
* **Công việc Tuần 2 đang làm dở (Chốt nộp ngày 01/11):**
  1. Viết tài liệu `docs/STYLE_GUIDE.md`.
  2. Xây dựng `security.PasswordHasher`: `generateSalt()` sinh 16 bytes ngẫu nhiên, `hashPassword()` băm SHA-256, `verifyPassword()`.
  3. Bổ sung `salt` vào `User.java` (Constructor Overloading `this.salt = "";`).
  4. Tạo `GenericDAO<T>`, `JsonStorage.java` (Gson pretty), `UserDAOImpl.java` lưu dữ liệu vào `data/users.json`.
  5. Viết `UserService.java` (`register`, `login`).
  6. Viết `PasswordHasherTest.java`, `UserDAOTest.java`.
* **Nâng cấp Tuần 8 & Tuần 9 (Thực hiện từ 02/11 đến 05/11):**
  1. *Tuần 8:* Tạo package `com.auction.server.exception` gồm 4 lớp: `AuctionException`, `InvalidBidException`, `AuctionClosedException`, `AuthenticationException`.
  2. *Tuần 8:* Cập nhật `UserService`: ném `AuthenticationException` khi sai mật khẩu; ném `AuctionException` khi trùng username.
  3. *Tuần 9:* Mở rộng CSDL: Viết `AuctionDAOImpl.java` đọc/ghi danh sách phiên đấu giá và `bidHistory` ra file `data/auctions.json`. Khởi động Server nạp lại toàn bộ phiên vào bộ nhớ.
* **Lệnh kiểm tra đầu ra:**
  ```bash
  mvn test -Dtest="PasswordHasherTest,UserDAOTest,UserTest"
  ```

---

### 👤 4. ĐỖ MẠNH HÙNG — Mảng 1: Client GUI Base & Scene Management (MVC)
* **Nhánh làm việc:** `feature/w7-gui-base`
* **Công việc Tuần 2 đang làm dở (Chốt nộp ngày 01/11):**
  1. Viết tài liệu `docs/javafx-guide.md`.
  2. Xây dựng `resources/css/theme.css`: Dark Theme hiện đại (nền `#1e1e2f`, `#27293d`, chữ trắng, nút bấm bo góc 8px hover cyan, style TableView và TextField).
  3. Viết `SceneManager.java` (Singleton) quản lý `switchScene(String fxmlPath)` trên Stage chính.
  4. Sửa `ClientApp.java` nạp `LoginView.fxml` làm màn hình mặc định.
  5. Dựng FXML và Controller: `LoginView.fxml` + `LoginController`, `RegisterView.fxml` + `RegisterController`, `SellerDashboard.fxml` + `SellerDashboardController`, `AddItemDialog.fxml`.
* **Nâng cấp Tuần 8 & Tuần 9 (Thực hiện từ 02/11 đến 05/11):**
  1. *Tuần 8:* Hoàn thiện MVC cho `LoginController`: Bắt `AuthenticationException` và hiển thị chữ đỏ thông báo lỗi lên `#lblError`. Bấm Đăng nhập thành công chuyển Scene sang `SellerDashboard.fxml` hoặc `BiddingRoomView.fxml`.
  2. *Tuần 8:* Kết nối `SellerDashboardController` với `AddItemDialog`: Hộp thoại thêm sản phẩm mới tương ứng với `ItemFactory`.
  3. *Tuần 9:* Nối các Form với `AuctionClient`: Gửi `Request(LOGIN)`, `Request(REGISTER)`, `Request(CREATE_ITEM)` lên Server qua Socket thật thay vì mock data. Hiển thị Alert nếu mất kết nối mạng.
* **Lệnh kiểm tra đầu ra:**
  ```bash
  cd auction-client && mvn javafx:run
  ```

---

### 👤 5. LÊ ĐỨC THIỆN SANG (Nhóm trưởng) — Mảng 3: Network Infrastructure, Protocol & CI/CD
* **Nhánh làm việc:** `feature/w7-network-protocol`
* **Công việc Tuần 2 đang làm dở (Chốt nộp ngày 01/11):**
  1. Tạo `.github/pull_request_template.md` và `.github/ISSUE_TEMPLATE/bug_report.md`.
  2. Tạo `ServerApp.java` chứa `main()` in log chào mừng và gọi `AuctionServer.start()`.
  3. Thiết kế giao thức JSON: Enum `ActionType`, Enum `ResponseStatus`, Class `Request`, Class `Response`, Class `MessageSerializer` (Gson).
  4. Hiện thực Socket Server đa luồng: `AuctionServer.java` (lắng nghe cổng 8888, CachedThreadPool), `ClientHandler.java` (implements Runnable, đọc JSON, gửi trả PONG).
  5. Viết kịch bản kiểm tra `MessageSerializerTest` và `EchoClientTest`.
* **Nâng cấp Tuần 8 & Tuần 9 (Thực hiện từ 02/11 đến 05/11):**
  1. *Tuần 8:* Viết file `com.auction.server.exception.ExceptionTest.java`: Tối thiểu 8 ca kiểm thử bắt buộc sử dụng `assertThrows` cho `InvalidBidException`, `AuctionClosedException`, `AuthenticationException`.
  2. *Tuần 8:* Rà soát mã nguồn toàn dự án theo chuẩn SOLID dựa trên `docs/STYLE_GUIDE.md` của Minh.
  3. *Tuần 9:* Nâng cấp `AuctionServer.java`: Bổ sung cơ chế Broadcast `broadcast(Response res)` đẩy gói tin `NEW_BID_NOTIFICATION` tới tất cả các client đang online.
  4. *Tuần 9:* Cấu hình `maven-checkstyle-plugin` (Google Java Style) vào `pom.xml` cha và cập nhật `.github/workflows/maven-ci.yml`.
* **Lệnh kiểm tra đầu ra:**
  ```bash
  mvn checkstyle:check
  mvn test -Dtest="MessageSerializerTest,ExceptionTest"
  ```

---

# 📅 PHẦN 4 — MA TRẬN TIẾN ĐỘ 7 NGÀY & HỘI TỤ TOÀN HỆ THỐNG

| Ngày | Quân (Concurrency) | Tuấn (Realtime UI) | Minh (Persistence) | Hùng (GUI Base) | Sang (Network & CI) | Checkpoint 21:00 hàng ngày |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CN (01/11)**<br>*(Chốt Tuần 2)* | Hoàn tất Fair Lock & Observer trong `Auction` | Hoàn tất FXML & Controller BiddingRoom | Hoàn tất `PasswordHasher` & `UserDAOImpl` | Hoàn tất CSS theme & các form FXML | Hoàn tất Server Socket cổng 8888 & JSON | **HỌP MERGE 5 PR TUẦN 2 VÀO `dev` THÀNH CÔNG.** |
| **T2 (02/11)**<br>*(Tuần 8 - P1)* | Refactor `placeBid()` ném Exception | Bắt ngoại lệ trên UI BiddingRoom hiện Alert | Tạo package `exception` với 4 Custom Exceptions | Cập nhật `LoginController` bắt lỗi hiện label đỏ | Viết khung `ExceptionTest.java` JUnit 5 | Tầng ngoại lệ và bắt lỗi trên giao diện hoàn thành. |
| **T3 (03/11)**<br>*(Tuần 8 - P2)* | Củng cố `AuctionScheduler` đếm ngược đóng phiên | Gắn Timeline đếm ngược đổi màu đỏ cảnh báo | Cập nhật `UserService` ném `AuthenticationException` | Nối `SellerDashboard` với `AddItemDialog` | Hoàn thiện 8 ca test `assertThrows` trong `ExceptionTest` | Toàn bộ Unit Test ngoại lệ Tuần 8 xanh 100%. |
| **T4 (04/11)**<br>*(Tuần 9 - P1)* | Viết `AuctionConcurrencyTest` 50 threads | Viết `AuctionClient` socket nhận push realtime | Viết `AuctionDAOImpl` lưu `data/auctions.json` | Nối các form GUI với `AuctionClient` qua Socket | Viết hàm `broadcast()` giá mới; Cấu hình Checkstyle | Đặt giá realtime qua Socket hoạt động; Checkstyle sạch lỗi. |
| **T5 (05/11)**<br>*(Tuần 9 - P2)* | Review PR Mảng 2, hỗ trợ fix bug Concurrency | Review PR Mảng 1, ghép nối phòng đấu giá | Review PR Mảng 5, test bền vững file JSON | Review PR Mảng 3, hoàn thiện luồng Scene | **HỌP MERGE CODE TUẦN 8 & 9 VÀO `dev`, TEST E2E** | **MÃ NGUỒN HỢP NHẤT TOÀN DIỆN TRÊN NHÁNH `dev` THÀNH CÔNG.** |
| **T6 (06/11)**<br>*(Diễn tập)* | Luyện giải thích Fair Lock, Lost update | Luyện giải thích JavaFX Single-Threaded UI | Luyện giải thích SHA-256 + Salt, DAO | Luyện giải thích mô hình MVC trong JavaFX | **TỔNG DUYỆT DEMO & PHỎNG VẤN THỬ NỘI BỘ (1.5 TIẾNG)** | Cả 5 người tự tin trả lời vấn đáp, kịch bản demo chạy trơn tru. |
| **T7 (07/11)**<br>*(Nghiệm thu)* | Phụ trách demo Concurrency & Timer | Phụ trách demo Realtime nhảy số qua Socket | Phụ trách demo Persistence tắt/bật server | Phụ trách demo luồng giao diện Login/Dashboard | Thuyết trình kiến trúc tổng quan, Class Diagram | **BÁO CÁO TIẾN ĐỘ TRƯỚC GIẢNG VIÊN — ĐẠT ĐIỂM TỐI ĐA!** |

---

# 🔄 PHẦN 5 — MA TRẬN REVIEW CHÉO & QUY TRÌNH MERGE CODE

| PR Cần Merge | Nhánh làm việc | Reviewer 1 (Chính) | Reviewer 2 (Phụ) | Trọng tâm cần soi kỹ khi Review |
| :--- | :--- | :---: | :---: | :--- |
| **PR 1: Minh** | `feature/w7-persistence-auth` | Sang | Hùng | Mật khẩu phải được băm kèm Salt; file `users.json` và `auctions.json` lưu đúng định dạng. |
| **PR 2: Quân** | `feature/w7-concurrency-engine` | Sang | Tuấn | Hàm `placeBid()` bọc trọn trong `lock()` và `finally { unlock(); }`; ném đúng ngoại lệ. |
| **PR 3: Sang** | `feature/w7-network-protocol` | Minh | Quân | ServerSocket cổng 8888 chạy đa luồng không bị treo; Checkstyle 0 lỗi convention. |
| **PR 4: Hùng** | `feature/w7-gui-base` | Tuấn | Sang | Cửa sổ mở ra hiển thị Dark Theme sắc nét, không bị lỗi FXML loader. |
| **PR 5: Tuấn** | `feature/w7-realtime-bidding-room` | Quân | Hùng | Đặt giá trên Client thì giao diện tự động nhảy số qua `Platform.runLater()`. |

---

# 🎯 PHẦN 6 — KỊCH BẢN DEMO 5 BƯỚC "ĂN TRỌN ĐIỂM" TRƯỚC GIẢNG VIÊN (07/11)

1. **Bước 1 — Báo cáo Cấu trúc Dự án & CI/CD (1 phút):**
   - Mở file `pom.xml`: Cho thầy xem cấu trúc Maven Multi-Module (`auction-server`, `auction-client`) và plugin `maven-checkstyle-plugin`.
   - Mở GitHub: Cho thầy xem các nhánh feature, danh sách Pull Request có review chéo nghiêm túc, và luồng **GitHub Actions CI đang báo xanh (BUILD SUCCESS)**.
2. **Bước 2 — Khởi động Server & Đăng nhập 2 Client (2 phút):**
   - Bật `ServerApp`: Console log hiển thị Server đang lắng nghe trên cổng 8888.
   - Mở đồng thời 2 cửa sổ `ClientApp`:
     - Cửa sổ 1: Đăng nhập tài khoản **Seller** $\rightarrow$ Tạo sản phẩm mới (chọn loại Laptop, giá sàn 10.000.000 VNĐ, bước giá 500.000 VNĐ).
     - Cửa sổ 2: Đăng nhập tài khoản **Bidder A** $\rightarrow$ Thấy phiên đấu giá mới xuất hiện tức thời $\rightarrow$ Bấm tham gia phòng đấu giá.
3. **Bước 3 — Demo Realtime Bidding & Concurrency (2 phút):**
   - Mở thêm Cửa sổ 3: Đăng nhập tài khoản **Bidder B** cùng vào phòng đấu giá đó.
   - Bidder A bấm nút `+1 Bước giá` rồi bấm `ĐẶT GIÁ NGAY` $\rightarrow$ Ngay lập tức cả 3 màn hình (Seller, Bidder A, Bidder B) đều **tự động nhảy số tiền 10.500.000 VNĐ và thêm 1 dòng vào bảng lịch sử** mà không cần ai phải bấm F5.
   - Giải thích cho thầy: Sự kiện từ UI $\rightarrow$ Socket Server $\rightarrow$ Khóa `ReentrantLock(true)` $\rightarrow$ Kích hoạt `AuctionObserver` $\rightarrow$ Broadcast qua Socket $\rightarrow$ Client dùng `Platform.runLater()` cập nhật JavaFX.
4. **Bước 4 — Demo Bắt Ngoại Lệ & Tính Bền Vững Dữ Liệu (2 phút):**
   - Cố tình nhập giá 10.600.000 VNĐ (nhỏ hơn bước giá 500.000 VNĐ) $\rightarrow$ Giao diện bắt đúng `InvalidBidException` và mở Alert cảnh báo đỏ: *"Mức giá đặt phải lớn hơn giá hiện tại ít nhất 1 bước giá!"*.
   - Tắt hoàn toàn Server, sau đó bật lại Server $\rightarrow$ Mở danh sách phiên: Dữ liệu phiên đấu giá và lịch sử bid vẫn còn nguyên vẹn nhờ tầng lưu trữ `JsonStorage` đọc từ file JSON.
5. **Bước 5 — Trình diễn Toàn Bộ Bộ Kiểm Thử Unit Test (1 phút):**
   - Chạy lệnh `mvn test` trên Terminal trước mặt thầy: Toàn bộ Unit Tests và test Concurrency 50 luồng đồng thời đều vượt qua 100%.

---

# ✅ PHẦN 7 — CHECKLIST KIỂM ĐỊNH TỔNG HỢP TRƯỚC GIỜ G

### Checklist chung cho cả 5 thành viên:
- [ ] Nắm chắc kiến trúc tổng quan hệ thống và trả lời trôi chảy 5 câu hỏi vấn đáp lý thuyết trong Phần 1.
- [ ] Đã hoàn thành review và bấm Approve ít nhất 2 Pull Request của đồng đội theo Ma trận Review.
- [ ] Mã nguồn trên nhánh cá nhân đã được merge sạch sẽ vào nhánh `dev`, không còn nhánh rác.
- [ ] Toàn bộ dự án chạy lệnh `mvn clean test` báo **BUILD SUCCESS** (100% test pass).
- [ ] Chạy lệnh `mvn checkstyle:check` không có bất kỳ cảnh báo lỗi format nào.

### Checklist riêng cho từng người:
- [ ] **Quân:** Hàm `placeBid()` bọc trọn trong `ReentrantLock(true)`; `AuctionScheduler` tự động đóng phiên; bộ test 50 threads chạy pass.
- [ ] **Tuấn:** `AuctionObserver` chuẩn tên hàm; `BiddingRoomView.fxml` hiển thị đẹp mắt; nhận push từ Socket nhảy số realtime qua `Platform.runLater()`.
- [ ] **Minh:** 4 Custom Exceptions ném đúng chỗ; mật khẩu được băm SHA-256 + Salt; file `data/users.json` và `data/auctions.json` lưu trữ bền vững.
- [ ] **Hùng:** Bộ Dark theme `theme.css` sắc nét; `LoginView`, `RegisterView`, `SellerDashboard` chuyển cảnh mượt qua `SceneManager`.
- [ ] **Sang:** `AuctionServer` cổng 8888 đa luồng ổn định; giao thức JSON Gson chuẩn hóa; GitHub Actions CI xanh 100%.
