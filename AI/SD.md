1. 資料庫詳細設計（ERD / Table Schema）
   本節將定義支援訂票系統核心功能所需的資料表結構。與 SA 僅有概念不同，SD 必須明確定義欄位名稱、資料型態、主鍵與關聯。
   1.1 資料表清單
   資料表
   欄位名稱
   型態
   說明
   關聯
   User (乘客)
   id
   INT
   主鍵 (PK)
   1:N (Order)
   username
   VARCHAR(50)
   乘客帳號 (登入/唯一)
   password
   VARCHAR(100)
   密碼 (加密儲存)
   Session (車次)
   id
   INT
   主鍵 (PK)
   1:N (Seat, Order)
   departure_station
   VARCHAR(30)
   起站
   arrival_station
   VARCHAR(30)
   訖站
   departure_time
   DATETIME
   發車時間
   price
   INT
   基本票價
   Seat (座位)
   id
   INT
   主鍵 (PK)
   N:1 (Session)
   session_id
   INT
   外鍵 (FK)
   屬於哪個車次
   seat_number
   VARCHAR(10)
   座位編號 (如: 1A, 2C)
   status
   VARCHAR(20)
   座位狀態 (可用/已鎖定/已售出)
   Order (訂單)
   id
   INT
   主鍵 (PK)
   1:N (OrderItem)
   user_id
   INT
   外鍵 (FK)
   訂票者
   session_id
   INT
   外鍵 (FK)
   訂購的車次
   total_amount
   INT
   總金額
   status
   VARCHAR(20)
   訂單狀態 (待付款/已完成/已取消)
   1.2 實體關係圖（ERD）
   erDiagram
   USER ||--o{ ORDER : has
   SESSION ||--o{ SEAT : has
   SESSION ||--o{ ORDER : belongs

   USER {
   int id PK
   string username
   string password
   }

   SESSION {
   int id PK
   string departure_station
   string arrival_station
   datetime departure_time
   }

   SEAT {
   int id PK
   int session_id FK
   string seat_number
   string status
   }

   ORDER {
   int id PK
   int user_id FK
   int session_id FK
   int total_amount
   string status
   }
2. API 詳細設計（前後端互動規格）
   本節定義前後端互動所需的 API 規格，這是 Gemini CLI 生成 Controller 的直接依據,。
   API
   Method
   URL
   說明
   Request (Input)
   Response (Output)
   查詢車次列表
   GET
   /sessions
   依條件查詢可訂購車次。
   ?dep={} &arr={} &date={}
   JSON Array of Session (包含 ID, Time, Price)
   建立訂單
   POST
   /api/orders
   暫時鎖定座位並建立訂單草稿。
   user_id, session_id, seat_list[]
   JSON: order_id, total_amount, status: PENDING
   訂單付款
   POST
   /api/orders/{id}/pay
   模擬金流，確認訂單並變更狀態。
   order_id, payment_info
   JSON: status: SUCCESS
   管理員：新增車次
   POST
   /admin/sessions/new
   新增車次資料。
   departure_station, arrival_station, time, price
   Redirect to /admin/sessions
   管理員：查詢訂單
   GET
   /admin/orders
   查詢所有訂單紀錄。
   ?status={}
   Thymeleaf HTML 頁面 (包含訂單列表)
3. 主要流程詳細描述（Main Flow Detailed Sequence）
   此處詳細描述**「建立訂單並鎖定座位」**的核心流程。流程必須具體到能寫程式，不可跳步，且必須拆解為 7–12 步,。我們將此流程分為四個層次,。
   3.1 關鍵業務流程圖（Sequence Diagram）
   sequenceDiagram
   participant User as 乘客
   participant Web as 前端頁面
   participant Ctl as OrderController
   participant Svc as BookingService
   participant Repo as SeatRepository/OrderRepository
   participant DB as Database

   User->>Web: 1. 選擇車次與座位 (POST 請求)
   Web->>Ctl: 2. POST /api/orders (Input: user_id, session_id, seat_list)
   Ctl->>Svc: 3. 呼叫 createOrder(data)

   Svc->>Repo: 4. 檢查座位庫存與狀態 (findBySeatIdAndStatus)
   Repo->>DB: 5. SELECT * FROM seat WHERE id IN (...) AND status = 'AVAILABLE'
   DB-->>Repo: 6. 回傳結果

   alt 座位全部可用
   Svc->>Svc: 7. 計算總金額
   Svc->>Repo: 8. 鎖定座位 (UPDATE seat SET status = 'LOCKED')
   Repo->>DB: 9. UPDATE Seat
   Svc->>Repo: 10. 寫入 Order 紀錄 (INSERT INTO Order)
   Repo->>DB: 11. COMMIT Transaction
   Svc-->>Ctl: 12. 回傳 Order ID
   else 座位不可用
   Svc-->>Ctl: 8. 拋出錯誤 (SeatUnavailableException)
   end

   Ctl-->>Web: 13. 回傳結果 (JSON: Order ID 或錯誤訊息)
   Web-->>User: 14. 顯示訂單草稿或錯誤提示
4. 畫面元件設計（UI Component Design）
   本節依據 SA 定義的頁面，列出前端頁面（Thymeleaf HTML）上所需的主要元件和欄位，以輔助 Gemini CLI 生成 HTML 模板,。
   4.1 前台：訂票頁 (/booking/{id})
   元件類型
   欄位/資訊
   說明
   對應 Entity 欄位
   資訊顯示
   車次資訊 (Session)
   顯示起訖站、時間、價格。
   Session
   互動元件
   座位選擇區
   顯示座位圖，並標示狀態（可用/已售出/已選）。
   Seat
   表單輸入
   票種/數量選擇
   選擇成人票、孩童票等。
   N/A
   按鈕
   確認訂票/付款
   觸發 POST /api/orders API。
   N/A
   4.2 後台：車次管理頁 (/admin/sessions/new)
   元件類型
   欄位/資訊
   說明
   對應 Entity 欄位
   表單輸入
   起站 (Departure)
   下拉選單或輸入框。
   Session.departure_station
   表單輸入
   訖站 (Arrival)
   下拉選單或輸入框。
   Session.arrival_station
   表單輸入
   發車時間
   時間選擇器。
   Session.departure_time
   表單輸入
   基本票價
   數字輸入框。
   Session.price
   按鈕
   儲存/送出
   觸發 POST /admin/sessions/new API。
   N/A
5. 相關類別與方法（Class Design 摘要）
   SD 必須定義主要類別（Controller, Service, Repository）之間的關係與方法，作為程式碼生成的基礎。
   5.1 訂票模組（Booking Module）
   classDiagram
   class OrderController {
   +createOrder(Request) JSON
   +listOrders(Model model) String
   }

   class BookingService {
   +createOrder(OrderDto dto) Order
   +findById(id) Order
   }

   class OrderRepository {
   +save(Order) Order
   +findByUserId(id) List~Order~
   }

   OrderController --> BookingService
   BookingService --> OrderRepository