# 📋 KẾ HOẠCH CHI TIẾT TUẦN 2 & 3 (TUẦN 7, 8, 9 HỌC KỲ) — TINH GỌN & ĐÚNG TIẾN ĐỘ
## Chủ đề: Chuẩn Hóa Cốt Lõi Nghiệp Vụ, Ngoại Lệ & Kiểm Thử Theo Đề Cương Giảng Viên
### Cột Mốc Nước Rút Về Đích Đợt 1: 07/11/2026 (Đạt Trọn Vẹn 10.0/10.0 Điểm Bắt Buộc)

> **Môn học:** Lập trình nâng cao (2627I_UET.CS2043_6)  
> **Dự án:** Hệ thống Đấu giá trực tuyến (Online Auction System) — Nhóm 1  
> **Thời gian thực hiện:** 26/10/2026 – 07/11/2026  
> **Mô hình triển khai:** **5 Trục Chuyên Môn Độc Lập (Zero-Dependency Architecture)**  
> **Mục tiêu cốt lõi:**  
> 1. Triệt tiêu hoàn toàn sự phụ thuộc chéo ("người này ngồi chờ người kia"). Mỗi thành viên sở hữu một khối mã nguồn tự chạy được, tự kiểm thử được (`mvn test`) trên nhánh riêng.  
> 2. Bám sát 100% barem chấm điểm của Giảng viên từ **Tuần 9 trở xuống** theo tài liệu `Hướng dẫn Bài tập lớn – LTNC 2026.pdf`.  
> 3. Cắt giảm các tính năng vượt cấp (của Tuần 10–13) để không bị quá tải, dồn lực hoàn thiện Custom Exceptions, JUnit 5 kiểm thử ngoại lệ và quy chuẩn Google Checkstyle.  
> **Deadline đẩy code & mở PR trên nhánh cá nhân:** Thứ 6 ngày 06/11/2026, 23:59  
> **Họp nhóm, tổng duyệt demo & vấn đáp miệng:** Thứ 7 ngày 07/11/2026  

---

## 🧭 NGUYÊN TẮC TINH GỌN (CHỐNG QUÁ TẢI CHO CẢ NHÓM)

1. **Kế thừa nền tảng Tuần 1 (Đã hoàn thành 100%):**
   * Cây kế thừa Domain Models: `User` (`Bidder`, `Seller`, `Admin`), `Item` (`Electronics`, `Art`, `Vehicle`), `Auction`, `Bid`.
   * 2 Design Patterns: **Singleton** (`AuctionManager`) và **Factory Method** (`ItemFactory`).
   * Khung Maven Multi-Module (`auction-server`, `auction-client`) và quy trình CI tự động build trên GitHub Actions.
2. **Cắt giảm triệt để các phần "Làm quá" (Dời sang Tuần 10–13 sau khi đã chấm xong Tuần 9):**
   * ❌ **Không ép JavaFX nối Socket mạng ở tuần này:** 2 bạn làm JavaFX chạy thử nghiệm trực tiếp trên bộ nhớ / Mock Observer nội bộ. (Việc nối Socket toàn diện thuộc về Tuần 11).
   * ❌ **Không làm Broadcast realtime đa Client qua Socket:** Phía Server chỉ cần tiếp nhận kết nối và trả lời `PING` $\rightarrow$ `PONG` (Việc Broadcast push thuộc về Tuần 10).
   * ❌ **Không dựng kiến trúc Generic DAO phân tầng phức tạp:** Chỉ dùng 1 class `JsonStorage` đọc/ghi file `data/users.json` bằng Gson (Việc hoàn thiện lưu trữ toàn diện thuộc về Tuần 11–12).
   * ❌ **Tạm hoãn `AuctionScheduler` ngầm và stress-test 50 luồng:** Chỉ cần dùng `ReentrantLock` và test 10–20 threads cơ bản.
3. **Bổ sung đúng 2 mảnh ghép cốt lõi còn thiếu của Tuần 8 & 9:**
   * ✅ **Custom Exceptions + JUnit 5 `assertThrows`** (Trọng tâm Tuần 8).
   * ✅ **Cấu hình `maven-checkstyle-plugin` vào `pom.xml` theo Google Java Style** (Trọng tâm Tuần 9).

---

## ⚡ BƯỚC 0 — KHỞI TẠO 6 FILE KHUNG MỒI (STUB) TRÊN NHÁNH `dev` (ĐÃ HOÀN THÀNH 100%)

> [!IMPORTANT]
> **Trạng thái thực tế:**  
> Nhóm trưởng Lê Đức Thiện Sang đã khởi tạo và commit thành công toàn bộ **6 file khung mồi (stub)** dưới đây trực tiếp lên nhánh `dev`.  
> Giờ đây, 4 thành viên chỉ cần gõ lệnh sau trên máy mình để kéo mã nguồn về nhánh làm việc:  
> ```bash
> git pull origin dev
> ```
> Toàn bộ class, interface và view dùng chung đã nằm sẵn trong dự án. Cả 5 thành viên lập tức có sẵn đầy đủ mã nguồn để import và gọi lệnh, **không ai phải tự tạo lại file và không bao giờ bị báo lỗi đỏ trong IDE!**

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

## 📌 PHẦN 1 — PHẦN CHUNG: TỰ HỌC NỀN TẢNG & VẤN ĐÁP MIỆNG (TUẦN 8 & 9)

Mỗi thành viên cần nắm vững các khối kiến thức dưới đây để phục vụ bài kiểm tra vấn đáp miệng của Giảng viên khi chấm tiến độ Tuần 9.

### 📊 Bảng tổng hợp kỹ năng cốt lõi Tuần 8 & 9

| # | Khối kiến thức kỹ thuật | Trọng tâm áp dụng | Đầu ra kiểm tra (Thực chứng trong mã nguồn) |
| :-: | :--- | :---: | :--- |
| **1** | **Cây phân cấp Ngoại lệ (Custom Exception Hierarchy)** | **Minh, Quân, Hùng, Tuấn** | Tạo lớp cha `AuctionException` kế thừa `RuntimeException`, các ngoại lệ con kế thừa rõ ràng theo domain; phân biệt được Checked vs Unchecked Exception. |
| **2** | **Kiểm thử Ngoại lệ JUnit 5 (`assertThrows`)** | **Minh, Quân** | Viết bài test bắt đúng Exception class và kiểm tra nội dung thông điệp `e.getMessage()`. |
| **3** | **Kiểm thử Giao diện Bắt Ngoại lệ (UI Error Handling)** | **Hùng, Tuấn** | Xử lý lỗi thân thiện với người dùng: hiển thị Label chữ đỏ trên form hoặc bật Popup `Alert(AlertType.WARNING)`. |
| **4** | **Checkstyle & Google Java Style** | **Cả 5 thành viên** | Cấu hình plugin Maven; hiểu các quy chuẩn: thụt lề 2 spaces, import không dùng `*`, format ngoặc nhọn `{}` cùng dòng. Chạy `mvn checkstyle:check` không lỗi. |
| **5** | **Observer Pattern trong MVC** | **Tuấn, Quân** | Subject (`Auction`) duy trì danh sách `AuctionObserver`, kích hoạt `notifyNewBid()`; Controller đóng vai trò Observer nhận callback để cập nhật JavaFX. |
| **6** | **An toàn Đa luồng (Thread-Safety) với `ReentrantLock`** | **Quân** | Sử dụng Fair Lock `new ReentrantLock(true)` bảo vệ hàm đặt giá; giải phóng khóa trong khối `finally`. |
| **7** | **Giao thức Mạng JSON hướng dòng (Single-line Protocol)** | **Sang** | Tuân thủ chuẩn Single-line JSON kết thúc bằng `\n` và `autoFlush = true`, không gây tắc nghẽn luồng đọc `in.readLine()`. |

---

### ❓ Bộ 8 câu hỏi vấn đáp miệng điển hình (Có hướng dẫn trả lời ngắn gọn)

1. **Tại sao hệ thống nên tự định nghĩa Custom Exception (`AuctionException`) thay vì ném chung chung `RuntimeException` hay `Exception`?**  
   * *Trả lời:* Giúp phân loại lỗi rõ ràng theo từng ngữ cảnh nghiệp vụ (đăng nhập sai, đấu giá sai bước giá, phòng đã đóng); cho phép tầng Controller bắt đúng kiểu ngoại lệ cụ thể để hiển thị thông báo chính xác cho người dùng mà không bắt nhầm các lỗi hệ thống khác.

2. **Tại sao các Custom Exception trong dự án này kế thừa `RuntimeException` (Unchecked Exception) mà không kế thừa `Exception` (Checked Exception)?**  
   * *Trả lời:* `RuntimeException` giúp mã nguồn gọn gàng, linh hoạt, không ép buộc mọi tầng trung gian phải khai báo `throws` rườm rà trong signature của hàm, trong khi vẫn có thể chủ động bắt (`try-catch`) ở tầng UI hoặc Server Handler khi cần thiết.

3. **Trong JUnit 5, hàm `assertThrows` hoạt động như thế nào?**  
   * *Trả lời:* `assertThrows(ExpectedException.class, executable)` nhận vào kiểu class ngoại lệ mong muốn và một biểu thức Lambda thực thi đoạn code gây lỗi. Bài test sẽ PASS nếu đoạn code ném ra đúng ngoại lệ đó (hoặc lớp con của nó), và FAIL nếu không có ngoại lệ nào được ném ra hoặc ném sai loại.

