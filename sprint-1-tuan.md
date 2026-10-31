# 📋 KẾ HOẠCH TỔNG HỢP 1 TUẦN NƯỚC RÚT (01/11 – 07/11/2026)
## Dự Án: Hệ Thống Đấu Giá Trực Tuyến (Online Auction System) — Nhóm 1
### Mục tiêu: Hoàn Thiện Toàn Diện Hạng Mục Tuần 7 – 8 – 9 Trước Giờ Chấm Tiến Độ 07/11

> **Môn học:** Lập trình nâng cao (2627I_UET.CS2043_6)  
> **Thời gian tác chiến:** Đúng 7 ngày (Từ Chủ Nhật 01/11/2026 đến Thứ Bảy 07/11/2026)  
> **Deadline nộp PR lên nhánh cá nhân:** Thứ Năm ngày 05/11/2026, 21:00  
> **Họp Merge Code & Test Tích Hợp Toàn Bộ:** Thứ Năm ngày 05/11/2026, 21:30 – 23:30  
> **Tổng Duyệt Demo & Vấn Đáp Thử:** Thứ Sáu ngày 06/11/2026, 19:30 – 21:30  
> **Thời điểm Giảng viên chấm tiến độ:** Thứ Bảy ngày 07/11/2026  
> **Mô hình tác chiến:** **5 Trục Chuyên Môn Độc Lập (Horizontal Tracks)** — Kế thừa 100% sự phân công từ Tuần 2 (`plan/week2.md`), mỗi người code và test độc lập trên nhánh cá nhân bằng Mock Data, ghép nối qua Socket và merge vào nhánh `dev` vào Thứ Năm (05/11).

---

# 📌 PHẦN 1 — PHẦN CHUNG: TỰ HỌC NỀN TẢNG & VẤN ĐÁP LÝ THUYẾT

Mỗi thành viên phụ trách trục nào phải nghiên cứu sâu mảng đó và giải thích được cho cả nhóm khi họp vấn đáp.

### 1.1. Bảng kỹ năng tự học trọng tâm

| # | Nội dung tự học | Thành viên cần nhất | Đầu ra kiểm tra (Demo khi họp nhóm) |
| :-: | :--- | :---: | :--- |
| **1** | **Gson:** Chuyển đổi `Java Object ↔ JSON` | **Minh, Sang** | Serialize và deserialize được `Request`, `Response`, `User`, `Auction`. |
| **2** | **Java Socket đa luồng:** `ServerSocket` + `Socket` + `ExecutorService` | **Sang, Tuấn** | Server lắng nghe cổng 8888, chấp nhận nhiều Client kết nối song song. |
| **3** | **JavaFX FXML + SceneBuilder + CSS Dark Theme** | **Hùng, Tuấn** | Dựng form Login, Register, Dashboard và BiddingRoom với Dark Theme. |
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
   * *Trả lời:* Vì thao tác đọc dữ liệu từ Socket (`in.readLine()`) là thao tác chặn luồng (*blocking I/O*). Nếu xử lý trên một luồng duy nhất, khi 1 Client đang kết nối hoặc bị lag mạng thì toàn bộ các Client khác sẽ bị treo cứng. Việc sử dụng Thread Pool (`ExecutorService`) cho phép Server tiếp nhận và phục vụ đồng thời hàng chục Client độc lập mà không làm nghẽn luồng chấp nhận kết nối chính `accept()`.
3. **Câu hỏi về Giao diện JavaFX (Hùng & Tuấn):**
   * *Hỏi:* Tại sao khi cập nhật giao diện JavaFX từ luồng Socket nhận tin nhắn bắt buộc phải dùng `Platform.runLater()`? Điều gì xảy ra nếu vi phạm?
   * *Trả lời:* JavaFX tuân thủ nguyên tắc *Single-Threaded UI Rule*. Toàn bộ thao tác vẽ hoặc thay đổi component (Label, TableView, Dialog) bắt buộc phải diễn ra trên luồng chính *JavaFX Application Thread*. Dữ liệu Socket chạy trên luồng ngầm (*background thread*). Nếu gọi trực tiếp thay đổi UI từ luồng ngầm sẽ gây ra lỗi nghiêm trọng `IllegalStateException: Not on FX application thread`. Do đó, phải bọc lệnh cập nhật trong `Platform.runLater()` để đưa tác vụ vào hàng đợi của luồng UI.
