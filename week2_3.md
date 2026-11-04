# 📋 KẾ HOẠCH TUẦN 2 & 3 (TUẦN 7, 8, 9 HỌC KỲ) — TINH GỌN & ĐÚNG TIẾN ĐỘ
## Bản Kế Hoạch Chuẩn Hóa Theo Đề Cương Giảng Viên — Cột Mốc Nước Rút 07/11/2026

> **Môn học:** Lập trình nâng cao (2627I_UET.CS2043_6)  
> **Dự án:** Hệ thống Đấu giá trực tuyến (Online Auction System) — Nhóm 1  
> **Thời gian thực hiện:** 26/10/2026 – 07/11/2026  
> **Mục tiêu tối thượng:** Bám sát 100% barem chấm điểm của Giảng viên từ **Tuần 9 trở xuống** theo tài liệu `Hướng dẫn Bài tập lớn – LTNC 2026.pdf`. Triệt tiêu hoàn toàn sự quá tải bằng cách **cắt giảm các tính năng vượt cấp (của Tuần 10–13)**, tập trung hoàn thiện mã nguồn độc lập, tự chạy được, tự test được và chuẩn hóa kiểm thử + Checkstyle.

---

## 🧭 NGUYÊN TẮC TINH GỌN (CHỐNG QUÁ TẢI)

1. **Kế thừa nền tảng Tuần 1 (Đã hoàn thành 100%):**
   * Cây kế thừa `User` (`Bidder`, `Seller`, `Admin`), `Item` (`Electronics`, `Art`, `Vehicle`).
   * 2 Design Patterns: **Singleton** (`AuctionManager`) và **Factory Method** (`ItemFactory`).
   * Khung Maven Multi-Module (`auction-server`, `auction-client`) và GitHub Actions CI.
2. **Cắt giảm triệt để các phần "Làm quá" (Chuyển sang Tuần 10–13):**
   * ❌ **Không ép JavaFX nối Socket mạng ở tuần này:** 2 bạn JavaFX chạy trực tiếp trên bộ nhớ / Mock Observer nội bộ. (Việc nối Socket toàn diện thuộc về Tuần 11).
   * ❌ **Không làm Broadcast realtime qua Socket:** Phía Server chỉ cần trả lời `PING` $\rightarrow$ `PONG` (Việc Broadcast push thuộc về Tuần 10).
   * ❌ **Không dựng kiến trúc Generic DAO phức tạp:** Chỉ dùng 1 class `JsonStorage` đọc/ghi file `data/users.json` bằng Gson (Việc hoàn thiện lưu trữ thuộc về Tuần 11–12).
   * ❌ **Tạm hoãn `AuctionScheduler` ngầm và stress-test 50 luồng:** Chỉ cần dùng `ReentrantLock` và test 10–20 threads cơ bản.
3. **Bổ sung đúng 2 mảnh ghép cốt lõi còn thiếu của Tuần 8 & 9:**
   * ✅ **Custom Exceptions + JUnit 5 `assertThrows`** (Trọng tâm Tuần 8).
   * ✅ **Cấu hình `maven-checkstyle-plugin` vào `pom.xml`** (Trọng tâm Tuần 9).

---

## 👥 PHÂN CÔNG CHI TIẾT 5 THÀNH VIÊN THEO TRỤC CHUYÊN MÔN

---

### 👤 1. ĐỖ MẠNH HÙNG — Mảng 1: Client GUI & Form Flow
* **Nhánh làm việc:** `feature/w7-hung/gui-base` (hoặc `feature/w8-hung/mvc-form-views`)
* **Trọng tâm Tuần 7–9 (CẦN LÀM):**
  1. **Hoàn thiện `LoginView.fxml` & `LoginController.java`:**
     * Ô nhập Username, Password, nút "Đăng nhập", Label `#lblError` thông báo lỗi.
     * Áp dụng bộ giao diện **Dark Theme CSS** (`resources/css/theme.css`).
  2. **Áp dụng mô hình MVC & Xử lý ngoại lệ (Tuần 8):**
     * Khi người dùng nhập sai tài khoản hoặc để trống $\rightarrow$ Controller bắt `AuthenticationException` và hiển thị chữ đỏ thông báo lỗi trực tiếp lên `#lblError`.
     * Khi đăng nhập thành công $\rightarrow$ Chuyển Scene sang màn hình đấu giá.
  3. **Tài liệu hướng dẫn JavaFX (`docs/javafx-guide.md`):** Tóm tắt cách dùng SceneBuilder và liên kết Controller.