4. **Sự khác biệt giữa việc hiển thị thông báo lỗi qua Label chữ đỏ và bật Popup `Alert` trong JavaFX là gì? Khi nào nên dùng loại nào?**  
   * *Trả lời:* Label chữ đỏ thường dùng cho các lỗi kiểm tra dữ liệu form nhập liệu tại chỗ (form validation như sai mật khẩu, bỏ trống ô) để tránh làm gián đoạn thao tác người dùng. Popup `Alert` dùng cho các cảnh báo quan trọng, vi phạm quy tắc nghiệp vụ tức thời (như đặt giá không hợp lệ, phiên đấu giá vừa kết thúc).

5. **`ReentrantLock(true)` (Fair Lock) khác gì so với `synchronized` thông thường? Tại sao lại quan trọng trong đấu giá?**  
   * *Trả lời:* `synchronized` không đảm bảo thứ tự ưu tiên (luồng nào chiếm CPU trước sẽ vào trước). Trong khi đó, Fair Lock (`new ReentrantLock(true)`) đảm bảo tính công bằng (FIFO) — luồng nào gọi lệnh đặt giá trước sẽ được cấp khóa trước, ngăn chặn tình trạng người bấm sau lại được ghi nhận trước người bấm trước.

6. **Tại sao gói tin JSON truyền qua Socket lại bắt buộc phải là Single-line JSON kèm `\n`?**  
   * *Trả lời:* Phương thức `BufferedReader.readLine()` của Socket đọc dữ liệu theo từng dòng (dừng lại khi gặp ký tự `\n`). Nếu gửi JSON dạng Pretty Print (nhiều dòng), phía nhận sẽ đọc từng mảnh vỡ và không thể phân tích cú pháp thành một đối tượng hoàn chỉnh, dẫn đến treo luồng hoặc lỗi cú pháp JSON.

7. **Checkstyle là gì và tại sao nhóm lại tích hợp `maven-checkstyle-plugin` vào đồ án?**  
   * *Trả lời:* Checkstyle là công cụ phân tích tĩnh (Static Analysis) giúp kiểm tra mã nguồn có tuân thủ các quy chuẩn định dạng (như Google Java Style) hay không. Nhóm tích hợp vào Maven để toàn bộ 5 thành viên đều viết code đồng nhất, sạch đẹp, chuyên nghiệp theo đúng tiêu chuẩn đánh giá của Giảng viên.

8. **Trong mô hình MVC của JavaFX, Controller kết nối với View FXML như thế nào?**  
   * *Trả lời:* File FXML khai báo thuộc tính `fx:controller="com.auction.client.controller.LoginController"`. Các control trên giao diện được định danh bằng `fx:id` và ánh xạ tương ứng vào các thuộc tính có annotation `@FXML` trong Controller. Các sự kiện click chuột được gắn qua thuộc tính `onAction="#handleMethod"`.

---

## 🤝 PHẦN 2 — CÁC GIAO KÈO KỸ THUẬT (CONTRACTS) THỐNG NHẤT GIỮA 5 THÀNH VIÊN

> [!IMPORTANT]
> Đây là bản giao kèo kỹ thuật chính thức. Mọi thành viên chỉ cần tuân thủ đúng tên lớp, tên phương thức và định dạng dữ liệu dưới đây là có thể tự tin code độc lập 100% trên nhánh riêng mà không sợ bị xung đột khi tích hợp vào nhánh `dev`.

### Contract 1: Interface Observer (`AuctionObserver`) — Giao tiếp giữa Tuấn & Quân
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

### Contract 2: Cụm Ngoại Lệ Nghiệp Vụ (Đã có sẵn từ Bước 0 trên `dev`)
* **Lớp cha gốc:**  
  `com.auction.server.exception.AuctionException` (extends `RuntimeException`)
* **Phân vùng Ngoại lệ Người dùng (Minh quản lý logic nghiệp vụ):**  
  `com.auction.server.exception.AuthenticationException` (extends `AuctionException`): Ném khi đăng nhập sai tài khoản hoặc mật khẩu.
* **Phân vùng Ngoại lệ Đấu giá (Quân quản lý logic nghiệp vụ):**  
  * `com.auction.server.exception.InvalidBidException` (extends `AuctionException`): Ném khi giá đặt nhỏ hơn giá sàn, không đủ bước giá, hoặc số dư ví không đủ.
  * `com.auction.server.exception.AuctionClosedException` (extends `AuctionException`): Ném khi cố đặt giá vào phiên không ở trạng thái `RUNNING`.
* **Giao diện bắt ngoại lệ (Hùng & Tuấn):**  
  * Hùng bắt `AuthenticationException` hiển thị thông báo chữ đỏ lên `#lblError`.
  * Tuấn bắt `InvalidBidException` và `AuctionClosedException` hiển thị popup `Alert(AlertType.WARNING/ERROR)`.

### Contract 3: Gói Tin Mạng JSON Chuẩn (Sang)
* Mọi gói tin truyền qua Socket bắt buộc nằm trên 1 dòng đơn kết thúc bằng `\n`:
* **Request mẫu:**  
  `{"action":"PING","data":{}}`
* **Response mẫu:**  
  `{"status":"OK","message":"Success","data":"PONG"}`

---

## 🌳 PHẦN 3 — CÂY THƯ MỤC NÂNG CẤP DỰ KIẾN (MỐC TUẦN 9)

```text
online-auction-system/
├── .github/
│   └── workflows/
│       └── maven-ci.yml                           # CI GitHub Actions tự động build & test
├── docs/
│   ├── STYLE_GUIDE.md                             # [Mảng 4 - Minh] Quy chuẩn code, AAA testing
│   └── javafx-guide.md                            # [Mảng 1 - Hùng] Hướng dẫn SceneBuilder & FXML
├── pom.xml                                        # [Mảng 3 - Sang] Cấu hình Checkstyle Google Style
│
├── auction-server/
│   ├── data/
│   │   └── users.json                             # [Mảng 4 - Minh] File lưu trữ danh sách User JSON
│   ├── pom.xml
│   └── src/
│       ├── main/java/com/auction/server/
│       │   ├── ServerApp.java                     # [Mảng 3 - Sang] Điểm khởi chạy Server cổng 8888
│       │   ├── domain/model/                      # Entity, User, Item, Auction, Bid (Tuần 1)
│       │   │   ├── auction/
│       │   │   │   ├── Auction.java               # [Mảng 5 - Quân] ReentrantLock & ném Exceptions
│       │   │   │   ├── AuctionStatus.java
│       │   │   │   ├── Bid.java
│       │   │   │   └── observer/
│       │   │   │       └── AuctionObserver.java   # [Bước 0 - ĐÃ CÓ TRÊN DEV]
│       │   │   └── user/
│       │   │       └── User.java
│       │   ├── exception/                         # Cụm Custom Exceptions (Tuần 8)
│       │   │   ├── AuctionException.java          # [Bước 0 - ĐÃ CÓ TRÊN DEV] Lớp cha gốc
│       │   │   ├── AuthenticationException.java   # [Bước 0 - ĐÃ CÓ TRÊN DEV] Ngoại lệ đăng nhập
│       │   │   ├── InvalidBidException.java       # [Bước 0 - ĐÃ CÓ TRÊN DEV] Ngoại lệ đặt giá
│       │   │   └── AuctionClosedException.java    # [Bước 0 - ĐÃ CÓ TRÊN DEV] Ngoại lệ phiên đóng
│       │   ├── network/                           # Hạ tầng Socket & Giao thức JSON (Mảng 3 - Sang)
│       │   │   ├── protocol/
│       │   │   │   ├── ActionType.java            # PING, LOGIN, REGISTER, PLACE_BID...
│       │   │   │   ├── ResponseStatus.java        # OK, ERROR, UNAUTHORIZED, NOT_FOUND
│       │   │   │   ├── Request.java
│       │   │   │   ├── Response.java
│       │   │   │   └── MessageSerializer.java     # Chuyển đổi JSON bằng Gson (serializeNulls)
│       │   │   └── server/
│       │   │       ├── AuctionServer.java         # ServerSocket lắng nghe cổng 8888
│       │   │       └── ClientHandler.java         # Tiếp nhận kết nối, xử lý PING -> PONG
│       │   ├── service/
│       │   │   └── UserService.java               # [Mảng 4 - Minh] Dịch vụ login/register ném Exception
│       │   └── storage/
│       │       └── JsonStorage.java               # [Mảng 4 - Minh] Tiện ích đọc/ghi users.json bằng Gson
│       └── test/java/com/auction/server/
│           ├── domain/model/auction/
│           │   ├── AuctionTest.java               # [Mảng 5 - Quân] Unit test assertThrows
│           │   └── AuctionConcurrencyTest.java    # [Mảng 5 - Quân] Test ReentrantLock 10-20 threads
│           ├── network/
│           │   ├── EchoClientTest.java            # [Mảng 3 - Sang] Test kết nối Socket PING/PONG
│           │   └── protocol/
│           │       └── MessageSerializerTest.java # [Mảng 3 - Sang] Test serialize/deserialize JSON
│           └── service/
│               └── UserServiceTest.java           # [Mảng 4 - Minh] Test assertThrows AuthenticationException
│
└── auction-client/
    ├── pom.xml                                    # Phụ thuộc vào module auction-server
    └── src/
        └── main/
            ├── java/com/auction/client/
            │   ├── Launcher.java                  # Điểm khởi động JavaFX không kế thừa Application
            │   ├── ClientApp.java                 # Kế thừa Application nạp FXML ban đầu
            │   ├── controller/
            │   │   ├── LoginController.java       # [Mảng 1 - Hùng] Bắt AuthenticationException hiện chữ đỏ
            │   │   ├── RegisterController.java    # [Mảng 1 - Hùng]
            │   │   └── BiddingRoomController.java # [Mảng 2 - Tuấn] Implements Observer, Popup Alert
            │   └── util/
            │       └── SceneManager.java          # [Mảng 1 - Hùng] Quản lý chuyển cảnh FXML
            └── resources/
                ├── css/
                │   └── theme.css                  # [Mảng 1 - Hùng] Dark Theme CSS hiện đại
                └── fxml/
                    ├── LoginView.fxml             # [Mảng 1 - Hùng] Form đăng nhập
                    ├── RegisterView.fxml          # [Mảng 1 - Hùng] Form đăng ký
                    └── BiddingRoomView.fxml       # [Mảng 2 - Tuấn / Bước 0] Phòng đấu giá trực tiếp
```

