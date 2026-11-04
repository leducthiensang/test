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

## ⚡ BƯỚC 0 — KHỞI TẠO 6 FILE KHUNG MỒI (STUB) TRÊN NHÁNH `dev` (TRIỆT TIÊU MỌI PHỤ THUỘC)

> [!IMPORTANT]
> **Giải pháp kỹ thuật loại bỏ 100% tình trạng "người này ngồi chờ người kia":**  
> Nhóm trưởng Sang (hoặc Minh) tạo nhanh 6 file "vỏ rỗng" dưới đây (mỗi file chỉ 5-10 dòng) và commit thẳng lên nhánh `dev`.  
> Sau đó 4 thành viên chỉ cần gõ: `git pull origin dev` về nhánh của mình. Cả 5 người lập tức có sẵn đầy đủ class để import và gọi lệnh, không bao giờ bị báo lỗi đỏ trong IDE!

### 1. `AuctionObserver.java`
* **Đường dẫn:** `auction-server/src/main/java/com/auction/server/domain/model/auction/observer/AuctionObserver.java`
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

### 2. `AuctionException.java`
* **Đường dẫn:** `auction-server/src/main/java/com/auction/server/exception/AuctionException.java`
```java
package com.auction.server.exception;

public class AuctionException extends RuntimeException {
    public AuctionException(String message) {
        super(message);
    }
    public AuctionException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### 3. `InvalidBidException.java`
* **Đường dẫn:** `auction-server/src/main/java/com/auction/server/exception/InvalidBidException.java`
```java
package com.auction.server.exception;

public class InvalidBidException extends AuctionException {
    public InvalidBidException(String message) {
        super(message);
    }
}
```

### 4. `AuctionClosedException.java`
* **Đường dẫn:** `auction-server/src/main/java/com/auction/server/exception/AuctionClosedException.java`
```java
package com.auction.server.exception;

public class AuctionClosedException extends AuctionException {
    public AuctionClosedException(String message) {
        super(message);
    }
}
```

### 5. `AuthenticationException.java`
* **Đường dẫn:** `auction-server/src/main/java/com/auction/server/exception/AuthenticationException.java`
```java
package com.auction.server.exception;

public class AuthenticationException extends AuctionException {
    public AuthenticationException(String message) {
        super(message);
    }
}
```

### 6. `BiddingRoomView.fxml` (File FXML mồi để Hùng test chuyển Scene)
* **Đường dẫn:** `auction-client/src/main/resources/fxml/BiddingRoomView.fxml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.layout.AnchorPane?>

<AnchorPane prefHeight="400.0" prefWidth="600.0" xmlns="http://javafx.com/javafx/21" xmlns:fx="http://javafx.com/fxml/1">
    <children>
        <Label layoutX="220.0" layoutY="180.0" text="PHÒNG ĐẤU GIÁ TRỰC TIẾP" />
    </children>
</AnchorPane>
```

---

## 👥 PHÂN CÔNG CHI TIẾT 5 THÀNH VIÊN THEO TRỤC CHUYÊN MÔN

---

### 👤 1. ĐỖ MẠNH HÙNG — Mảng 1: Client GUI & Form Flow
* **Nhánh làm việc:** `feature/w7-hung/gui-base` (hoặc `feature/w8-hung/mvc-form-views`)
* **1. Đã có trong Kế hoạch Tuần 2 (`plan/week2.md`):**
  * Viết tài liệu hướng dẫn JavaFX (`docs/javafx-guide.md`).
  * Xây dựng bộ CSS Theme Dark Mode dùng chung (`resources/css/theme.css`).
  * Khởi tạo `ClientApp.java` nạp FXML và `SceneManager.java` quản lý chuyển màn hình.
  * Thiết kế các file FXML cơ bản: `LoginView.fxml`, `RegisterView.fxml`, `SellerDashboard.fxml`, `AddItemDialog.fxml`.
* **2. ⚡ CẦN LÀM THÊM SO VỚI KẾ HOẠCH TUẦN 2 (Bổ sung cho Tuần 8 & 9):**
  * **Xác thực Đăng nhập độc lập (KHÔNG CẦN CHỜ MINH VIẾT USER SERVICE):** Trong `LoginController.java`, Hùng kiểm tra đăng nhập bằng dữ liệu mẫu (Mock data) nội bộ để test giao diện:
    ```java
    // Hùng test trực tiếp trong Controller, không cần chờ UserService của Minh:
    if ("admin".equals(username) && "123".equals(password)) {
        SceneManager.getInstance().switchScene("/fxml/BiddingRoomView.fxml");
    } else {
        throw new AuthenticationException("Sai tên đăng nhập hoặc mật khẩu!");
    }
    ```
  * **Bắt ngoại lệ hiển thị chữ đỏ (Tuần 8):** Bọc khối `try-catch` bắt `AuthenticationException` và hiển thị thông báo lỗi trực tiếp bằng chữ màu đỏ lên nhãn `#lblError`.
  * **Chuyển Scene mượt mà:** Nút Đăng nhập gọi `SceneManager.getInstance().switchScene("/fxml/BiddingRoomView.fxml")` (đã có sẵn file FXML mồi từ Bước 0 nên không bao giờ bị lỗi crash app).