* ⏸️ **VIỆC HOÃN LẠI (Sang Tuần 10–11):** Không cần nối Socket mạng ngầm; không cần trau chuốt quá nhiều màn hình phụ (Seller, Dialogs).
* ✅ **Tiêu chí nghiệm thu:** Chạy `mvn javafx:run` mở ra màn hình Đăng nhập sắc nét; gõ sai hiện thông báo lỗi; gõ đúng chuyển màn hình mượt mà không crash.

---

### 👤 2. LÊ ANH TUẤN — Mảng 2: Realtime UI & Observer Pattern
* **Nhánh làm việc:** `feature/w7-tuan/realtime-bidding-room`
* **Trọng tâm Tuần 7–9 (CẦN LÀM):**
  1. **Tạo Interface Observer (`domain.model.auction.observer`):**
     ```java
     public interface AuctionObserver {
         void onNewBidPlaced(Auction auction, Bid newBid);
         void onAuctionStatusChanged(Auction auction, AuctionStatus newStatus);
     }
     ```
  2. **Giao diện phòng đấu giá `BiddingRoomView.fxml`:**
     * Hiển thị: Tên sản phẩm, Giá cao nhất hiện tại (chữ to, nổi bật), Người đang dẫn đầu, Bảng lịch sử đặt giá (`TableView`).
     * Ô nhập số tiền và nút bấm `ĐẶT GIÁ NGAY`.
  3. **Tích hợp Observer nội bộ & Xử lý ngoại lệ (Tuần 8):**
     * `BiddingRoomController` implements `AuctionObserver`.
     * Khi bấm nút Đặt giá $\rightarrow$ gọi `auction.placeBid(...)` $\rightarrow$ UI tự động nhảy số tiền mới thông qua callback `onNewBidPlaced` (Chứng minh Observer hoạt động).
     * Bắt ngoại lệ `InvalidBidException` hoặc `AuctionClosedException` $\rightarrow$ Mở popup `Alert(AlertType.WARNING)` cảnh báo.
  4. **Kiểm thử Observer:** Viết file `AuctionObserverTest.java` kiểm tra callback nhận đúng sự kiện.
* ⏸️ **VIỆC HOÃN LẠI (Sang Tuần 10–11):** Chưa cần viết Socket Client nhận push dữ liệu từ mạng.
* ✅ **Tiêu chí nghiệm thu:** Mở màn hình đấu giá, bấm đặt giá hợp lệ thì giá nhảy số tức thì; gõ giá thấp hơn giá sàn thì hiện Popup Alert báo lỗi.

---

### 👤 3. LÊ ĐỨC THIỆN SANG (Nhóm trưởng) — Mảng 3: Network Socket & DevOps CI
* **Nhánh làm việc:** `feature/w7-sang/network-protocol`
* **Trọng tâm Tuần 7–9 (CẦN LÀM - ĐÃ HOÀN THÀNH 100%):**
  1. **Khung giao thức JSON (`network.protocol`):**
     * `ActionType` (`LOGIN`, `REGISTER`, `PING`), `ResponseStatus` (`OK`, `ERROR`).
     * `Request.java` và `Response.java` đóng gói dữ liệu.
     * `MessageSerializer.java` sử dụng `GsonBuilder().serializeNulls()` chuyển đổi JSON 1 dòng cho Socket.
  2. **Máy chủ Socket đa luồng (`network.server`):**
     * `AuctionServer.java`: Mở cổng 8888, chấp nhận kết nối qua `new Thread(new ClientHandler(socket)).start()`.
     * `ClientHandler.java`: Đọc dòng JSON qua UTF-8, giải mã và phản hồi `PING` $\rightarrow$ `PONG`.
     * `ServerApp.java`: Lớp khởi chạy máy chủ.
  3. **Kiểm thử tự động & Checkstyle:**
     * `MessageSerializerTest.java`: Kiểm thử chuyển đổi JSON 3 trường hợp.
     * `EchoClientTest.java`: Tự động bật server, kết nối socket, gửi PING nhận PONG.
     * Cấu hình `maven-checkstyle-plugin` (Google Style) vào `pom.xml`.