4. **Câu hỏi về Bảo mật & Lưu trữ Dữ liệu (Minh):**
   * *Hỏi:* Tại sao không được lưu mật khẩu dạng thô (plain-text) và tại sao khi băm SHA-256 lại bắt buộc phải thêm chuỗi Salt?
   * *Trả lời:* Lưu mật khẩu dạng thô sẽ khiến tài khoản người dùng bị lộ hoàn toàn nếu file CSDL bị đánh cắp. Băm SHA-256 là hàm một chiều giúp giấu mật khẩu gốc. Tuy nhiên, nếu chỉ băm SHA-256 thuần, kẻ tấn công có thể dùng bảng tra cứu dựng sẵn (*Rainbow Table*) để giải mã ngược các mật khẩu phổ biến. Việc thêm chuỗi Salt (16 bytes ngẫu nhiên từ `SecureRandom`) đảm bảo rằng hai người dùng có cùng mật khẩu thì chuỗi băm lưu trữ vẫn hoàn toàn khác nhau, vô hiệu hóa hoàn toàn Rainbow Table.
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
│   └── ClassDiagram.png                         # Sơ đồ lớp UML hệ thống
│
├── auction-server/                              # [MODULE BACKEND: NGHIỆP VỤ, MẠNG & CSDL]
│   ├── pom.xml
│   ├── data/                                    # [Minh] Thư mục lưu trữ CSDL JSON bền vững
│   │   ├── users.json                           # Danh sách tài khoản đã băm mật khẩu
│   │   └── auctions.json                        # Danh sách phiên và lịch sử đặt giá
│   └── src/
│       ├── main/java/com/auction/server/
│       │   ├── ServerApp.java                   # [Sang] Lớp mồi khởi động Server cổng 8888
│       │   ├── domain/model/                    # Tầng thực thể nghiệp vụ (Đã có từ Tuần 1)
│       │   │   ├── Entity.java
│       │   │   ├── user/ (User, Role, Bidder, Seller, Admin)
│       │   │   ├── item/ (Item, ItemType, Electronics, Art, Vehicle, ItemFactory)
│       │   │   └── auction/
│       │   │       ├── Auction.java             # [Quân DUY NHẤT SỬA] ReentrantLock & Observer
│       │   │       ├── AuctionStatus.java       # [Minh] Trạng thái vòng đời phiên
│       │   │       ├── AuctionManager.java      # [Sang] Singleton quản lý phiên & hủy phiên
│       │   │       ├── Bid.java                 # Đối tượng đặt giá bất biến
│       │   │       ├── AuctionScheduler.java    # [Quân] Timer tự động đóng phiên khi hết giờ
│       │   │       └── observer/
│       │   │           └── AuctionObserver.java # [Tuấn TẠO] Interface Observer chuẩn
│       │   ├── exception/                       # [Minh] Package Custom Exceptions
│       │   │   ├── AuctionException.java        # Exception cha chung
│       │   │   ├── InvalidBidException.java     # Lỗi giá đặt không hợp lệ / nhỏ hơn bước giá
│       │   │   ├── AuctionClosedException.java  # Lỗi đặt giá khi phiên đã kết thúc / bị hủy
│       │   │   └── AuthenticationException.java # Lỗi đăng nhập thất bại
│       │   ├── security/                        # [Minh]
│       │   │   └── PasswordHasher.java          # Tiện ích băm SHA-256 + 16 bytes Salt
│       │   ├── persistence/                     # [Minh] Tầng lưu trữ DAO
│       │   │   ├── GenericDAO.java              # Interface CRUD tổng quát
│       │   │   ├── UserDAOImpl.java             # Đọc/ghi users.json
│       │   │   ├── AuctionDAOImpl.java          # Đọc/ghi auctions.json
│       │   │   └── JsonStorage.java             # Tiện ích đọc/ghi file Gson Pretty
│       │   ├── service/                         # [Minh]
│       │   │   └── UserService.java             # Dịch vụ đăng ký / đăng nhập người dùng
│       │   └── network/                         # [Sang]
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
│           ├── domain/model/                    # Tests từ Tuần 1 (UserTest, ItemFactoryTest, BidTest...)
│           ├── concurrency/
│           │   └── AuctionConcurrencyTest.java  # [Quân] Test áp lực 50 threads đặt giá đồng thời
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
        │   ├── network/                         # [Tuấn]
        │   │   └── AuctionClient.java           # Luồng Socket Client ngầm lắng nghe Server push
        │   ├── util/                            # [Hùng]
        │   │   └── SceneManager.java            # Singleton điều hướng chuyển màn hình
        │   └── controller/                      # [Hùng & Tuấn] Các Controller FXML (MVC)
        │       ├── LoginController.java         # [Hùng] Điều khiển Đăng nhập
        │       ├── RegisterController.java      # [Hùng] Điều khiển Đăng ký
        │       ├── SellerDashboardController.java # [Hùng] Điều khiển Quản lý hàng Seller
        │       └── BiddingRoomController.java   # [Tuấn] Điều khiển Phòng đấu giá trực tiếp
        └── resources/
            ├── css/
            │   └── theme.css                    # [Hùng] Dark Theme CSS hiện đại
            └── fxml/
                ├── LoginView.fxml               # [Hùng] Giao diện đăng nhập
                ├── RegisterView.fxml            # [Hùng] Giao diện đăng ký
                ├── SellerDashboard.fxml         # [Hùng] Giao diện bảng điều khiển Seller
                ├── AddItemDialog.fxml           # [Hùng] Hộp thoại tạo sản phẩm
                └── BiddingRoomView.fxml         # [Tuấn] Giao diện phòng đấu giá trực tiếp