---

## 🔨 PHẦN 4 — ĐẶC TẢ NHIỆM VỤ 5 TRỤC ĐỘC LẬP & PHÂN CÔNG THÀNH VIÊN

---

### 👤 1. ĐỖ MẠNH HÙNG — Mảng 1: Client GUI & Form Flow
* **Nhánh làm việc:** `feature/w7-hung/gui-base` (hoặc `feature/w8-hung/mvc-form-views`)
* **Vai trò kiến trúc:** Chịu trách nhiệm toàn bộ tầng giao diện tương tác (Presentation Layer) của hệ thống Client theo mô hình MVC. Xây dựng tài liệu chuẩn hóa JavaFX cho toàn nhóm, thiết kế các Form xác thực và quản trị bằng CSS Dark Theme, quản lý cơ chế chuyển màn hình qua Stage tập trung, và xử lý thông báo ngoại lệ thân thiện trực tiếp trên Form đăng nhập.

#### 📝 Mô tả bài tập:
Bạn thiết lập nền móng giao diện JavaFX cho toàn bộ dự án Client. Bạn tạo bộ tài liệu hướng dẫn nhóm cách sử dụng SceneBuilder và kết nối FXML với Controller. Bạn thiết kế bộ CSS Dark Theme dùng chung mang phong cách hiện đại. Bạn xây dựng Form Đăng nhập và Form Đăng ký, hiện thực lớp `SceneManager` theo mẫu Singleton để điều hướng giữa các màn hình. Đặc biệt, bạn áp dụng kiến thức Tuần 8 về xử lý ngoại lệ: khi người dùng nhập sai thông tin đăng nhập, hệ thống sẽ bắt `AuthenticationException` và hiển thị thông báo lỗi màu đỏ ngay dưới ô nhập liệu mà không làm gián đoạn trải nghiệm người dùng.

#### 📋 Nhiệm vụ chi tiết (Đặc tả kỹ thuật & Luồng xử lý):
1. **Biên soạn tài liệu hướng dẫn thực hành JavaFX (`docs/javafx-guide.md`):**
   * Soạn thảo tài liệu bằng tiếng Việt chi tiết, rõ ràng làm kim chỉ nam cho các thành viên làm việc với JavaFX trong dự án.
   * Cấu trúc tài liệu gồm 5 chuyên đề bắt buộc:
     * *Chuyên đề 1:* Hướng dẫn cài đặt SceneBuilder và tích hợp vào các IDE (IntelliJ IDEA, VS Code).
     * *Chuyên đề 2:* Quy chuẩn đặt tên định danh `fx:id` cho các Control giao diện (tiền tố chuẩn hóa: `txt` cho ô nhập văn bản, `btn` cho nút bấm, `lbl` cho nhãn văn bản, `tbl` cho bảng dữ liệu, `cb` cho hộp chọn thả xuống).
     * *Chuyên đề 3:* Cách liên kết View FXML với Controller thông qua thuộc tính `fx:controller` và gán sự kiện click chuột qua thuộc tính `onAction="#handleMethodName"`.
     * *Chuyên đề 4:* Hướng dẫn nhúng file CSS vào FXML bằng thẻ `<stylesheets>` để phân tách triệt để giữa cấu trúc giao diện và định dạng phong cách.
     * *Chuyên đề 5:* Cách nạp và khởi chạy FXML trong mã nguồn Java bằng `FXMLLoader.load()`, xử lý ngoại lệ `IOException` và nạp dữ liệu vào Scene.
2. **Xây dựng bộ CSS Theme Dark Mode dùng chung (`resources/css/theme.css`):**
   * Định hình bảng màu giao diện tối hiện đại (Modern Dark Palette):
     * Màu nền chính: Tone xám đen công nghệ (`#1e1e2f`, `#27293d`).
     * Màu chữ: Trắng sáng (`#ffffff`) cho tiêu đề, xám sáng (`#a0a0b0`) cho nhãn phụ.
     * Màu nhấn (*Accent Color*): Tím ánh kim (`#7928ca`) hoặc Xanh Cyan (`#00f2fe`) cho nút bấm chính và viền trạng thái kích hoạt.
     * Màu cảnh báo lỗi: Đỏ tươi (`#ff4d4f`) phục vụ hiển thị ngoại lệ.
   * Định nghĩa các Selector CSS quy chuẩn:
     * Lớp `.root`: Cấu hình font chữ hệ thống (`Segoe UI`, `sans-serif`), cỡ chữ cơ sở 14px, thiết lập nền tối cho toàn bộ cửa sổ.
     * Lớp `.button`: Bo góc 8px (`-fx-background-radius: 8`), tạo hiệu ứng chuyển màu khi di chuột (`:hover`) và khi nhấn (`:pressed`), đổi con trỏ chuột sang hình bàn tay (`-fx-cursor: hand`).
     * Lớp `.text-field` và `.password-field`: Nền xám tối, bo góc 6px, viền mỏng mờ, tự động phát sáng viền theo màu nhấn khi trường được trỏ chuột (`:focused`).
     * Lớp nhãn lỗi `.error-label`: Cấu hình màu chữ đỏ (`#ff4d4f`), font in nghiêng, cỡ chữ 12px, đảm bảo thông báo nổi bật trên nền tối.
3. **Hiện thực hóa cơ chế quản lý màn hình `SceneManager.java` & nạp FXML trong `ClientApp.java`:**
   * Lớp `SceneManager` (package `com.auction.client.util`):
     * Triển khai mẫu thiết kế **Singleton**: Thuộc tính tĩnh lưu trữ instance duy nhất, constructor khai báo `private`, phương thức công khai `public static SceneManager getInstance()`.
     * Lưu trữ tham chiếu biến `private Stage primaryStage`.
     * Phương thức `public void setPrimaryStage(Stage stage)`: Lưu trữ cửa sổ chính khi ứng dụng JavaFX khởi động.
     * Phương thức `public void switchScene(String fxmlPath)`: Nhận đường dẫn file FXML tuyệt đối tính từ thư mục resources (bắt buộc có tiền tố `/fxml/`, ví dụ `/fxml/BiddingRoomView.fxml`), nạp cây giao diện qua `FXMLLoader`, khởi tạo `Scene` mới với kích thước phù hợp, gắn vào `primaryStage` và gọi phương thức hiển thị. Toàn bộ quá trình nạp được bọc trong khối kiểm soát ngoại lệ `IOException`.
   * Lớp `ClientApp.java` (package `com.auction.client`):
     * Kế thừa `javafx.application.Application`.
     * Trong phương thức `start(Stage stage)`: Gán `stage` vào `SceneManager`, nạp màn hình khởi đầu `/fxml/LoginView.fxml`, cấu hình tiêu đề cửa sổ hệ thống ("Online Auction System - Nhóm 1"), thiết lập kích thước chuẩn 800x600 pixel, và kích hoạt hiển thị cửa sổ.
4. **Thiết kế Form Đăng nhập & Xử lý Ngoại lệ trong `LoginController.java` (Độc lập 100%):**
   * Khai báo các liên kết `@FXML`: Ô nhập tên đăng nhập `txtUsername`, ô nhập mật khẩu `txtPassword`, nhãn hiển thị thông báo lỗi `lblError`, nút đăng nhập `btnLogin`, nút chuyển đăng ký `btnGoRegister`.
   * Đặc tả luồng xử lý phương thức `handleLogin()`:
     * *Bước 1 (Làm sạch dữ liệu):* Đọc giá trị người dùng nhập từ `txtUsername` và loại bỏ khoảng trắng thừa hai đầu; đọc mật khẩu từ `txtPassword`.
     * *Bước 2 (Kiểm tra rỗng tại chỗ):* Nếu người dùng để trống một trong hai ô, hiển thị ngay thông báo yêu cầu nhập đầy đủ thông tin lên `lblError` với màu đỏ.
     * *Bước 3 (Xác thực đăng nhập nội bộ & Ném ngoại lệ):* Để hoàn toàn độc lập với tầng Server và Service của Minh, Hùng thực hiện kiểm tra xác thực bằng dữ liệu mẫu (Mock Credentials) được định nghĩa sẵn trong Controller (ví dụ: tài khoản hợp lệ `"admin"` với mật khẩu `"123"`). Nếu thông tin nhập vào không trùng khớp, Controller chủ động khởi tạo và ném ngoại lệ: `throw new AuthenticationException("Sai tên đăng nhập hoặc mật khẩu!")`.
     * *Bước 4 (Bắt ngoại lệ & Hiển thị chữ đỏ):* Khối `try-catch` bắt trực tiếp ngoại lệ `AuthenticationException` (được kế thừa từ lớp cha `AuctionException` đã có sẵn từ Bước 0 trên nhánh `dev`). Khi bắt được lỗi, gán nội dung thông điệp `e.getMessage()` lên nhãn `lblError` và áp dụng style chữ màu đỏ.
     * *Bước 5 (Đăng nhập thành công):* Nếu thông tin khớp, xóa sạch thông báo lỗi cũ trên `lblError` và gọi `SceneManager.getInstance().switchScene("/fxml/BiddingRoomView.fxml")` để chuyển sang phòng đấu giá. Nhờ Sang đã tạo sẵn file mồi `BiddingRoomView.fxml` ở Bước 0, quá trình chuyển màn hình diễn ra mượt mà và an toàn tuyệt đối.
   * Phương thức `handleGoRegister()`: Gọi `SceneManager` chuyển cửa sổ sang `/fxml/RegisterView.fxml`.