* **3. ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch Tuần 2 (Để không quá tải):**
  * Tạm hoãn trau chuốt các dialog phụ (`AddItemDialog.fxml`, `SellerDashboard.fxml`) và các bộ lọc nâng cao.
  * **Tuyệt đối không nối Socket mạng ngầm vào các Controller ở tuần này** (chạy mock/in-memory data nội bộ để tránh lỗi crash luồng UI).
* **4. ✅ Tiêu chí nghiệm thu:** Chạy `mvn javafx:run` mở ra màn hình Đăng nhập sắc nét; gõ sai hiện thông báo lỗi chữ đỏ; gõ đúng chuyển sang phòng đấu giá không lỗi runtime.

---

### 👤 2. LÊ ANH TUẤN — Mảng 2: Realtime UI & Observer Pattern
* **Nhánh làm việc:** `feature/w7-tuan/realtime-bidding-room`
* **1. Đã có trong Kế hoạch Tuần 2 (`plan/week2.md`):**
  * Tạo interface `AuctionObserver.java` (`onNewBidPlaced`, `onAuctionStatusChanged`).
  * Thiết kế giao diện phòng đấu giá `BiddingRoomView.fxml` (giá cao nhất hiện tại, leading bidder, bảng lịch sử `TableView`, nút đặt giá).
  * Viết `BiddingRoomController.java` implements `AuctionObserver` (nhận callback nhảy số giả lập).
  * Viết Unit Test `AuctionObserverTest.java`.
* **2. ⚡ CẦN LÀM THÊM SO VỚI KẾ HOẠCH TUẦN 2 (Bổ sung cho Tuần 8 & 9):**
  * **Độc lập giao diện (KHÔNG PHẢI CHỜ HÙNG LÀM CSS):** Tuấn thiết kế `BiddingRoomView.fxml` bằng theme mặc định của JavaFX (Modena). FXML và Controller chạy thử nghiệm bình thường, sau này CSS Dark Theme sẽ được tự động nạp qua `SceneManager` khi tích hợp.
  * **Kích hoạt nhảy số Observer độc lập (KHÔNG CẦN CHỜ QUÂN VIẾT ATTACH_OBSERVER):** Trong `BiddingRoomController`, khi bấm nút "Đặt giá thử", Tuấn tự gọi callback Observer của chính mình:
    ```java
    // Tuấn tự gọi callback của mình với 1 đối tượng Bid mẫu:
    Bid dummyBid = new Bid(currentUser, auction.getId(), newAmount);
    this.onNewBidPlaced(auction, dummyBid);
    ```
    *(Giá trên màn hình lập tức nhảy số và thêm 1 dòng vào TableView, chứng minh Controller Observer hoạt động hoàn hảo mà không cần chờ hàm attachObserver của Quân)*.
  * **Bắt ngoại lệ nghiệp vụ hiển thị Popup `Alert` (Tuần 8):** Trong `BiddingRoomController.java`, bọc lệnh gọi đặt giá trong khối `try-catch`:
    * Bắt `InvalidBidException` (đặt giá thấp hơn bước giá, số dư không đủ) $\rightarrow$ hiển thị popup `Alert(AlertType.WARNING)` mang đúng thông điệp `e.getMessage()`.
    * Bắt `AuctionClosedException` (đấu giá khi phiên đã đóng/hết giờ) $\rightarrow$ hiển thị popup `Alert(AlertType.ERROR)`.
    * Bắt `NumberFormatException` (nhập chữ/ký tự lạ vào ô số tiền) $\rightarrow$ hiển thị `Alert` nhắc nhở nhập số hợp lệ.
  * **Đồng hồ đếm ngược trực quan (`javafx.animation.Timeline`):** Cập nhật nhãn đếm ngược `#lblCountdown` mỗi 1 giây (đổi màu đỏ cảnh báo khi còn dưới 1 phút).
* **3. ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch Tuần 2 (Để không quá tải):**
  * Chưa cần viết `AuctionClient` chạy Socket ngầm để nhận push thông báo qua mạng TCP.
