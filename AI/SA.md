1. 系統分層概觀 (System Layer Overview)
   本系統採用標準四層式架構，目的是確保職責分離，並支援前後台的資料流動。
   層次名稱
   職責 (Responsibilities)
   範例模組
   前台 (WebUI)
   顯示資料與使用者互動 (如查詢車次、座位選擇)
   Thymeleaf HTML 模板
   後台 (Controller/API)
   接收 HTTP 請求、驗證輸入、並協調業務邏輯
   SessionController, OrderController
   服務層 (Service)
   執行核心業務邏輯 (如訂票、座位鎖定、退改票)
   BookingService, SeatService
   資料層 (Repository/DB)
   資料存取與持久化 (CRUD 操作)
   OrderRepository, SessionRepository
   資料流向說明：使用者操作 WebUI → 發送 HTTP 請求至 Controller → 呼叫 Service 處理邏輯 → 透過 Repository 存取 DB。
2. 系統元件描述與技術選用
   2.1 系統元件
   • 前台元件 (WebUI)：負責乘客端的查詢、訂票、訂單管理等頁面。
   • 後台元件 (AdminUI)：負責管理員的車次、座位庫存、訂單紀錄與銷售報表等頁面。
   • 服務分層 (Service Layer)：根據業務劃分，可能包含 SessionService（車次管理）、OrderService（訂單處理）、SeatService（座位鎖定）等核心業務模組。
   2.2 技術選用 (Technology Selection)
   本專案必須遵循專題統一樣板規範，採用以下技術與工具：
   類別
   選用技術
   用途與說明
   後端框架
   Java (17+), Spring Boot 3.x
   後端 API 與業務邏輯實作。
   前端模板
   Thymeleaf
   用於渲染前台與後台的 HTML 頁面。
   資料庫
   MySQL 或 H2 (開發階段)
   用於儲存車次、座位、訂單等資料。
   AI 工具
   NotebookLM, Gemini CLI
   NotebookLM 進行文件審查，Gemini CLI 產生程式碼骨架。
   版本控制
   GitHub
   團隊協作與版本控管。
3. 頁面架構列表 (Page Architecture) — 生成骨架的關鍵
   這是 SA 文件中最核心的部分，Gemini CLI 將依此列表生成專案的 Controller/Service 骨架以及對應的 Thymeleaf HTML 模板。本節只需列出「頁面名稱」與「該頁面主要功能/按鈕」，無需列出詳細欄位。
   3.1 前台頁面 (Frontend Pages)
   頁面名稱
   URL
   主要功能/資訊
   關鍵按鈕
   首頁/查詢頁
   /index
   起訖站與日期輸入欄位
   查詢車次
   車次列表頁
   /sessions
   顯示車次編號、時間、剩餘座位
   選擇座位/訂票
   座位選擇/訂票頁
   /booking/{id}
   顯示座位圖、票種選擇、訂單資訊
   確認訂票、線上付款
   訂單管理頁
   /orders
   顯示使用者所有訂單紀錄
   查詢詳情、改票、退票
   電子票券頁
   /ticket/{id}
   顯示 QR Code 與電子票資訊
   下載票券
   3.2 後台頁面 (Backend Pages)
   頁面名稱
   URL
   主要功能/資訊
   關鍵按鈕
   管理員登入頁
   /admin/login
   管理員帳號密碼輸入
   登入
   車次管理頁
   /admin/sessions
   車次 CRUD 列表 (新增/修改車次)
   新增車次、編輯、刪除
   座位/票量管理頁
   /admin/seats/{session_id}
   管理特定車次的座位庫存
   更新座位資訊
   訂單紀錄查詢頁
   /admin/orders
   顯示所有訂單列表（含篩選功能）
   查看詳情
   銷售報表頁
   /admin/reports
   顯示銷售統計與報表生成
   下載報表
4. 系統架構圖 (System Architecture Diagram)
   使用 Mermaid 語法呈現本系統的高階分層結構，強調模組之間的關係：
   graph TD
   User[使用者/管理員] -->|Browser| WebUI[前端頁面 Thymeleaf]
   WebUI -->|HTTP Request| Controller[控制層 Controller]
   Controller -->|呼叫方法| Service[服務層 Service]
   Service -->|JPA/SQL| Repo[資料存取層 Repository]
   Repo -->|JDBC| DB[(資料庫 MySQL/H2)]
5. 關鍵業務流程順序圖 (User Sequence Diagram)
   針對核心的「訂票流程」，以 Mermaid 順序圖繪製業務流程動線。此圖應著重於使用者與系統頁面之間的互動與回應，避免寫出程式內部的函數名稱。
   範例：使用者訂票與付款流程
   sequenceDiagram
   participant User as 乘客
   participant Web as 前台頁面
   participant System as 後端系統(Controller/Service)

   User->>Web: 1. 輸入起訖站/日期 (查詢車次)
   Web->>System: 2. 請求車次列表 API
   System-->>Web: 3. 回傳車次與剩餘座位

   User->>Web: 4. 選擇車次與座位 (訂票頁)
   Web->>System: 5. 呼叫鎖定座位/產生訂單草稿 API
   System-->>Web: 6. 回傳訂單草稿 (座位暫時鎖定)

   User->>Web: 7. 點擊「線上付款」並輸入資訊
   Web->>System: 8. 呼叫付款/確認訂單 API (Create Order)
   System->>System: 9. 處理金流模擬 (依據限制與假設)
   System-->>Web: 10. 訂單建立成功

   Web-->>User: 11. 顯示電子票券頁