```

---

# 🔨 PHẦN 3 — ĐẶC TẢ NHIỆM VỤ CÁ NHÂN CHI TIẾT (5 THÀNH VIÊN)

> [!IMPORTANT]
> Toàn bộ tên class, tên hàm, kiểu dữ liệu và tham số dưới đây được chuẩn hóa 100%. Mỗi thành viên làm việc độc lập trên **nhánh riêng**, tuân thủ Conventional Commits, viết đầy đủ Unit Test và tự test độc lập trên máy trước khi merge.

---

### 👤 1. TRẦN MẠNH QUÂN — Mảng 5: Concurrency Engine & Auction Lifecycle
* **Nhánh làm việc:** `feature/w7-concurrency-engine`
* **Vai trò:** Chịu trách nhiệm bảo đảm tính toàn vẹn của dữ liệu đấu giá khi có tranh chấp đa luồng và tự động hóa kết thúc phiên.

#### Nhiệm vụ cụ thể:
1. **Nâng cấp An toàn Đa luồng cho `Auction.java`:**
   - Thay thế từ khóa `synchronized` bằng khóa công bằng:
     ```java
     private final ReentrantLock lock = new ReentrantLock(true);
     ```
   - Khai báo danh sách Observer luồng an toàn:
     ```java
     private final List<AuctionObserver> observers = new CopyOnWriteArrayList<>();
     ```
   - Hiện thực các phương thức quản lý Observer:
     - `public void attachObserver(AuctionObserver observer)`: Thêm observer vào danh sách nếu khác `null` và chưa có.
     - `public void detachObserver(AuctionObserver observer)`: Xóa observer khỏi danh sách.
     - `private void notifyNewBid(Bid bid)`: Duyệt `observers` và gọi `o.onNewBidPlaced(this, bid)`.
     - `public void notifyStatusChanged(AuctionStatus newStatus)`: Duyệt `observers` và gọi `o.onAuctionStatusChanged(this, newStatus)`.
2. **Refactor phương thức `placeBid()` ném Custom Exceptions:**
   - Thay đổi kiểu trả về và ném ngoại lệ tường minh:
     ```java
     public void placeBid(Bidder bidder, double amount) throws InvalidBidException, AuctionClosedException {
         if (bidder == null) throw new IllegalArgumentException("Người đặt giá không được null");
         lock.lock();
         try {
             if (status != AuctionStatus.RUNNING && status != AuctionStatus.OPEN) {
                 throw new AuctionClosedException("Phiên đấu giá đang ở trạng thái " + status + ", không thể đặt giá!");
             }
             if (bidder.getId().equals(seller.getId())) {
                 throw new InvalidBidException("Người bán không được phép tự đấu giá sản phẩm của chính mình!");
             }
             if (bidder.getBalance() < amount) {
                 throw new InvalidBidException("Số dư tài khoản (" + bidder.getBalance() + ") không đủ để đặt mức giá: " + amount);
             }
             double minRequired = bidHistory.isEmpty() ? item.getStartingPrice() : currentHighestBid + bidIncrement;
             if (amount < minRequired) {
                 throw new InvalidBidException("Mức giá đặt (" + amount + ") phải lớn hơn hoặc bằng giá tối thiểu: " + minRequired);
             }
             Bid newBid = new Bid(bidder, getId(), amount);
             bidHistory.add(newBid);
             currentHighestBid = amount;
             currentWinner = bidder;
             notifyNewBid(newBid);
         } finally {
             lock.unlock();
         }
     }
     ```
3. **Hiện thực Bộ đếm thời gian tự động `AuctionScheduler.java`:**
   - Tạo tại package `com.auction.server.domain.model.auction`:
   - Sử dụng `private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);`
   - Phương thức `public void scheduleAuctionEnd(Auction auction)`:
     - Tính số mili-giây còn lại: `Duration.between(LocalDateTime.now(), auction.getEndTime()).toMillis()`.
     - Nếu đã hết giờ $\rightarrow$ gọi đóng phiên ngay. Nếu còn thời gian $\rightarrow$ lập lịch tác vụ:
       ```java
       scheduler.schedule(() -> {
           lock.lock(); // đảm bảo an toàn luồng khi đổi trạng thái
           try {
               auction.updateStatus(AuctionStatus.FINISHED);
               Bid winnerBid = auction.determineWinner();
               System.out.println("Phiên " + auction.getId() + " đã kết thúc. Người thắng: " + (winnerBid != null ? winnerBid.getBidder().getUsername() : "Không có"));
           } finally {
               lock.unlock();
           }
       }, delayMillis, TimeUnit.MILLISECONDS);
       ```
   - Phương thức `public void shutdown()`: gọi `scheduler.shutdownNow()`.
4. **Viết Unit Test Concurrency chuyên sâu `AuctionConcurrencyTest.java`:**
   - Tạo tại package `com.auction.server.concurrency`:
   - Khởi tạo 1 phiên đấu giá với giá khởi điểm `100.0`, bước giá `10.0`.
   - Tạo **50 đối tượng `Bidder`** độc lập, mỗi người có số dư `50,000.0`.
   - Sử dụng `CountDownLatch startLatch = new CountDownLatch(1);` và `CountDownLatch doneLatch = new CountDownLatch(50);`.
   - Cho 50 luồng đồng thời gọi `auction.placeBid(bidder, price)` tại cùng một thời điểm xuất phát.
   - **Assertions kiểm tra:**
     - Không phát sinh ngoại lệ không mong muốn.
     - Lịch sử đặt giá `bidHistory` có giá tăng dần đơn điệu (`bidHistory.get(i).getAmount() >= bidHistory.get(i-1).getAmount() + 10.0`).
     - Mức giá cao nhất `currentHighestBid` bằng chính xác mức giá của phần tử cuối cùng trong `bidHistory`.

#### ✅ Lệnh kiểm tra đầu ra:
```bash
mvn test -Dtest="AuctionConcurrencyTest,AuctionTest"
# PASS nếu: 50 luồng chạy đồng thời an toàn tuyệt đối, 100% test cases pass.
```

---

### 👤 2. LÊ ANH TUẤN — Mảng 2: Realtime UI & Socket Client Receiver
* **Nhánh làm việc:** `feature/w7-realtime-bidding-room`
* **Vai trò:** Xây dựng màn hình Phòng đấu giá trực tiếp trung tâm và luồng nhận dữ liệu Socket thời gian thực.

#### Nhiệm vụ cụ thể:
1. **Định nghĩa Interface `AuctionObserver.java`:**
   - Tạo tại: `com.auction.server.domain.model.auction.observer.AuctionObserver.java`
     ```java
     public interface AuctionObserver {
         void onNewBidPlaced(Auction auction, Bid newBid);
         void onAuctionStatusChanged(Auction auction, AuctionStatus newStatus);
     }
     ```
2. **Thiết kế layout FXML `BiddingRoomView.fxml`:**
   - Tạo tại `resources/fxml/BiddingRoomView.fxml` bằng SceneBuilder:
     - Khung thông tin: Tên sản phẩm `#lblItemTitle`, Giá khởi điểm `#lblStartingPrice`, Người bán `#lblSellerName`, Mô tả `#txtDescription`.
     - Nhãn giá cao nhất hiện tại `#lblCurrentPrice`: Font chữ 34px đậm, màu cyan sáng `#00f2fe`.
     - Nhãn người đang dẫn đầu `#lblCurrentWinner`: Chữ màu trắng sáng.
     - Đồng hồ đếm ngược `#lblCountdown`: Hiển thị định dạng `00:05:30`.
     - Bảng `TableView<Bid>` lịch sử đặt giá `#tblBidHistory` với 3 cột: Thời gian (`#colTime`), Người đặt giá (`#colBidder`), Mức giá (`#colAmount`).
     - Cụm đặt giá: Ô nhập `#txtBidAmount`, các nút bấm nhanh: `+1 Bước giá` (`#btnQuick1`), `+2 Bước giá` (`#btnQuick2`), nút nổi bật `ĐẶT GIÁ NGAY` (`#btnPlaceBid`).