* **4. ✅ Tiêu chí nghiệm thu:** Bấm đặt giá hợp lệ thì UI nhảy số giá mới và thêm dòng lịch sử tức thì qua Observer nội bộ; gõ sai giá thì popup Alert cảnh báo bật lên rõ ràng.

---

### 👤 3. LÊ ĐỨC THIỆN SANG (Nhóm trưởng) — Mảng 3: Network Socket & DevOps CI
* **Nhánh làm việc:** `feature/w7-sang/network-protocol`
* **1. Đã có trong Kế hoạch Tuần 2 (`plan/week2.md`):**
  * Giao thức gói tin JSON (`ActionType`, `ResponseStatus`, `Request`, `Response`, `MessageSerializer`).
  * Máy chủ `AuctionServer` lắng nghe cổng 8888 và `ClientHandler` đa luồng tiếp nhận Client, xử lý PING $\rightarrow$ PONG.
  * Lớp khởi chạy `ServerApp.java`.
  * Bộ test tích hợp Socket `EchoClientTest.java`.
* **2. ⚡ CẦN LÀM THÊM SO VỚI KẾ HOẠCH TUẦN 2 (Bổ sung cho Tuần 8 & 9 — ĐÃ HOÀN THÀNH 100%):**
  * **Tích hợp Checkstyle Google Style vào Maven (Tuần 9 - ĐÃ XONG):** Cấu hình `maven-checkstyle-plugin` (Google Java Style) vào `pom.xml` để chạy lệnh `mvn checkstyle:check`.
  * **Bộ kiểm thử JSON hoàn chỉnh (Tuần 9 - ĐÃ XONG):** Viết `MessageSerializerTest.java` kiểm thử 3 trường hợp chuyển đổi Request/Response và giữ `data: null`.
  * **Độc lập tuyệt đối:** Toàn bộ mã nguồn và test của Sang hoạt động độc lập 100%, không phải chờ đợi bất kỳ ai. Sang đã hoàn thành trọn vẹn nhiệm vụ của Tuần 8 & 9.
* **3. ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch Tuần 2 (Để không quá tải):**
  * Bỏ `CachedThreadPool`, dùng `new Thread(...)` cơ bản kinh điển; bỏ các enum action chưa dùng.
  * Tạm hoãn Broadcast push realtime đa client qua Socket (để dành sang Tuần 10).
* **4. ✅ Tiêu chí nghiệm thu:** `mvn test` đỗ 100% các bài test mạng; `mvn checkstyle:check` chạy thành công không lỗi; Server chạy cổng 8888 phản hồi PING/PONG chuẩn xác.

---

### 👤 4. HOÀNG THẾ CÔNG MINH — Mảng 4: Storage, Auth & Exception Core
* **Nhánh làm việc:** `feature/w7-minh/persistence-auth` (hoặc `feature/w8-minh/custom-exceptions-auth`)
* **1. Đã có trong Kế hoạch Tuần 2 (`plan/week2.md`):**
  * Viết tài liệu quy chuẩn code `docs/STYLE_GUIDE.md`.
  * Viết class tiện ích `JsonStorage.java` đọc/ghi file `auction-server/data/users.json`.
  * Cập nhật `User.java` và viết `UserService.java` (hàm `login`, `register`).
  * Viết Unit Test `UserDAOTest.java` và `UserServiceTest.java`.
* **2. ⚡ CẦN LÀM THÊM SO VỚI KẾ HOẠCH TUẦN 2 (Bổ sung cho Tuần 8 - Trọng tâm môn học):**
  * **Tự tạo Cụm Ngoại lệ Xác thực người dùng (Độc lập, không chờ ai):**
    * Tạo package `com.auction.server.exception`:
      * `AuctionException.java` (`extends RuntimeException`): Lớp ngoại lệ cha gốc của hệ thống.
      * `AuthenticationException.java` (`extends AuctionException`): Ném khi đăng nhập sai tài khoản hoặc sai mật khẩu.
  * **Cập nhật `UserService.java` ném ngoại lệ:**
    * Hàm `login(username, password)`: Nếu không tìm thấy user hoặc sai mật khẩu $\rightarrow$ `throw new AuthenticationException("Sai tên đăng nhập hoặc mật khẩu!")`.
    * Hàm `register(user)`: Nếu trùng username $\rightarrow$ `throw new AuctionException("Tên đăng nhập đã tồn tại!")`.
  * **Tự viết bài test JUnit 5 `assertThrows` cho Exception của mình (Tuần 8):**
    * Trong `UserServiceTest.java`, viết test: `assertThrows(AuthenticationException.class, () -> userService.login("sai_user", "123"));`
    * Kiểm tra đăng ký trùng username ném `AuctionException`.