5. **Thiết kế Form Đăng ký `RegisterView.fxml` & `RegisterController.java`:**
   * Thiết kế giao diện đầy đủ các trường nhập liệu: Tên tài khoản, Mật khẩu, Xác nhận mật khẩu, Địa chỉ email, và ComboBox lựa chọn vai trò người dùng (`BIDDER` hoặc `SELLER`).
   * Kiểm tra tính hợp lệ cơ bản của form: Mật khẩu xác nhận phải trùng khớp với mật khẩu đã nhập, định dạng email phải chứa ký tự `@` và tên miền hợp lệ. Khi đăng ký thành công (giả lập), hiển thị thông báo thành công và cung cấp nút bấm điều hướng quay trở lại màn hình Đăng nhập.

#### ⚡ Cơ chế đảm bảo độc lập tuyệt đối (Không chờ đợi ai):
* Đã có sẵn file FXML mồi `BiddingRoomView.fxml` từ Bước 0 trên nhánh `dev`, chuyển màn hình không lo lỗi file thiếu.
* Đã có sẵn class ngoại lệ `AuthenticationException` từ Bước 0 trên nhánh `dev`, import vào Controller bắt lỗi bình thường.
* Tự xác thực đăng nhập bằng Mock Data nội bộ, không cần chờ `UserService` của Minh hay Socket của Sang.

#### ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch cũ:
* Tạm hoãn thiết kế các dialog phụ nâng cao (`AddItemDialog.fxml`, `SellerDashboard.fxml`) và các bộ lọc tìm kiếm sản phẩm.
* **Tuyệt đối không kết nối Socket mạng ngầm vào các Controller ở tuần này** (chạy mock in-memory để đảm bảo độ ổn định của luồng giao diện UI).

#### ✅ Tiêu chí nghiệm thu đầu ra:
```bash
cd auction-client
mvn javafx:run
# PASS nếu:
# 1. Cửa sổ mở ra hiển thị form LoginView.fxml với phong cách Dark Theme sắc nét, căn chỉnh chuẩn.
# 2. Gõ sai tài khoản/mật khẩu -> Nhãn lblError lập tức hiện dòng chữ đỏ: "Sai tên đăng nhập hoặc mật khẩu!".
# 3. Gõ đúng tài khoản mẫu ("admin" / "123") -> Chuyển màn hình sang phòng đấu giá BiddingRoomView không gặp lỗi runtime.
# 4. Có đầy đủ file tài liệu hướng dẫn docs/javafx-guide.md trong thư mục dự án.
```

---

### 👤 2. LÊ ANH TUẤN — Mảng 2: Realtime UI & Observer Pattern
* **Nhánh làm việc:** `feature/w7-tuan/realtime-bidding-room`
* **Vai trò kiến trúc:** Xây dựng màn hình trung tâm của hệ thống — Phòng đấu giá trực tiếp (`BiddingRoomView.fxml`), đóng vai trò Observer trong mô hình MVC để cập nhật dữ liệu tự động, và xử lý các tình huống lỗi nghiệp vụ đấu giá bằng Popup cảnh báo.

#### 📝 Mô tả bài tập:
Bạn chịu trách nhiệm thiết kế giao diện phòng đấu giá trực tiếp với các thành phần hiển thị chỉ số thời gian thực, bảng lịch sử và đồng hồ đếm ngược. Bạn hiện thực hóa `BiddingRoomController` triển khai interface `AuctionObserver`. Khi có sự kiện đặt giá mới, Controller sẽ tự động cập nhật số liệu hiển thị trên giao diện mà không cần tải lại trang. Khi người dùng thao tác đặt giá sai quy định, hệ thống sẽ bắt các Custom Exceptions nghiệp vụ của Tuần 8 và hiển thị hộp thoại cảnh báo Popup tương ứng.

#### 📋 Nhiệm vụ chi tiết (Đặc tả kỹ thuật & Luồng xử lý):
1. **Sử dụng Interface `AuctionObserver.java` (Đã có sẵn trên `dev` từ Bước 0):**
   * Vị trí: `com.auction.server.domain.model.auction.observer.AuctionObserver`.
   * Gồm 2 phương thức trừu tượng định hình cơ chế giao tiếp Observer:
     * `void onNewBidPlaced(Auction auction, Bid newBid)`: Kích hoạt khi có lượt đặt giá mới thành công trong phiên.
     * `void onAuctionStatusChanged(Auction auction, AuctionStatus newStatus)`: Kích hoạt khi phiên thay đổi trạng thái hoạt động (bắt đầu, kết thúc, tạm dừng).
2. **Thiết kế giao diện phòng đấu giá `BiddingRoomView.fxml`:**
   * Xây dựng giao diện dựa trên theme mặc định của JavaFX (Modena), đảm bảo hoàn toàn độc lập với tiến độ CSS của Hùng.
   * Phân chia bố cục màn hình khoa học thành 3 khối chính:
     * *Khối 1 (Thông tin sản phẩm & Trạng thái):*
       - Tên sản phẩm (`lblItemName`): Cỡ chữ to, in đậm.
       - Mô tả sản phẩm, danh mục hàng hóa, và giá khởi điểm ban đầu.
       - Nhãn trạng thái phiên (`lblStatus`): Hiển thị trạng thái hiện tại (`RUNNING`, `FINISHED`...).
       - Đồng hồ đếm ngược (`lblCountdown`): Sử dụng `javafx.animation.Timeline` chạy chu kỳ lặp 1 giây để đếm lùi thời gian phiên đấu giá; cấu hình tự động đổi màu nhãn sang đỏ khi thời gian còn dưới 60 giây để cảnh báo người dùng.
     * *Khối 2 (Bảng chỉ số trực tiếp & Lịch sử đặt giá):*
       - Mức giá cao nhất hiện tại (`lblCurrentHighestBid`): Vị trí trung tâm, kích thước chữ lớn nhất giao diện.
       - Tên người đang dẫn đầu phiên (`lblLeadingBidder`).
       - Bảng lịch sử đặt giá `TableView<Bid>`: 3 cột dữ liệu gồm Cột Thời gian (`colTime`), Cột Người đặt giá (`colBidder`), và Cột Mức giá (`colAmount`). Cấu hình tự động chèn lượt bid mới lên vị trí trên cùng.
     * *Khối 3 (Bàn điều khiển đặt giá):*
       - Ô nhập số tiền tùy chọn `txtBidAmount` (TextField).
       - Các nút bấm đặt giá nhanh (`btnQuickBid1` cộng 1 bước giá, `btnQuickBid2` cộng 2 bước giá).
       - Nút bấm chính `btnPlaceBid` ("ĐẶT GIÁ").
3. **Hiện thực hóa `BiddingRoomController.java` (Observer Pattern & Xử lý Ngoại lệ bằng Popup Alert):**
   * Khai báo lớp thực thi: `public class BiddingRoomController implements AuctionObserver, Initializable`.
   * Cài đặt 2 phương thức callback của Observer:
     * `onNewBidPlaced(Auction auction, Bid newBid)`: Bắt buộc sử dụng `Platform.runLater(...)` để chuyển việc cập nhật giao diện vào đúng luồng sự kiện chính của JavaFX (JavaFX Application Thread). Cập nhật lại giá trị trên `lblCurrentHighestBid`, gán tên người dẫn đầu lên `lblLeadingBidder`, và thêm đối tượng `newBid` vào danh sách dữ liệu của bảng `TableView`.
     * `onAuctionStatusChanged(Auction auction, AuctionStatus newStatus)`: Cập nhật văn bản trên `lblStatus`; nếu trạng thái chuyển thành `FINISHED`, tự động làm mờ và vô hiệu hóa nút bấm đặt giá để người dùng không thể gửi thêm bid.
   * Xử lý sự kiện bấm nút `btnPlaceBid` với cơ chế bắt ngoại lệ độc lập (Tuần 8):
     * Toàn bộ thao tác kiểm tra và đặt giá được đặt trong khối `try-catch`.
     * *Xử lý lỗi định dạng số:* Chuyển đổi dữ liệu từ `txtBidAmount` sang kiểu số thực `double`. Nếu người dùng nhập chữ hoặc để trống, bắt ngoại lệ `NumberFormatException` và hiển thị Popup `Alert(AlertType.WARNING)` yêu cầu nhập số tiền hợp lệ.
     * *Xử lý ngoại lệ nghiệp vụ Tuần 8:*
       - Nếu số tiền đặt nhỏ hơn hoặc bằng mức giá hiện tại cộng bước giá $\rightarrow$ Ném ngoại lệ `InvalidBidException` kèm thông điệp giải thích rõ ràng.
       - Nếu phiên đấu giá đã kết thúc hoặc bị đóng $\rightarrow$ Ném ngoại lệ `AuctionClosedException`.
     * *Xử lý hiển thị Popup Alert:*
       - Khối `catch (InvalidBidException e)`: Kích hoạt hiển thị hộp thoại `Alert` dạng cảnh báo (WARNING) hiển thị chính xác thông điệp lỗi nghiệp vụ được lưu trong `e.getMessage()`.
       - Khối `catch (AuctionClosedException e)`: Kích hoạt hiển thị hộp thoại `Alert` dạng lỗi nghiêm trọng (ERROR) thông báo phiên đã kết thúc.
     * *Cơ chế kích hoạt Observer độc lập (Không chờ đợi hàm attachObserver của Quân):*
       - Khi dữ liệu đặt giá hoàn toàn hợp lệ, Controller tự khởi tạo một đối tượng `Bid` mẫu với thông tin người dùng hiện tại và mức giá mới, sau đó gọi trực tiếp hàm callback của chính mình: `this.onNewBidPlaced(currentAuction, dummyBid)`.
       - Màn hình lập tức nhảy số giá mới và thêm một dòng lịch sử vào bảng $\rightarrow$ Chứng minh luồng Observer của Controller chạy hoàn hảo 100% độc lập.