3. **Hiện thực `BiddingRoomController.java` (MVC Controller):**
   - Implements `AuctionObserver`.
   - Xây dựng đồng hồ đếm ngược: Dùng `javafx.animation.Timeline` lặp lại mỗi 1 giây để trừ dần thời gian. Khi còn dưới 60 giây, tự động đổi màu nhãn sang đỏ rực `#ff4d4d`.
   - `@Override public void onNewBidPlaced(Auction auction, Bid newBid)`:
     ```java
     Platform.runLater(() -> {
         lblCurrentPrice.setText(String.format("%,.0f VNĐ", newBid.getAmount()));
         lblCurrentWinner.setText("Người dẫn đầu: " + newBid.getBidder().getUsername());
         tblBidHistory.getItems().add(0, newBid); // Chèn lên đầu bảng
     });
     ```
   - Bắt ngoại lệ khi bấm nút `ĐẶT GIÁ`:
     ```java
     try {
         double amount = Double.parseDouble(txtBidAmount.getText().trim());
         // Gửi yêu cầu đặt giá (gọi AuctionClient hoặc gọi Model)
     } catch (NumberFormatException e) {
         showAlert("Lỗi nhập liệu", "Vui lòng nhập một số tiền hợp lệ!");
     } catch (InvalidBidException | AuctionClosedException e) {
         showAlert("Từ chối đặt giá", e.getMessage());
     }
     ```
   - Hàm tiện ích: `private void showAlert(String title, String content)` mở popup `Alert(AlertType.WARNING)`.
4. **Hiện thực lớp mạng Client `AuctionClient.java`:**
   - Tạo tại package `com.auction.client.network`:
   - Quản lý `Socket socket; BufferedReader in; PrintWriter out;`.
   - Hàm `public void connect(String host, int port)` kết nối tới `localhost:8888`.
   - Khởi chạy một `Thread` chạy ngầm liên tục đọc gói tin JSON từ Server. Khi nhận gói tin `NEW_BID_NOTIFICATION` $\rightarrow$ phân tích dữ liệu và kích hoạt callback tới `BiddingRoomController`.
   - Hàm `public synchronized void sendRequest(Request request)`: chuyển `Request` thành JSON và gửi qua Socket.
5. **Viết Unit Test cho Observer:**
   - Tạo `AuctionObserverTest.java`: Tạo mock observer, gắn vào Auction, gọi đặt giá và assert kiểm tra hàm `onNewBidPlaced` được gọi chính xác.