* **3. ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch Tuần 2 (Để không quá tải):**
  * Tạm hoãn dựng mô hình Generic DAO phân tầng phức tạp (`GenericDAO<T>`, `UserDAOImpl`); chỉ dùng `JsonStorage.java` đọc/ghi file `users.json`.
  * Tạm hoãn thuật toán mã hóa SHA-256 + Salt (ngoài barem đề cương).
* **4. ✅ Tiêu chí nghiệm thu:** `AuthenticationException` và `AuctionException` hoàn chỉnh; `UserService` ném đúng exception và bài test `assertThrows` đỗ 100%; file `users.json` lưu trữ người dùng thành công khi chạy test.

---

### 👤 5. TRẦN MẠNH QUÂN — Mảng 5: Concurrency Engine & Business Core
* **Nhánh làm việc:** `feature/w7-quan/concurrency-engine`
* **1. Đã có trong Kế hoạch Tuần 2 (`plan/week2.md`):**
  * Nâng cấp `ReentrantLock(true)` bảo vệ tài nguyên chia sẻ trong `Auction.placeBid()`.
  * Tích hợp Observer Pattern vào `Auction.java` (`CopyOnWriteArrayList<AuctionObserver>`, `attachObserver`, `detachObserver`, `notifyNewBid`).
  * Viết Unit Test Concurrency `AuctionConcurrencyTest.java`.
* **2. ⚡ CẦN LÀM THÊM SO VỚI KẾ HOẠCH TUẦN 2 (Bổ sung cho Tuần 8):**
  * **Tự tạo Ngoại lệ Đấu giá trên nhánh của mình (ĐỘC LẬP 100%, KHÔNG CHỜ MINH):**
    * Quân tự tạo 2 file ngoại lệ chuyên biệt cho nghiệp vụ phòng đấu giá trong package `com.auction.server.exception`:
      * `InvalidBidException.java` (`extends RuntimeException`): Ném khi đặt giá sai bước giá hoặc số dư không đủ.
      * `AuctionClosedException.java` (`extends RuntimeException`): Ném khi cố đặt giá vào phiên không ở trạng thái `RUNNING`.
    * *(Nhờ tự tạo 2 file này trên nhánh mình, code `Auction.java` của Quân không bao giờ bị báo lỗi đỏ, không phải đợi Minh)*.
  * **Refactor ném Custom Exceptions trong `Auction.java` (Tuần 8):**
    * Trong hàm `placeBid()`, thay thế toàn bộ các lệnh `return false` bằng việc ném Custom Exceptions vừa tạo:
      - Nếu phiên không `RUNNING` $\rightarrow$ `throw new AuctionClosedException("Phiên đấu giá đang đóng hoặc đã kết thúc!");`.
      - Nếu số tiền đặt nhỏ hơn giá sàn hoặc bước giá $\rightarrow$ `throw new InvalidBidException("Giá đặt không hợp lệ, phải tăng tối thiểu theo bước giá!");`.
      - Nếu số dư `bidder.getBalance() < amount` $\rightarrow$ `throw new InvalidBidException("Số dư trong tài khoản không đủ để đặt giá!");`.
  * **Tự cập nhật Unit Test kiểm thử ném ngoại lệ trong `AuctionTest.java` (Tuần 8):**
    * Viết test sử dụng `assertThrows(InvalidBidException.class, () -> auction.placeBid(bidder, 50.0));`.
    * Viết test sử dụng `assertThrows(AuctionClosedException.class, () -> auction.placeBid(bidder, 200.0));`.
* **3. ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch Tuần 2 (Để không quá tải):**
  * Tạm hoãn bộ lập lịch chạy ngầm `AuctionScheduler` (`ScheduledExecutorService` đếm ngược tự đóng phiên) sang Tuần 11–12.
  * Giảm quy mô bài test concurrency xuống 10–20 threads cơ bản, không cần stress-test 50 threads `CountDownLatch`.
* **4. ✅ Tiêu chí nghiệm thu:** `AuctionConcurrencyTest` chạy pass 100%; hàm `placeBid()` được bảo vệ an toàn bằng `ReentrantLock` và test `assertThrows` bắt đúng Exception khi vi phạm.

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

### Contract 2: Cụm Ngoại Lệ Nghiệp Vụ (Áp dụng Cách 1 — Phân chia theo Domain Ownership)