4. **Viết Unit Test kiểm thử Observer độc lập (`AuctionObserverTest.java`):**
   * Tạo bài kiểm thử JUnit 5 độc lập trong thư mục test client:
     * Khởi tạo một đối tượng giả lập (Mock Listener) thực thi interface `AuctionObserver`.
     * Kích hoạt gọi hàm `onNewBidPlaced` với đối tượng `Bid` giả định và kiểm tra trạng thái nhận dữ liệu bằng `assertEquals`.
     * Kích hoạt gọi hàm `onAuctionStatusChanged` và kiểm tra trạng thái mới được cập nhật chuẩn xác.

#### ⚡ Cơ chế đảm bảo độc lập tuyệt đối (Không chờ đợi ai):
* Giao diện dùng theme mặc định Modena, không phụ thuộc vào tiến độ file `theme.css` của Hùng.
* Đã có sẵn `AuctionObserver`, `InvalidBidException`, `AuctionClosedException` từ Bước 0 trên `dev`.
* Tự gọi hàm callback nội bộ với dữ liệu mẫu khi bấm nút, không phải đợi hàm `attachObserver` của Quân hay Socket của Sang.

#### ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch cũ:
* Chưa cần viết lớp `AuctionClient` chạy Socket mạng ngầm để nhận push thông báo qua giao thức TCP.

#### ✅ Tiêu chí nghiệm thu đầu ra:
```bash
mvn test -Dtest="AuctionObserverTest"
# PASS nếu:
# 1. Mở màn hình BiddingRoomView hiển thị đầy đủ đồng hồ đếm ngược, bảng lịch sử và các nút điều khiển.
# 2. Nhập số tiền hợp lệ và bấm Đặt giá -> Giá trên màn hình tự động nhảy số tức thì và thêm dòng mới vào TableView.
# 3. Nhập số tiền sai bước giá hoặc nhập chữ -> Bật popup Alert cảnh báo hiển thị đúng thông điệp ngoại lệ.
# 4. Toàn bộ test case trong AuctionObserverTest chạy pass 100%.
```

---

### 👤 3. LÊ ĐỨC THIỆN SANG (Nhóm trưởng) — Mảng 3: Network Socket & DevOps CI
* **Nhánh làm việc:** `feature/w7-sang/network-protocol`
* **Vai trò kiến trúc:** Kiến trúc sư hạ tầng mạng và tích hợp DevOps. Thiết kế chuẩn hóa giao thức gói tin JSON, xây dựng máy chủ Socket đa luồng phục vụ Client, tích hợp bộ kiểm tra quy chuẩn mã nguồn Google Checkstyle vào Maven, và bảo trợ kỹ thuật cho toàn nhóm.

#### 📝 Mô tả bài tập:
Bạn thiết lập hạ tầng giao tiếp mạng hướng dòng qua giao thức TCP Socket cho hệ thống. Bạn chuẩn hóa cấu trúc gói tin JSON phân định rõ ràng giữa `Request` và `Response`. Bạn khởi tạo 6 file khung nền tảng ở Bước 0 trực tiếp trên nhánh `dev`. Bạn tích hợp `maven-checkstyle-plugin` cấu hình Google Java Style vào file `pom.xml` gốc. Bạn hiện thực hóa máy chủ `AuctionServer` và luồng tiếp nhận `ClientHandler` đa luồng, đồng thời hoàn thành bộ kiểm thử tích hợp mạng pass 100%.

#### 📋 Nhiệm vụ chi tiết (Đã hoàn thành 100%):
1. **Khởi tạo và thiết lập 6 file khung mồi trên nhánh `dev` (Bước 0 - ĐÃ XONG):**
   * Trực tiếp tạo và đẩy lên nhánh `dev` 6 file nền tảng: `AuctionObserver.java`, `AuctionException.java`, `InvalidBidException.java`, `AuctionClosedException.java`, `AuthenticationException.java`, và `BiddingRoomView.fxml`.
   * Giúp toàn bộ 4 thành viên kéo code về là có sẵn class để import, triệt tiêu 100% lỗi đỏ IDE.
2. **Tích hợp kiểm tra chuẩn mã nguồn Google Checkstyle vào Maven (`pom.xml` - ĐÃ XONG):**
   * Bổ sung `maven-checkstyle-plugin` (phiên bản 3.3.1) vào phần `<build><plugins>` của `pom.xml` gốc.
   * Liên kết với bộ luật chính thức `google_checks.xml` của Google (thụt lề 2 spaces, ngoặc nhọn cùng dòng, không import dạng `*`, quy ước tên biến chuẩn).
   * Cấu hình tham số `<failsOnError>false</failsOnError>` và `<consoleOutput>true</consoleOutput>` để khi chạy lệnh `mvn checkstyle:check`, Maven sẽ quét toàn bộ mã nguồn, in báo cáo các dòng vi phạm ra màn hình terminal một cách rõ ràng mà không làm gián đoạn tiến trình build.
3. **Chuẩn hóa Giao thức Gói tin Mạng JSON (`com.auction.server.network.protocol` - ĐÃ XONG):**
   * Định nghĩa Enum `ActionType`: Chứa các định danh hành vi nghiệp vụ cơ bản gồm `PING`, `LOGIN`, `REGISTER`, và `PLACE_BID`.
   * Định nghĩa Enum `ResponseStatus`: Chứa các mã trạng thái phản hồi gồm `OK`, `ERROR`, `UNAUTHORIZED`, và `NOT_FOUND`.
   * Thiết kế lớp `Request.java`: Chứa các trường thuộc tính `private ActionType action`, `private Map<String, Object> data`, và `private String token` kèm đầy đủ constructor, getter và setter.
   * Thiết kế lớp `Response.java`: Chứa các trường thuộc tính `private ResponseStatus status`, `private String message`, và `private Object data`, cùng các phương thức tĩnh tiện ích `Response.ok(Object data)` và `Response.error(String message)`.
   * Thiết kế lớp `MessageSerializer.java`:
     * Sử dụng thư viện Gson để chuyển đổi hai chiều giữa đối tượng Java và chuỗi JSON.
     * Cấu hình đặc biệt: Sử dụng `new GsonBuilder().serializeNulls().create()` để đảm bảo khi thuộc tính `data` mang giá trị `null` thì chuỗi JSON sinh ra vẫn giữ nguyên cặp khóa-giá trị `"data": null`, không làm mất trường dữ liệu khi Client giải mã.
     * Phương thức `serialize(Object obj)`: Chuyển đổi đối tượng sang chuỗi JSON đơn dòng.
     * Phương thức `deserialize(String json)`: Giải mã chuỗi JSON thành đối tượng `Request`.
     * Phương thức `deserializeResponse(String json)`: Giải mã chuỗi JSON thành đối tượng `Response`.
4. **Hiện thực hóa Socket Server Đa Luồng (`com.auction.server.network.server` - ĐÃ XONG):**
   * Lớp `ClientHandler.java` (triển khai `Runnable`):
     * Nhận đối tượng `Socket` kết nối từ Client, khởi tạo luồng đọc văn bản `BufferedReader` và luồng gửi dữ liệu `PrintWriter(socket.getOutputStream(), true)` với chế độ `autoFlush = true` để đẩy gói tin đi tức thì.
     * Vòng lặp `while ((line = in.readLine()) != null)`: Tiếp nhận từng gói tin JSON nằm trên một dòng đơn kết thúc bằng ký tự `\n`.
     * Giải mã chuỗi văn bản thành đối tượng `Request`. Nếu gói tin có hành động `PING`, Handler lập tức phản hồi một gói tin `Response` có trạng thái `OK`, thông điệp `"Success"` và dữ liệu `"PONG"`.
     * Xử lý ngắt kết nối an toàn (*Graceful Disconnect*) trong khối `finally`: Tự động đóng các luồng I/O và Socket khi Client ngắt kết nối hoặc tắt ứng dụng đột ngột.
   * Lớp `AuctionServer.java`:
     * Quản lý vòng đời Server lắng nghe trên cổng `8888`.
     * Trong vòng lặp `while (running)`: Phương thức `serverSocket.accept()` chờ đợi kết nối. Mỗi khi có một Client mới kết nối đến, Server lập tức khởi tạo một luồng thực thi độc lập bằng `new Thread(new ClientHandler(clientSocket)).start()`, tuân thủ đúng chuẩn kỹ thuật giảng dạy của Giảng viên.
   * Lớp `ServerApp.java`: Điểm vào chính của Server (`main`), in log chào mừng, thông tin phiên bản và gọi khởi chạy `AuctionServer`.