* ⏸️ **VIỆC HOÃN LẠI (Sang Tuần 10):** Không làm `CachedThreadPool` phức tạp; không làm Broadcast realtime qua Socket ở tuần này.
* ✅ **Tiêu chí nghiệm thu:** `mvn test` đỗ 100% các bài test mạng; `mvn checkstyle:check` chạy thành công.

---

### 👤 4. HOÀNG THẾ CÔNG MINH — Mảng 4: Storage, Auth & Exception Core
* **Nhánh làm việc:** `feature/w7-minh/persistence-auth` (hoặc `feature/w8-minh/custom-exceptions-auth`)
* **Trọng tâm Tuần 7–9 (CẦN LÀM):**
  1. **Xây dựng Cụm Custom Exceptions (Tuần 8 - Trọng tâm môn học):**
     * Tạo package `com.auction.server.exception`:
       * `AuctionException.java` (`extends RuntimeException`): Lớp ngoại lệ cha.
       * `InvalidBidException.java`: Ném khi giá đặt nhỏ hơn giá sàn / bước giá, hoặc không đủ số dư.
       * `AuctionClosedException.java`: Ném khi cố đặt giá vào phiên không ở trạng thái `RUNNING`.
       * `AuthenticationException.java`: Ném khi sai tài khoản, sai mật khẩu.
  2. **Cập nhật `UserService.java`:**
     * Hàm `login(username, password)`: Kiểm tra tài khoản, nếu sai $\rightarrow$ `throw new AuthenticationException("Sai tên đăng nhập hoặc mật khẩu!")`.
     * Hàm `register(user)`: Nếu trùng username $\rightarrow$ `throw new AuctionException("Tên đăng nhập đã tồn tại!")`.
  3. **Lưu trữ dữ liệu ra file JSON (`JsonStorage.java` - Tuần 9):**
     * Viết class tiện ích `JsonStorage.java` sử dụng Gson đọc và ghi danh sách `User` ra file `auction-server/data/users.json`.
     * Viết 1 bài test JUnit kiểm tra lưu và tải dữ liệu người dùng không bị mất mát.
* ⏸️ **VIỆC HOÃN LẠI (Sang Tuần 11–12):** Không dựng `GenericDAO` phân tầng phức tạp; không cần mã hóa SHA-256 + Salt (ngoài barem); chưa cần lưu `auctions.json`.
* ✅ **Tiêu chí nghiệm thu:** 4 class Exception hoàn chỉnh; file `users.json` được sinh ra khi tạo user mới; `UserServiceTest` pass 100%.

---

### 👤 5. TRẦN MẠNH QUÂN — Mảng 5: Concurrency Engine & Business Core
* **Nhánh làm việc:** `feature/w7-quan/concurrency-engine`
* **Trọng tâm Tuần 7–9 (CẦN LÀM):**
  1. **An toàn Đa luồng cho `Auction.java` (Tuần 7):**
     * Đặt `private final ReentrantLock lock = new ReentrantLock(true);` bọc toàn bộ khối kiểm tra và cập nhật giá trong hàm `placeBid()`.
     * Sử dụng cấu trúc chuẩn: `lock.lock(); try { ... } finally { lock.unlock(); }`.
  2. **Tích hợp Observer Pattern vào `Auction.java` (Tuần 7):**
     * Danh sách Observer an toàn luồng: `private final List<AuctionObserver> observers = new CopyOnWriteArrayList<>();`.
     * Các hàm: `attachObserver(AuctionObserver o)`, `detachObserver(AuctionObserver o)`.
     * Khi đặt giá thành công trong `placeBid()` $\rightarrow$ lập tức gọi `notifyNewBid(newBid)` tới các observers.
  3. **Refactor Ném Ngoại Lệ Trong `Auction.java` (Tuần 8):**
     * Thay thế việc `return false` bằng cách ném Custom Exceptions của Minh:
       * Phiên không chạy $\rightarrow$ `throw new AuctionClosedException("Phiên đấu giá đang đóng!");`.
       * Bước giá không đủ $\rightarrow$ `throw new InvalidBidException("Giá đặt không hợp lệ!");`.
  4. **Kiểm thử Concurrency cơ bản (`AuctionConcurrencyTest.java`):**
     * Cho 10–20 threads đại diện cho các Bidder cùng gọi `placeBid()` đồng thời qua `ExecutorService`.
     * Khẳng định không xảy ra race condition, giá cao nhất cuối cùng khớp đúng chuẩn.