> [!TIP]
> **Nguyên tắc phân chia Cách 1:**  
> Ngoại lệ thuộc mảng nghiệp vụ nào thì thành viên phụ trách mảng đó tự tạo và tự test trên nhánh riêng của mình. Không ai phải chờ ai!

* **Nhóm ngoại lệ Người dùng (Hoàng Thế Công Minh sở hữu trên nhánh `feature/w7-minh/...`):**
  * `com.auction.server.exception.AuctionException` (extends `RuntimeException`): Lớp cha gốc.
  * `com.auction.server.exception.AuthenticationException` (extends `AuctionException`): Ném khi đăng nhập sai tài khoản hoặc mật khẩu.
* **Nhóm ngoại lệ Đấu giá (Trần Mạnh Quân sở hữu trên nhánh `feature/w7-quan/...`):**
  * `com.auction.server.exception.InvalidBidException` (extends `RuntimeException`): Ném khi giá đặt nhỏ hơn giá sàn, không đủ bước giá, hoặc số dư không đủ.
  * `com.auction.server.exception.AuctionClosedException` (extends `RuntimeException`): Ném khi cố đặt giá vào phiên không ở trạng thái `RUNNING`.
* **Giao diện bắt lỗi (Hùng & Tuấn):**
  * Hùng bắt `AuthenticationException` hiện chữ đỏ trên `LoginView`.
  * Tuấn bắt `InvalidBidException` & `AuctionClosedException` hiện Popup `Alert` trên `BiddingRoomView`.

*(Khi merge 5 nhánh vào `dev`, 4 file Exception sẽ tự động hợp nhất hoàn hảo vào package `com.auction.server.exception` mà không hề bị xung đột Merge Conflict).*

### Contract 3: Gói Tin Mạng JSON Chuẩn (Sang)
* `Request(action, data)` $\rightarrow$ Chuyển thành 1 dòng JSON:  
  `{"action":"PING","data":{}}`
* `Response(status, message, data)` $\rightarrow$ Chuyển thành 1 dòng JSON:  
  `{"status":"OK","message":"Success","data":"PONG"}`

---

## 🛡️ BẢNG RÀ SOÁT TÍNH ĐỘC LẬP TUYỆT ĐỐI (ZERO-DEPENDENCY MATRIX)

| Thành viên | Trọng tâm công việc | Có phụ thuộc ai không? | Cơ chế đảm bảo độc lập 100% |
| :--- | :--- | :---: | :--- |
| **Sang** (Mạng & CI) | Socket PING/PONG, Checkstyle, Test mạng | 🟢 **KHÔNG** | Đã hoàn thành 100%, commit xong trên nhánh riêng. |
| **Minh** (Auth & Storage) | `AuctionException`, `AuthenticationException`, `UserService`, `users.json` | 🟢 **KHÔNG** | Tự tạo exception của User trên nhánh mình, test `assertThrows` bằng `UserServiceTest`. |
| **Quân** (Đa luồng & Core) | `InvalidBidException`, `AuctionClosedException`, `Auction.java`, Lock đa luồng | 🟢 **KHÔNG** | Tự tạo exception của Auction trên nhánh mình, ném trong `Auction.placeBid()`, test `assertThrows` bằng `AuctionTest`. |
| **Hùng** (GUI Form) | `LoginView.fxml`, Dark theme, SceneManager | 🟢 **KHÔNG** | Bắt `AuthenticationException` (đã có ở Bước 0), test đăng nhập bằng Mock nội bộ, chuyển scene sang file FXML mồi. |
| **Tuấn** (GUI Room) | `BiddingRoomView.fxml`, Controller, Observer callback | 🟢 **KHÔNG** | Dùng `AuctionObserver` và `InvalidBidException` (đã có ở Bước 0), test nhảy số bằng Mock Callback, theme mặc định không chờ Hùng. |

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
| **Thứ 5 (05/11)** | • **Minh:** Tạo 2 class Exception (`AuctionException`, `AuthenticationException`) + `UserService` + viết `JsonStorage.java`.<br>• **Quân:** Tạo 2 class Exception (`InvalidBidException`, `AuctionClosedException`) + đặt `ReentrantLock` và ném Exception trong `Auction.java`.<br>• **Tuấn:** Hoàn thiện `BiddingRoomView.fxml` + gắn Observer nhảy số + Popup Alert.<br>• **Hùng:** Hoàn thiện `LoginView.fxml` + bắt lỗi chữ đỏ.<br>• **Sang:** Rà soát Checkstyle, hỗ trợ nhóm test code. | Toàn bộ 5 nhánh cá nhân code xong các chức năng tinh gọn. |
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