5. **Bộ kiểm thử Tích hợp Mạng & Serialization (Pass 100%):**
   * `MessageSerializerTest.java`: Kiểm thử chuyển đổi gói tin Request sang JSON và ngược lại; kiểm thử chuyển đổi gói tin Response; kiểm tra tính toàn vẹn của trường dữ liệu `null`.
   * `EchoClientTest.java`: Kiểm thử tích hợp kết nối Socket thực tế: Khởi chạy Server trên cổng 8888 trong một luồng nền, mở kết nối Socket từ phía test client, gửi gói tin JSON PING và assert kiểm tra nhận về đúng gói tin JSON PONG trong thời gian quy định.

#### ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch cũ:
* Bỏ `CachedThreadPool`, sử dụng cơ chế `new Thread(...)` cơ bản kinh điển để mã nguồn ngắn gọn, dễ hiểu và dễ giải trình khi vấn đáp.
* Tạm hoãn cơ chế Broadcast push dữ liệu đồng thời tới tất cả Client sang Tuần 10.

#### ✅ Tiêu chí nghiệm thu đầu ra:
```bash
mvn test -Dtest="MessageSerializerTest,EchoClientTest"
mvn checkstyle:check
# PASS nếu:
# 1. Bộ test MessageSerializerTest và EchoClientTest chạy pass xanh 100%.
# 2. Lệnh mvn checkstyle:check chạy thành công và xuất báo cáo kiểm tra mã nguồn theo chuẩn Google Style.
# 3. Chạy ServerApp trên cổng 8888 in log hoạt động và phản hồi lệnh PING/PONG chuẩn xác.
```

---

### 👤 4. HOÀNG THẾ CÔNG MINH — Mảng 4: Storage, Auth & Exception Core
* **Nhánh làm việc:** `feature/w7-minh/persistence-auth` (hoặc `feature/w8-minh/custom-exceptions-auth`)
* **Vai trò kiến trúc:** Quản trị tầng lưu trữ dữ liệu người dùng (Persistence Layer), xây dựng dịch vụ nghiệp vụ đăng ký và đăng nhập, sở hữu cụm ngoại lệ người dùng, ném ngoại lệ nghiệp vụ và viết bộ kiểm thử bắt ngoại lệ bằng JUnit 5 `assertThrows`.

#### 📝 Mô tả bài tập:
Bạn chịu trách nhiệm lưu trữ và xác thực thông tin người dùng trong hệ thống. Bạn viết tài liệu quy chuẩn lập trình `STYLE_GUIDE.md` cho toàn nhóm. Bạn tạo class tiện ích `JsonStorage` đọc và ghi danh sách tài khoản ra file `users.json` bằng thư viện Gson để bảo toàn dữ liệu khi tắt ứng dụng. Bạn cập nhật `UserService` ném Custom Exceptions (`AuthenticationException`, `AuctionException`) khi có sai sót nghiệp vụ, và viết các bài kiểm thử tự động sử dụng `assertThrows` của JUnit 5 để chứng minh code bắt lỗi chuẩn xác.

#### 📋 Nhiệm vụ chi tiết (Đặc tả kỹ thuật & Luồng xử lý):
1. **Sở hữu Cụm Ngoại lệ Xác thực người dùng (Đã có sẵn từ Bước 0 trên `dev`):**
   * Kéo code nhánh `dev` về là có sẵn 2 class trong package `com.auction.server.exception` (không cần tạo mới):
     * `AuctionException.java` (`extends RuntimeException`): Lớp ngoại lệ cha gốc của toàn bộ hệ thống.
     * `AuthenticationException.java` (`extends AuctionException`): Lớp ngoại lệ con chuyên trách ném khi xảy ra lỗi xác thực tài khoản hoặc sai mật khẩu.
2. **Biên soạn tài liệu quy chuẩn lập trình (`docs/STYLE_GUIDE.md`):**
   * Xây dựng cẩm nang coding conventions làm tiêu chuẩn cho cả 5 thành viên tuân thủ:
     * Quy tắc đặt tên: Tên Class viết theo PascalCase; tên phương thức và biến viết theo camelCase; tên hằng số viết theo UPPER_SNAKE_CASE.
     * Quy tắc kiểm thử theo mô hình 3 bước **AAA (Arrange – Act – Assert)**: Khởi tạo dữ liệu giả lập (Arrange) $\rightarrow$ Gọi phương thức cần kiểm thử (Act) $\rightarrow$ Khẳng định kết quả trả về hoặc bắt ngoại lệ (Assert).
     * Quy chuẩn Exception: Mọi thông điệp lỗi trong constructor của Exception bắt buộc phải dùng tiếng Việt rõ nghĩa, mô tả chính xác nguyên nhân lỗi để phục vụ việc hiển thị trên giao diện người dùng.
3. **Hiện thực hóa tiện ích lưu trữ dữ liệu JSON (`com.auction.server.storage.JsonStorage`):**
   * Tạo class `JsonStorage.java` trong package storage của module `auction-server`.
   * Sử dụng thư viện Google Gson để đọc và ghi dữ liệu:
     * *Phương thức lưu dữ liệu `saveUsers(List<User> users)`:* Nhận danh sách người dùng trong hệ thống, cấu hình Gson với chế độ Pretty Printing để định dạng file dễ đọc, và ghi đè nội dung vào file `auction-server/data/users.json`. Tự động kiểm tra và tạo thư mục `data/` nếu chưa tồn tại trên ổ đĩa.
     * *Phương thức đọc dữ liệu `loadUsers()`:* Kiểm tra sự tồn tại của file `auction-server/data/users.json`. Nếu file chưa tồn tại hoặc rỗng, trả về một danh sách `ArrayList<User>` rỗng; nếu có dữ liệu, sử dụng `TypeToken<List<User>>` của Gson để giải mã nội dung JSON thành danh sách các đối tượng `User` hoàn chỉnh.
4. **Cập nhật Dịch vụ Người dùng ném Ngoại lệ trong `UserService.java` (Tuần 8):**
   * Lớp `UserService` quản lý danh sách `users` trên bộ nhớ và đồng bộ với `JsonStorage`.
   * *Phương thức đăng ký `public void register(User newUser)`:*
     * Duyệt qua danh sách người dùng hiện tại để kiểm tra xem `username` đã tồn tại hay chưa.
     * Nếu bị trùng username $\rightarrow$ Chủ động ném ngoại lệ: `throw new AuctionException("Tên đăng nhập đã tồn tại trong hệ thống!")`.
     * Nếu hợp lệ $\rightarrow$ Thêm `newUser` vào danh sách và gọi `jsonStorage.saveUsers(users)` để lưu dữ liệu xuống file.
   * *Phương thức đăng nhập `public User login(String username, String password)`:*
     * Tìm kiếm đối tượng người dùng theo `username`.
     * Nếu không tìm thấy người dùng HOẶC mật khẩu không trùng khớp $\rightarrow$ Chủ động ném ngoại lệ: `throw new AuthenticationException("Sai tên đăng nhập hoặc mật khẩu!")`.
     * Nếu khớp thông tin $\rightarrow$ Trả về đối tượng `User` đăng nhập thành công.
5. **Viết Unit Test kiểm thử ném ngoại lệ bằng JUnit 5 (`UserServiceTest.java` - Tuần 8):**
   * Tạo file `UserServiceTest.java` trong thư mục `src/test/java/com/auction/server/service/`.
   * Thiết lập các kịch bản kiểm thử sử dụng `assertThrows`:
     * *Kịch bản 1 (Đăng nhập sai mật khẩu):* Gọi `userService.login("user_dung", "mat_khau_sai")` bên trong biểu thức Lambda của `assertThrows(AuthenticationException.class, ...)`. Nhận đối tượng ngoại lệ trả về và assert kiểm tra chuỗi `e.getMessage()` chứa đúng nội dung `"Sai tên đăng nhập hoặc mật khẩu!"`.
     * *Kịch bản 2 (Đăng nhập tài khoản không tồn tại):* Gọi `userService.login("khong_ton_tai", "123")` và kiểm tra ném `AuthenticationException`.
     * *Kịch bản 3 (Đăng ký trùng tên tài khoản):* Khởi tạo trước một tài khoản mẫu `"user1"`. Sau đó gọi `userService.register(...)` với một tài khoản khác cũng có tên `"user1"` bên trong `assertThrows(AuctionException.class, ...)`.
     * *Kịch bản 4 (Đăng nhập thành công):* Kiểm tra đăng nhập với tài khoản hợp lệ trả về đối tượng `User` khác `null` và đúng thông tin username bằng `assertEquals`.

#### ⚡ Cơ chế đảm bảo độc lập tuyệt đối (Không chờ đợi ai):
* Đã có sẵn các class ngoại lệ `AuctionException`, `AuthenticationException` từ Bước 0 trên `dev`.
* Tự kiểm thử toàn diện `UserService` và `JsonStorage` thông qua các bài test JUnit 5 độc lập mà không cần phải chạy giao diện JavaFX hay khởi động Socket Server.

#### ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch cũ:
* Tạm hoãn mô hình Generic DAO phân tầng phức tạp (`GenericDAO<T>`, `UserDAOImpl`); dùng trực tiếp `JsonStorage` để mã nguồn tinh gọn, dễ quản lý.
* Tạm hoãn thuật toán mã hóa SHA-256 + Salt (nằm ngoài phạm vi barem chấm điểm Tuần 9).

#### ✅ Tiêu chí nghiệm thu đầu ra:
```bash
mvn test -Dtest="UserServiceTest"
# PASS nếu:
# 1. Toàn bộ các test cases trong UserServiceTest chạy pass xanh 100%.
# 2. Các bài test assertThrows bắt đúng AuthenticationException và AuctionException kèm đúng nội dung thông điệp.
# 3. File auction-server/data/users.json được tự động sinh ra và lưu trữ dữ liệu người dùng thành công khi chạy test.
# 4. Có file tài liệu docs/STYLE_GUIDE.md quy chuẩn trong thư mục dự án.
```

---

### 👤 5. TRẦN MẠNH QUÂN — Mảng 5: Concurrency Engine & Business Core
* **Nhánh làm việc:** `feature/w7-quan/concurrency-engine`
* **Vai trò kiến trúc:** Quản trị động cơ nghiệp vụ trung tâm của phiên đấu giá, bảo vệ tài nguyên chia sẻ bằng cơ chế Fair Lock (`ReentrantLock(true)`), refactor hàm đặt giá sang ném Custom Exceptions thay cho giá trị boolean, và cập nhật bộ kiểm thử JUnit 5.

#### 📝 Mô tả bài tập:
Bạn phụ trách bài toán kỹ thuật trọng tâm của hệ thống: xử lý tranh chấp khi nhiều người dùng cùng đặt giá đồng thời tại một thời điểm. Bạn nâng cấp hàm `Auction.placeBid()` bằng khóa công bằng `ReentrantLock(true)`. Bạn thực hiện yêu cầu cốt lõi của Tuần 8: thay thế toàn bộ các lệnh `return false` cũ trong hàm đặt giá bằng việc ném Custom Exceptions (`InvalidBidException`, `AuctionClosedException`). Đồng thời, bạn cập nhật các bài test kiểm thử ném ngoại lệ bằng `assertThrows` trong file `AuctionTest.java` và viết bài test đa luồng `AuctionConcurrencyTest.java`.

#### 📋 Nhiệm vụ chi tiết (Đặc tả kỹ thuật & Luồng xử lý):
1. **Sở hữu Cụm Ngoại lệ Đấu giá (Đã có sẵn từ Bước 0 trên `dev`):**
   * Kéo code nhánh `dev` về là có sẵn 2 class trong package `com.auction.server.exception` (không cần tạo mới):
     * `InvalidBidException.java` (`extends AuctionException`): Ném khi mức giá đặt không hợp lệ (thấp hơn giá sàn, không đủ bước giá) hoặc số dư tài khoản không đủ.
     * `AuctionClosedException.java` (`extends AuctionException`): Ném khi cố tình đặt giá vào một phiên không ở trạng thái mở hoạt động (`RUNNING`).
2. **Nâng cấp An toàn Đa luồng với `ReentrantLock` trong `Auction.java`:**
   * Trong lớp `Auction.java`, khai báo biến khóa công bằng:
     * `private final ReentrantLock lock = new ReentrantLock(true);` (Khởi tạo Fair Lock đảm bảo luồng nào đến trước được phục vụ trước theo nguyên tắc FIFO, loại bỏ hoàn toàn nguy cơ chen ngang khi tranh chấp).
     * Khai báo danh sách Observer an toàn đa luồng: `private final List<AuctionObserver> observers = new CopyOnWriteArrayList<>();`.
     * Bổ sung các phương thức quản lý Observer: `attachObserver(AuctionObserver o)` và `detachObserver(AuctionObserver o)`.
     * Phương thức `notifyObservers(Bid bid)`: Duyệt danh sách observers và gọi callback `o.onNewBidPlaced(this, bid)`.
3. **Refactor hàm `placeBid()` ném Custom Exceptions (Tuần 8):**
   * Cập nhật phương thức `public boolean placeBid(User bidder, double amount)`:
   * Toàn bộ quá trình kiểm tra và cập nhật dữ liệu bắt buộc phải nằm trọn trong khối `lock.lock()` và giải phóng khóa trong khối `finally { lock.unlock(); }` để đảm bảo không bao giờ bị nghẽn khóa (*Deadlock*).
   * **Thay thế triệt để các lệnh `return false` cũ bằng việc ném Custom Exceptions:**
     * *Quy tắc kiểm tra 1 (Trạng thái phiên):* Kiểm tra nếu `this.status != AuctionStatus.RUNNING` $\rightarrow$ Chủ động ném ngoại lệ: `throw new AuctionClosedException("Phiên đấu giá đang đóng hoặc đã kết thúc!")`.
     * *Quy tắc kiểm tra 2 (Bước giá tối thiểu):* Xác định mức giá tối thiểu hợp lệ. Nếu phiên chưa có ai đặt giá (`currentHighestBid == 0`), giá tối thiểu bằng `startingPrice`; nếu đã có người đặt, giá tối thiểu bằng `currentHighestBid + minimumStep`. Nếu `amount < minValidPrice` $\rightarrow$ Chủ động ném ngoại lệ: `throw new InvalidBidException("Giá đặt không hợp lệ, phải tăng tối thiểu theo bước giá!")`.
     * *Quy tắc kiểm tra 3 (Số dư ví người đặt giá):* Kiểm tra số dư tài khoản `bidder.getBalance()`. Nếu `bidder.getBalance() < amount` $\rightarrow$ Chủ động ném ngoại lệ: `throw new InvalidBidException("Số dư trong tài khoản không đủ để đặt giá!")`.
     * *Quy tắc cập nhật thành công:* Nếu vượt qua tất cả các điều kiện kiểm tra trên $\rightarrow$ Cập nhật `currentHighestBid = amount`, cập nhật `currentWinner = bidder`, tạo đối tượng `Bid` mới thêm vào danh sách `bidHistory`, gọi hàm `notifyObservers(newBid)` phát tín hiệu, và trả về `true`.
4. **Cập nhật Unit Test kiểm thử ngoại lệ trong `AuctionTest.java` (Tuần 8):**
   * File `AuctionTest.java` đã có sẵn từ Tuần 1. Quân mở file này trên nhánh cá nhân của mình và tiến hành sửa/thêm các hàm `@Test`:
     * *Cập nhật các test cũ:* Những bài test cũ từng assert `assertFalse(auction.placeBid(...))` khi đặt giá sai thì nay phải chuyển sang dùng `assertThrows(InvalidBidException.class, () -> auction.placeBid(...))` để tránh việc bài test bị gãy do ngoại lệ chưa được xử lý.
     * *Viết test mới cho phiên đóng:* Thiết lập trạng thái `auction.setStatus(AuctionStatus.FINISHED)`, sau đó gọi `assertThrows(AuctionClosedException.class, () -> auction.placeBid(bidder, 200.0))` và assert thông điệp lỗi.
     * *Viết test mới cho số dư ví không đủ:* Khởi tạo một bidder có số dư tài khoản nhỏ hơn mức giá muốn đặt, sau đó kiểm tra việc ném `InvalidBidException`.
5. **Bộ kiểm thử Concurrency đa luồng cơ bản (`AuctionConcurrencyTest.java`):**
   * Xây dựng kịch bản kiểm thử đa luồng: Khởi tạo 1 phiên đấu giá và sử dụng `ExecutorService` cùng `CountDownLatch` để kích hoạt đồng thời **10 đến 20 luồng (Threads)** cùng gọi hàm `auction.placeBid()` trong cùng một tích tắc.
   * Assert kiểm tra kết quả: Khóa `ReentrantLock` bảo vệ dữ liệu an toàn 100%, không xảy ra ngoại lệ tranh chấp luồng, danh sách `bidHistory` lưu trữ nhất quán các lượt đặt giá thành công, và giá trị `currentHighestBid` cuối cùng phản ánh chính xác mức giá hợp lệ cao nhất.

#### ⚡ Cơ chế đảm bảo độc lập tuyệt đối (Không chờ đợi ai):
* Đã có sẵn `InvalidBidException` và `AuctionClosedException` từ Bước 0 trên `dev`.
* Quân chỉ làm việc trực tiếp trên file `Auction.java` và các file test của `Auction`, không chạm vào file của Minh, Hùng hay Sang, đảm bảo khi merge không hề bị conflict.

#### ⏸️ Cắt giảm / Hoãn lại so với Kế hoạch cũ:
* Tạm hoãn bộ lập lịch chạy ngầm `AuctionScheduler` (`ScheduledExecutorService` đếm ngược tự đóng phiên) sang Tuần 11–12.
* Giới hạn quy mô test đa luồng ở mức 10–20 threads cơ bản, không stress-test 50 threads quá tải.

#### ✅ Tiêu chí nghiệm thu đầu ra:
```bash
mvn test -Dtest="AuctionConcurrencyTest,AuctionTest"
# PASS nếu:
# 1. Toàn bộ các test cũ và test mới trong AuctionTest chạy pass xanh 100%.
# 2. Các bài test assertThrows bắt đúng InvalidBidException và AuctionClosedException.
# 3. Test đa luồng AuctionConcurrencyTest chạy mượt mà, không xảy ra hiện tượng race condition hay sai lệch dữ liệu.
```

---

## 🔄 PHẦN 5 — MA TRẬN PHÂN CÔNG REVIEW CHÉO PULL REQUEST