#### ✅ Lệnh kiểm tra đầu ra:
```bash
mvn test -Dtest="AuctionObserverTest"
# PASS: 100% test observer pass; mở BiddingRoomView chạy thử nhảy số realtime mượt mà.
```

---

### 👤 3. HOÀNG THẾ CÔNG MINH — Mảng 4: Persistence, Security & Custom Exceptions
* **Nhánh làm việc:** `feature/w7-persistence-auth`
* **Vai trò:** Xây dựng hệ thống ngoại lệ, mã hóa mật khẩu và lưu trữ CSDL ra file JSON bền vững.

#### Nhiệm vụ cụ thể:
1. **Xây dựng Package Custom Exceptions (`com.auction.server.exception`):**
   - `AuctionException.java` (`public class AuctionException extends RuntimeException`): Lớp cha gốc.
   - `InvalidBidException.java` (`extends AuctionException`): Lỗi bước giá hoặc số dư.
   - `AuctionClosedException.java` (`extends AuctionException`): Lỗi đặt giá vào phiên đã đóng.
   - `AuthenticationException.java` (`extends AuctionException`): Lỗi sai thông tin đăng nhập.
2. **Hiện thực hóa `PasswordHasher.java` (`security`):**
   - `public static String generateSalt()`: Sinh mảng ngẫu nhiên 16 bytes bằng `SecureRandom`, chuyển thành chuỗi Hex 32 ký tự.
   - `public static String hashPassword(String password, String salt)`: Kết hợp `password + salt`, băm qua `MessageDigest.getInstance("SHA-256")`, trả về chuỗi Hex 64 ký tự.
   - `public static boolean verifyPassword(String inputPassword, String storedHash, String storedSalt)`: Băm lại mật khẩu nhập vào với salt đã lưu và so sánh chuỗi hash.
3. **Cập nhật thuộc tính `salt` cho `User.java`:**
   - Bổ sung trường `private String salt;` kèm getter/setter.
   - Áp dụng **Constructor Overloading** để gán mặc định `this.salt = ""` nhằm giữ nguyên vẹn 100% tương thích với các class `Bidder`, `Seller`, `Admin` và bộ `UserTest` từ Tuần 1.
4. **Xây dựng Tầng Lưu Trữ File JSON (`persistence`):**
   - `GenericDAO.java`:
     ```java
     public interface GenericDAO<T extends Entity> {
         void save(T entity);
         T findById(String id);
         List<T> findAll();
         void update(T entity);
         void delete(String id);
     }
     ```
   - `JsonStorage.java`: Sử dụng Gson với `.setPrettyPrinting()` và đăng ký TypeAdapter cho `LocalDateTime`. Cung cấp 2 hàm tổng quát:
     - `public static <T> List<T> readList(String filePath, Class<T> clazz)`
     - `public static <T> void writeList(String filePath, List<T> list)`
   - `UserDAOImpl.java`: Lưu danh sách người dùng vào `auction-server/data/users.json`. Mật khẩu lưu bắt buộc là chuỗi băm kèm salt.
   - `AuctionDAOImpl.java`: Lưu danh sách phiên và `bidHistory` vào `auction-server/data/auctions.json`. Khi khởi động Server, nạp lại toàn bộ phiên vào `AuctionManager`.
5. **Hiện thực `UserService.java`:**
   - `public User register(String username, String password, String email, Role role)`: Validate dữ liệu, kiểm tra trùng lặp qua `UserDAO`, sinh salt, băm mật khẩu, lưu vào file JSON.
   - `public User login(String username, String password)`: Tìm qua `UserDAO`, verify mật khẩu băm, trả về User hợp lệ (ném `AuthenticationException` nếu sai).
6. **Viết Unit Test Đầy Đủ:**
   - `PasswordHasherTest.java`: Test hash nhất quán, cùng password khác salt cho kết quả khác nhau, verify đúng/sai.
   - `UserDAOTest.java`: Test CRUD tài khoản vào file JSON thật.

#### ✅ Lệnh kiểm tra đầu ra:
```bash
mvn test -Dtest="PasswordHasherTest,UserDAOTest,UserTest"
# PASS nếu: 100% test pass, file data/users.json được sinh ra chứa mật khẩu băm an toàn.
```

---

### 👤 4. ĐỖ MẠNH HÙNG — Mảng 1: Client GUI Base & Scene Management (MVC)
* **Nhánh làm việc:** `feature/w7-gui-base`
* **Vai trò:** Phụ trách toàn bộ mỹ thuật giao diện ứng dụng, hệ thống điều hướng Scene và các màn hình quản lý.