* ⏸️ **VIỆC HOÃN LẠI (Sang Tuần 12–13):** Không cần lập trình `AuctionScheduler` chạy ngầm phức tạp; không cần stress-test 50 luồng `CountDownLatch`.
* ✅ **Tiêu chí nghiệm thu:** `AuctionConcurrencyTest` chạy pass 100%; `placeBid()` ném đúng ngoại lệ khi vi phạm nghiệp vụ.

---

## 🤝 CÁC GIAO KÈO KỸ THUẬT (CONTRACTS) THỐNG NHẤT GIỮA 5 THÀNH VIÊN

### Contract 1: Interface Observer (Tuấn & Quân)
```java
package com.auction.server.domain.model.auction.observer;

import com.auction.server.domain.model.auction.Auction;
import com.auction.server.domain.model.auction.AuctionStatus;
import com.auction.server.domain.model.auction.Bid;

public interface AuctionObserver {
    void onNewBidPlaced(Auction auction, Bid newBid);
    void onAuctionStatusChanged(Auction auction, AuctionStatus newStatus);
}
```

### Contract 2: Cụm Ngoại Lệ Nghiệp Vụ (Minh, Quân, Hùng, Tuấn)
* `com.auction.server.exception.AuctionException` (extends `RuntimeException`)
* `com.auction.server.exception.InvalidBidException` (extends `AuctionException`)
* `com.auction.server.exception.AuctionClosedException` (extends `AuctionException`)
* `com.auction.server.exception.AuthenticationException` (extends `AuctionException`)

### Contract 3: Gói Tin Mạng JSON Chuẩn (Sang)
* `Request(action, data)` $\rightarrow$ Chuyển thành 1 dòng JSON:  
  `{"action":"PING","data":{}}`
* `Response(status, message, data)` $\rightarrow$ Chuyển thành 1 dòng JSON:  
  `{"status":"OK","message":"Success","data":"PONG"}`

---

## 📊 BẢNG ĐỐI CHIẾU TIÊU CHÍ CHẤM ĐIỂM (MỐC TUẦN 9)