Mỗi Pull Request bắt buộc phải có ít nhất **2 thành viên khác** review và phê duyệt (*Approve*) trước khi merge vào nhánh `dev`:

| Nhánh Pull Request | Tác giả | Reviewer 1 (Bắt buộc) | Reviewer 2 (Bắt buộc) |
| :--- | :---: | :---: | :---: |
| `feature/w7-hung/gui-base` | **Hùng** | Tuấn | Minh |
| `feature/w7-tuan/realtime-bidding-room` | **Tuấn** | Quân | Hùng |
| `feature/w7-sang/network-protocol` | **Sang** | Minh | Quân |
| `feature/w7-minh/persistence-auth` | **Minh** | Sang | Hùng |
| `feature/w7-quan/concurrency-engine` | **Quân** | Sang | Tuấn |

### Quy trình 4 bước review một Pull Request:
1. **Pull nhánh của đồng đội về máy cục bộ:**
   ```bash
   git fetch origin
   git checkout feature/w7-[tên]/[tên-chức-năng]
   ```
2. **Kiểm tra kỹ thuật:**
   * Chạy `mvn clean test` $\rightarrow$ Toàn bộ tests của nhánh phải **PASS 100%**.
   * Với nhánh giao diện (Hùng, Tuấn): Mở chạy thử ứng dụng JavaFX kiểm tra tương tác mượt mà, không crash runtime.
3. **Đọc hiểu logic & Đối chiếu quy chuẩn:**
   * Kiểm tra xem code có tuân thủ quy chuẩn không (Checkstyle, Javadoc phương thức public, AAA testing).
4. **Để lại nhận xét trên GitHub:**
   * Chọn **Approve** nếu đạt chuẩn hoặc **Request Changes** nếu còn phát hiện lỗi cần sửa.

---

## 📝 PHẦN 6 — CHECKLIST TỔNG HỢP TRƯỚC KHI NỘP BÀI

### Checklist chung cho cả 5 thành viên:
- [ ] Đã trả lời được 8 câu hỏi vấn đáp miệng trong Phần 1.
- [ ] Làm việc trên đúng nhánh `feature/w7-[tên]/...`, không commit bừa bãi vào `dev` hay `main`.
- [ ] Tuân thủ chuẩn Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `style:`).
- [ ] Khối code của mình tự chạy và tự test thành công (`mvn test` báo xanh 100%).
- [ ] Chạy lệnh `mvn checkstyle:check` không phát sinh lỗi cú pháp.
- [ ] Đã hoàn thành review và Approve ít nhất 2 PR của đồng đội theo Ma trận Review.

### Checklist riêng cho từng mảng:
- [ ] **Hùng (Mảng 1):** `LoginView.fxml` hiển thị Dark Theme; nhập sai tài khoản hiện Label chữ đỏ; nhập đúng chuyển sang `BiddingRoomView`; có tài liệu `docs/javafx-guide.md`.
- [ ] **Tuấn (Mảng 2):** `BiddingRoomView.fxml` có đồng hồ đếm ngược và bảng lịch sử; nút đặt giá tự nhảy số qua Observer callback; nhập giá sai bật Popup `Alert` cảnh báo; `AuctionObserverTest` pass 100%.
- [ ] **Sang (Mảng 3):** Khởi tạo thành công 6 file khung Bước 0 trên `dev`; `ServerApp.java` lắng nghe cổng 8888 phản hồi PING/PONG chuẩn xác; `pom.xml` tích hợp Checkstyle Google Style; `MessageSerializerTest` và `EchoClientTest` pass 100%.
- [ ] **Minh (Mảng 4):** `UserService` ném `AuthenticationException` và `AuctionException`; viết bài test `assertThrows` trong `UserServiceTest` pass 100%; file `data/users.json` lưu trữ người dùng thành công; có `docs/STYLE_GUIDE.md`.
- [ ] **Quân (Mảng 5):** Hàm `Auction.placeBid()` được bảo vệ an toàn bằng `ReentrantLock` và ném `InvalidBidException`, `AuctionClosedException`; `AuctionTest` có test `assertThrows` pass 100%; `AuctionConcurrencyTest` chạy 10–20 threads không race condition.

---

## 📊 PHẦN 7 — BẢNG ĐỐI CHIẾU TIÊU CHÍ CHẤM ĐIỂM (10.0/10.0 ĐIỂM BAREM TUẦN 9)

| Tiêu chí trong Barem Giảng viên (Trang 6 PDF) | Điểm | Hiện trạng nhóm đáp ứng | Trạng thái nghiệm thu |
| :--- | :---: | :--- | :---: |
| **1. Thiết kế lớp và cây kế thừa** | **0.5đ** | Đã xong từ Tuần 1 (`User`, `Item`, `Entity`, `Bid`, `Auction`). | ✅ 100% |
| **2. Áp dụng OOP (4 tính chất)** | **1.0đ** | Đã xong từ Tuần 1 (Đóng gói private, Đa hình `displayProfile`, Trừu tượng hóa Entity). | ✅ 100% |
| **3. Design Patterns phù hợp** | **1.0đ** | **Singleton** (`AuctionManager`, `SceneManager`), **Factory** (`ItemFactory`), **Observer** (`AuctionObserver`). | ✅ 100% |
| **4. Quản lý người dùng, sản phẩm** | **1.0đ** | CRUD User, CRUD Item, phân quyền Bidder / Seller / Admin. | ✅ 100% |
| **5. Chức năng đấu giá** | **1.0đ** | Logic đặt giá, kiểm tra giá sàn, bước giá, cập nhật trạng thái phòng. | ✅ 100% |
| **6. Xử lý lỗi & ngoại lệ** | **1.0đ** | Cây 4 Custom Exceptions + JUnit 5 `assertThrows` + Alert popup / Label chữ đỏ JavaFX. | ✅ 100% (Hoàn thiện đợt này) |
| **7. Xử lý đấu giá đồng thời** | **1.0đ** | `ReentrantLock(true)` bảo vệ `placeBid()` + Unit Test kiểm thử đồng thời. | ✅ 100% |
| **8. Realtime update (Observer/Socket)** | **0.5đ** | Observer Pattern nhảy số thời gian thực trên UI phòng đấu giá. | ✅ 100% |
| **9. Kiến trúc Client–Server** | **0.5đ** | `AuctionServer` lắng nghe port 8888, tiếp nhận Client gửi PING/PONG. | ✅ 100% |
| **10. MVC (JavaFX + FXML + Model)** | **0.5đ** | FXML View tách biệt Controller, gọi sang Model xử lý dữ liệu. | ✅ 100% |
| **11. Maven & Coding convention** | **0.5đ** | Maven Multi-Module + `maven-checkstyle-plugin` Google Style toàn diện. | ✅ 100% |
| **12. Unit Test (JUnit 5)** | **0.5đ** | Bộ test tự động phủ khắp Domain, Network, Observer, Exception (`mvn test` xanh 100%). | ✅ 100% |
| **13. CI/CD (GitHub Actions)** | **0.5đ** | File `.github/workflows/maven-ci.yml` tự động build và chạy test trên repo GitHub. | ✅ 100% |
| **TỔNG ĐIỂM BẮT BUỘC** | **10.0đ** | **ĐẠT TRỌN VẸN 10.0/10.0 ĐIỂM BẮT BUỘC TUẦN 9** | 🎯 **XUẤT SẮC** |

---

## ⏰ PHẦN 8 — TIMELINE HÀNH ĐỘNG 3 NGÀY NƯỚC RÚT (05/11 – 07/11)

| Thời gian | Trọng tâm công việc từng thành viên | Mục tiêu cuối ngày |
| :--- | :--- | :--- |
| **Thứ 5 (05/11)** | • **Cả 4 bạn:** Gõ `git pull origin dev` lấy 6 file khung dùng chung.<br>• **Minh:** Viết `JsonStorage.java` + hoàn thiện `UserService` (ném `AuthenticationException`).<br>• **Quân:** Đặt `ReentrantLock` và ném Exception trong `Auction.java` + cập nhật `AuctionTest.java`.<br>• **Tuấn:** Hoàn thiện `BiddingRoomView.fxml` + gắn Observer nhảy số + Popup Alert.<br>• **Hùng:** Hoàn thiện `LoginView.fxml` + bắt lỗi chữ đỏ.<br>• **Sang:** Rà soát Checkstyle, hỗ trợ các thành viên chạy test. | Toàn bộ 5 nhánh cá nhân code xong các chức năng tinh gọn. |
| **Thứ 6 (06/11)** | • Từng bạn chạy kiểm thử toàn bộ trên máy: `mvn clean test` và `mvn checkstyle:check`.<br>• Đẩy commit cuối cùng lên nhánh cá nhân, mở Pull Request vào nhánh `dev`.<br>• Tiến hành review chéo theo ma trận và approve PR. | Merge thành công 5 PR vào nhánh `dev`. GitHub Actions báo xanh 100%. |
| **Thứ 7 (07/11)** | • **HỌP NHÓM & TỔNG DUYỆT BÁO CÁO:**<br>1. Demo luồng chạy ứng dụng JavaFX (Login $\rightarrow$ Đặt giá nhảy số $\rightarrow$ Báo lỗi popup).<br>2. Demo Server Socket phản hồi PING/PONG trên console.<br>3. Kiểm tra chéo 8 câu hỏi vấn đáp miệng theo Phần 1. | **Sẵn sàng 100% cho Giảng viên chấm tiến độ Tuần 9!** |