#### Nhiệm vụ cụ thể:
1. **Xây dựng Bộ CSS Theme Dùng Chung (`resources/css/theme.css`):**
   - Thiết kế giao diện **Dark Theme hiện đại**:
     - Nền tối: `#1e1e2f` (Background chính), `#27293d` (Card / Panel).
     - Màu chữ: `#ffffff` (Chính), `#a0a0b0` (Phụ).
     - Accent: Gradient `#4facfe` $\rightarrow$ `#00f2fe`.
   - Định dạng chi tiết cho các thẻ:
     - `.button`: Nền gradient, chữ trắng, bo góc `8px`, font bold, hiệu ứng `:hover` sáng bóng, `:pressed` co nhẹ.
     - `.text-field`, `.password-field`: Nền tối `#181824`, viền mờ `1px solid #3a3a4c`, khi `:focused` viền sáng cyan.
     - `.table-view`: Nền tối, xen kẽ màu dòng chẵn/lẻ, header phẳng hiện đại.
     - `.label-error`: Chữ đỏ `#ff4d4d`, font size 12px dùng hiển thị lỗi form.
2. **Hiện thực `SceneManager.java` (Singleton):**
   - Tạo tại `com.auction.client.util.SceneManager.java`:
   - `public static synchronized SceneManager getInstance()`
   - `public void setPrimaryStage(Stage stage)`: Gán cửa sổ chính khi ứng dụng start.
   - `public void switchScene(String fxmlPath, String title)`:
     ```java
     FXMLLoader loader = new FXMLLoader(getClass().getResource(fxmlPath));
     Parent root = loader.load();
     Scene scene = new Scene(root);
     scene.getStylesheets().add(getClass().getResource("/css/theme.css").toExternalForm());
     primaryStage.setScene(scene);
     primaryStage.setTitle(title);
     primaryStage.show();
     ```
3. **Thiết kế các màn hình FXML và Controller chuẩn MVC:**
   - `LoginView.fxml` + `LoginController.java`:
     - Ô nhập: `#txtUsername`, `#txtPassword` (PasswordField).
     - Nút bấm: `#btnLogin`, `#btnGoToRegister`.
     - Nhãn lỗi: `#lblError` (Mặc định ẩn, khi bắt `AuthenticationException` thì hiện chữ đỏ).
     - Đăng nhập thành công $\rightarrow$ chuyển sang `SellerDashboard.fxml` hoặc `BiddingRoomView.fxml`.
   - `RegisterView.fxml` + `RegisterController.java`:
     - Ô nhập: Username, Password, Confirm Password, Email.
     - ComboBox chọn vai trò: `Role.BIDDER` hoặc `Role.SELLER`.
     - Kiểm tra validate: Mật khẩu tối thiểu 6 ký tự, email có '@', password khớp confirm.
   - `SellerDashboard.fxml` + `SellerDashboardController.java`:
     - Bảng `TableView` hiển thị danh sách sản phẩm đăng bán của Seller.
     - Nút `btnOpenAddDialog`: Mở hộp thoại popup `AddItemDialog.fxml`.
   - `AddItemDialog.fxml`: Hộp thoại nhập Tên sản phẩm, Giá khởi điểm, Bước giá, chọn loại (`ELECTRONICS`, `ART`, `VEHICLE`) và các trường đặc thù tương ứng với `ItemFactory`.
4. **Kết nối Controller với Socket Client:**
   - Khi bấm Đăng nhập/Đăng ký $\rightarrow$ gọi `AuctionClient.sendRequest()` thay vì gọi dữ liệu mock.
   - Xử lý mất kết nối: Nếu không nối được Server $\rightarrow$ hiện Alert thông báo lỗi mạng thân thiện.

#### ✅ Lệnh kiểm tra đầu ra:
```bash
cd auction-client
mvn javafx:run
# PASS nếu: Mở form Login với Dark Theme đẹp mắt, bấm chuyển qua Register mượt mà, không lỗi FXML.
```

---

### 👤 5. LÊ ĐỨC THIỆN SANG — Mảng 3: Hạ Tầng Mạng Socket, Protocol & CI/CD
* **Nhánh làm việc:** `feature/w7-network-protocol`
* **Vai trò:** Thiết kế giao thức mạng JSON, xây dựng ServerSocket đa luồng, tích hợp Checkstyle và điều phối CI/CD.

#### Nhiệm vụ cụ thể:
1. **Thiết kế Giao Thức Gói Tin Chuẩn JSON (`com.auction.server.network.protocol`):**
   - Enum `ActionType`: `LOGIN`, `REGISTER`, `GET_ACTIVE_AUCTIONS`, `PLACE_BID`, `CREATE_ITEM`, `NEW_BID_NOTIFICATION`, `AUCTION_CLOSED_NOTIFICATION`, `PING`.
   - Enum `ResponseStatus`: `OK`, `ERROR`, `UNAUTHORIZED`, `NOT_FOUND`.
   - Class `Request.java`:
     - `private ActionType action; private Map<String, Object> data; private String token;`
     - Constructor đầy đủ, getters/setters, hàm tiện ích `Request.of(action, data)`.
   - Class `Response.java`:
     - `private ResponseStatus status; private String message; private Object data;`
     - Các hàm tiện ích: `Response.ok(Object data)`, `Response.error(String message)`, `Response.push(ActionType event, Object data)`.
   - Class `MessageSerializer.java`:
     - `public static String serialize(Object obj)`: Chuyển Request/Response sang JSON bằng Gson.
     - `public static Request deserializeRequest(String json)`: Chuyển JSON sang Request.
     - `public static Response deserializeResponse(String json)`: Chuyển JSON sang Response.