| Tiêu chí trong Barem (Trang 6 PDF) | Điểm | Hiện trạng nhóm đáp ứng | Trạng thái |
| :--- | :---: | :--- | :---: |
| **1. Thiết kế lớp và cây kế thừa** | **0.5đ** | Đã xong từ Tuần 1 (`User`, `Item`, `Entity`, `Bid`, `Auction`). | ✅ 100% |
| **2. Áp dụng OOP (4 tính chất)** | **1.0đ** | Đã xong từ Tuần 1 (Đóng gói, Đa hình `displayProfile`, Trừu tượng). | ✅ 100% |
| **3. Design Patterns phù hợp** | **1.0đ** | **Singleton** (`AuctionManager`), **Factory** (`ItemFactory`), **Observer** (`AuctionObserver`). | ✅ 100% |
| **4. Quản lý người dùng, sản phẩm** | **1.0đ** | CRUD User, CRUD Item, phân quyền Bidder / Seller / Admin. | ✅ 100% |
| **5. Chức năng đấu giá** | **1.0đ** | Logic đặt giá, kiểm tra bước giá, đổi trạng thái phòng. | ✅ 100% |
| **6. Xử lý lỗi & ngoại lệ** | **1.0đ** | 4 Custom Exceptions + `assertThrows` + Alert popup JavaFX. | ✅ 100% (Hoàn thiện đợt này) |
| **7. Xử lý đấu giá đồng thời** | **1.0đ** | `ReentrantLock` bảo vệ `placeBid()` + Unit Test đồng thời. | ✅ 100% |
| **8. Realtime update (Observer/Socket)** | **0.5đ** | Observer Pattern nhảy số thời gian thực trên UI. | ✅ 100% |
| **9. Kiến trúc Client–Server** | **0.5đ** | `AuctionServer` lắng nghe port 8888, tiếp nhận Client gửi PING/PONG. | ✅ 100% |
| **10. MVC (JavaFX + FXML + Model)** | **0.5đ** | FXML View tách biệt Controller, gọi sang Model. | ✅ 100% |
| **11. Maven & Coding convention** | **0.5đ** | Maven Multi-Module + `maven-checkstyle-plugin` Google Style. | ✅ 100% |
| **12. Unit Test (JUnit 5)** | **0.5đ** | Bộ test phủ khắp Domain, Network, Observer, Exception. | ✅ 100% |
| **13. CI/CD (GitHub Actions)** | **0.5đ** | File `.github/workflows/maven-ci.yml` tự động build và chạy test. | ✅ 100% |
| **TỔNG ĐIỂM BẮT BUỘC** | **10.0đ** | **ĐẠT TRỌN VẸN 10/10 ĐIỂM BẮT BUỘC TUẦN 9** | 🎯 **XUẤT SẮC** |

---

## ⏰ TIMELINE HÀNH ĐỘNG 3 NGÀY NƯỚC RÚT (05/11 – 07/11)

| Thời gian | Trọng tâm từng thành viên | Mục tiêu cuối ngày |
| :--- | :--- | :--- |
| **Thứ 5 (05/11)** | • **Minh:** Tạo 4 class Custom Exception + viết `JsonStorage.java`.<br>• **Quân:** Đặt `ReentrantLock` và ném Exception trong `Auction.java`.<br>• **Tuấn:** Hoàn thiện `BiddingRoomView.fxml` + gắn Observer nhảy số.<br>• **Hùng:** Hoàn thiện `LoginView.fxml` + bắt lỗi chữ đỏ.<br>• **Sang:** Rà soát Checkstyle, hỗ trợ nhóm test code. | Toàn bộ 5 nhánh cá nhân code xong các chức năng tinh gọn. |
| **Thứ 6 (06/11)** | • Chạy toàn bộ Unit Test trên máy từng người: `mvn clean test`.<br>• Mở Pull Request từ nhánh cá nhân vào nhánh `dev`.<br>• Tiến hành review chéo theo ma trận và approve PR. | Merge thành công 5 PR vào nhánh `dev`. GitHub Actions báo xanh 100%. |
| **Thứ 7 (07/11)** | • **HỌP NHÓM & TỔNG DUYỆT BÁO CÁO:**<br>1. Demo luồng chạy ứng dụng JavaFX (Login $\rightarrow$ Đặt giá nhảy số $\rightarrow$ Báo lỗi popup).<br>2. Demo Server Socket phản hồi PING/PONG.<br>3. Kiểm tra câu hỏi vấn đáp miệng theo tài liệu `plan/qa-van-dap.md`. | **Sẵn sàng 100% cho Giảng viên chấm tiến độ Tuần 9!** |

---

## 🔄 MA TRẬN REVIEW CHÉO PULL REQUEST TRƯỚC KHI MERGE

| Nhánh Pull Request | Tác giả | Reviewer 1 (Bắt buộc) | Reviewer 2 (Bắt buộc) |
| :--- | :---: | :---: | :---: |
| `feature/w7-hung/gui-base` | **Hùng** | Tuấn | Minh |
| `feature/w7-tuan/realtime-bidding-room` | **Tuấn** | Quân | Hùng |
| `feature/w7-sang/network-protocol` | **Sang** | Minh | Quân |
| `feature/w7-minh/persistence-auth` | **Minh** | Sang | Hùng |
| `feature/w7-quan/concurrency-engine` | **Quân** | Sang | Tuấn |