2. **Hiện thực ServerSocket Đa Luồng (`com.auction.server.network.server`):**
   - `AuctionServer.java`:
     - Lắng nghe cổng `8888`.
     - Quản lý luồng bằng `ExecutorService threadPool = Executors.newCachedThreadPool();`.
     - Danh sách Client kết nối: `private final List<ClientHandler> clients = new CopyOnWriteArrayList<>();`.
     - Vòng lặp lắng nghe:
       ```java
       while (running) {
           Socket socket = serverSocket.accept();
           ClientHandler handler = new ClientHandler(socket, this);
           clients.add(handler);
           threadPool.execute(handler);
       }
       ```
     - Hàm Broadcast thời gian thực:
       ```java
       public void broadcast(Response response) {
           for (ClientHandler client : clients) {
               client.sendMessage(response);
           }
       }
       ```
   - `ClientHandler.java` (implements `Runnable`):
     - Đọc dữ liệu JSON từ `BufferedReader in`, chuyển đổi sang `Request`.
     - Phân phối yêu cầu (Routing): Gọi `UserService` (cho LOGIN, REGISTER) hoặc `Auction` (cho PLACE_BID).
     - Phản hồi kết quả qua `PrintWriter out`.
     - Hàm gửi an toàn luồng: `public synchronized void sendMessage(Response res)`.
3. **Cấu hình `ServerApp.java`:**
   - Chứa hàm `main(String[] args)` khởi động `AuctionServer` trên cổng 8888, in log chào mừng và sẵn sàng tiếp nhận kết nối.
4. **Tích hợp Checkstyle & Nâng cấp GitHub Actions:**
   - Thêm `maven-checkstyle-plugin` (phiên bản `3.3.1`, cấu hình `google_checks.xml`) vào `pom.xml` cha.
   - Cập nhật `.github/workflows/maven-ci.yml`: Bổ sung bước kiểm tra checkstyle `mvn checkstyle:check` trước khi chạy unit tests.
5. **Viết Unit Test Ngoại Lệ & Giao Thức Mạng:**
   - `MessageSerializerTest.java`: Test chuyển đổi 2 chiều JSON $\leftrightarrow$ Request/Response.
   - `ExceptionTest.java`: Viết tối thiểu 8 ca kiểm thử bắt buộc sử dụng `assertThrows` cho `InvalidBidException`, `AuctionClosedException`, `AuthenticationException`.

#### ✅ Lệnh kiểm tra đầu ra:
```bash
mvn checkstyle:check
mvn test -Dtest="MessageSerializerTest,ExceptionTest"
# PASS nếu: 0 lỗi checkstyle, 100% test cases ngoại lệ và giao thức JSON đều xanh lá.
```

---

# 📅 PHẦN 4 — MA TRẬN TIẾN ĐỘ 7 NGÀY & HỘI TỤ TOÀN HỆ THỐNG

### Lịch làm việc chi tiết từng ngày (01/11 – 07/11):

| Ngày | Quân (Concurrency) | Tuấn (Realtime UI) | Minh (Persistence) | Hùng (GUI Base) | Sang (Network & CI) | Checkpoint 21:00 hàng ngày |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CN (01/11)** | Khóa `ReentrantLock(true)` & `observers` vào `Auction` | Tạo Interface `AuctionObserver` | Tạo 4 Custom Exceptions trong `exception/` | Thiết kế `theme.css` Dark theme & `SceneManager` | Định nghĩa `Request`/`Response` JSON Gson | Cả 5 nhánh biên dịch thành công, không ai thiếu interface. |
| **T2 (02/11)** | Viết `AuctionScheduler` đếm ngược đóng phiên | Dựng layout `BiddingRoomView.fxml` bằng SceneBuilder | Viết `PasswordHasher` SHA-256 + 16 bytes Salt | Dựng FXML `LoginView` và `RegisterView` | Nâng cấp hủy phiên trong `AuctionManager` | Mỗi người tự chạy thử độc lập code của mình trên nhánh riêng. |
| **T3 (03/11)** | Sửa `placeBid()` ném `InvalidBidException` | Gắn Controller phòng đấu giá, bắt ngoại lệ Alert | Viết `UserService` (register/login ném ngoại lệ) | Dựng `SellerDashboard` và `AddItemDialog` | Dựng `AuctionServer` cổng 8888 & `ClientHandler` | Socket Server mở cổng 8888 nhận gói tin PING; Test ngoại lệ xanh. |
| **T4 (04/11)** | Viết test 50 threads `AuctionConcurrencyTest` | Viết `AuctionClient` socket nhận push realtime | Viết `GenericDAO` & `JsonStorage` đọc ghi JSON | Nối các Form với `AuctionClient` qua Socket | Hoàn thiện Broadcast; Tích hợp Checkstyle Maven | Đặt giá realtime qua Socket hoạt động; Dữ liệu lưu file JSON. |
| **T5 (05/11)** | Review PR Mảng 2, hỗ trợ fix bug Concurrency | Review PR Mảng 1, ghép nối phòng đấu giá | Review PR Mảng 5, test bền vững file JSON | Review PR Mảng 3, hoàn thiện luồng Scene | **HỌP MERGE 5 PR VÀO `dev`**, chạy full CI GitHub | **MÃ NGUỒN HỢP NHẤT TOÀN DIỆN TRÊN NHÁNH `dev` THÀNH CÔNG.** |
| **T6 (06/11)** | Luyện giải thích Fair Lock, Lost update, Race condition | Luyện giải thích JavaFX Single-Threaded UI | Luyện giải thích SHA-256 + Salt, DAO Pattern | Luyện giải thích mô hình MVC trong JavaFX | **TỔNG DUYỆT DEMO & PHỎNG VẤN THỬ NỘI BỘ (1.5 TIẾNG)** | Cả 5 người tự tin trả lời vấn đáp, kịch bản demo chạy trơn tru. |
| **T7 (07/11)** | Phụ trách demo Concurrency & Timer | Phụ trách demo Realtime nhảy số qua Socket | Phụ trách demo Persistence tắt/bật server | Phụ trách demo luồng giao diện Login/Dashboard | Thuyết trình kiến trúc tổng quan, Class Diagram | **BÁO CÁO TIẾN ĐỘ TRƯỚC GIẢNG VIÊN — ĐẠT ĐIỂM TỐI ĐA!** |

---

# 🔄 PHẦN 5 — MA TRẬN REVIEW CHÉO & QUY TRÌNH MERGE CODE (TỐI THỨ NĂM 05/11)

Mỗi Pull Request bắt buộc phải có ít nhất **2 thành viên khác** review và bấm **Approve** trước khi gộp code:

| PR Cần Merge | Nhánh làm việc | Reviewer 1 (Chính) | Reviewer 2 (Phụ) | Trọng tâm cần soi kỹ khi Review |
| :--- | :--- | :---: | :---: | :--- |
| **PR 1: Minh** | `feature/w7-persistence-auth` | Sang | Hùng | Mật khẩu phải được băm kèm Salt; file `users.json` được tạo ra hợp lệ. |
| **PR 2: Quân** | `feature/w7-concurrency-engine` | Sang | Tuấn | Thân hàm `placeBid()` phải bọc trọn trong `lock()` và `finally { unlock(); }`. |
| **PR 3: Sang** | `feature/w7-network-protocol` | Minh | Quân | ServerSocket cổng 8888 chạy đa luồng không bị treo; Checkstyle 0 lỗi. |
| **PR 4: Hùng** | `feature/w7-gui-base` | Tuấn | Sang | Cửa sổ mở ra hiển thị Dark Theme sắc nét, không bị lỗi FXML loader. |
| **PR 5: Tuấn** | `feature/w7-realtime-bidding-room` | Quân | Hùng | Đặt giá trên Client thì giao diện tự động nhảy số qua `Platform.runLater()`. |

### Thứ tự merge chính xác vào tối Thứ Năm (05/11):
1. **Merge PR 1 (Minh):** Bổ sung Exceptions, Security và Tầng DAO vào `dev`.
2. **Merge PR 2 (Quân):** Bổ sung Concurrency Fair Lock và Scheduler vào `Auction` trên `dev`.
3. **Merge PR 3 (Sang):** Bổ sung Hạ tầng Socket Server và Checkstyle vào `dev`.
4. **Merge PR 4 & 5 (Hùng & Tuấn):** Bổ sung toàn bộ Giao diện FXML và Socket Client vào `dev`.
5. **Kiểm tra tự động:** GitHub Actions trên nhánh `dev` phải báo **BUILD SUCCESS** màu xanh lá.

---

# 🎯 PHẦN 6 — KỊCH BẢN DEMO 5 BƯỚC "ĂN TRỌN ĐIỂM" TRƯỚC GIẢNG VIÊN (07/11)

Khi giảng viên gọi nhóm lên chấm tiến độ, nhóm trưởng Sang làm theo đúng 5 bước sau:

1. **Bước 1 — Báo cáo Cấu trúc Dự án & Chất Lượng Mã Nguồn (1 phút):**
   - Mở file `pom.xml`: Cho thầy xem cấu trúc Maven Multi-Module (`auction-server`, `auction-client`) và plugin `maven-checkstyle-plugin`.
   - Mở GitHub: Cho thầy xem các nhánh feature, danh sách Pull Request có review chéo nghiêm túc, và luồng **GitHub Actions CI đang báo xanh (BUILD SUCCESS)**.
2. **Bước 2 — Khởi động Server & Đăng nhập 2 Client (2 phút):**
   - Bật `ServerApp`: Console log hiển thị rõ ràng Server đang lắng nghe trên cổng 8888.
   - Mở đồng thời 2 cửa sổ `ClientApp` trên màn hình:
     - Cửa sổ 1: Đăng nhập tài khoản **Seller** $\rightarrow$ Tạo một sản phẩm đấu giá mới (Chọn loại Laptop, giá sàn 10.000.000 VNĐ, bước giá 500.000 VNĐ).
     - Cửa sổ 2: Đăng nhập tài khoản **Bidder A** $\rightarrow$ Thấy phiên đấu giá mới xuất hiện tức thời $\rightarrow$ Bấm tham gia vào Phòng đấu giá trực tiếp.
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
