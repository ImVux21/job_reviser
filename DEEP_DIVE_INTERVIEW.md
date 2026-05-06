# 🧠 BỘ CÂU HỎI PHỎNG VẤN "XOÁY SÂU" (DEEP-DIVE) - PROJECT: PRODDELMONTEUI

> **Lưu ý dành cho bạn:** Bộ câu hỏi này được thiết kế theo tư duy của một **Lead/Architect**. Người phỏng vấn sẽ không dừng lại ở câu trả lời đầu tiên mà sẽ bám vào các chi tiết kỹ thuật bạn đưa ra để "vặn" (drill-down). 
> **Tỉ lệ:** 80% Angular Architect - 20% System Design/Backend.

---

## 🏗️ PILLAR 1: KIẾN TRÚC MONOREPO & BUILD SYSTEM

**Câu hỏi Starter:** "Tôi thấy bạn quản lý dự án theo mô hình Monorepo với `pnpm workspace`. Tại sao bạn lại chọn hướng tiếp cận này cho dự án Hotel PMS thay vì chia nhỏ các repo riêng biệt?"

*   **Trả lời (STAR):**
    *   **Situation:** Hệ thống PMS của chúng em gồm nhiều ứng dụng (App quản lý khách sạn, App dọn phòng, App POS) có chung rất nhiều logic nghiệp vụ và thành phần giao diện.
    *   **Task:** Cần một cơ chế chia sẻ mã nguồn (Code sharing) hiệu quả, đảm bảo tính nhất quán (Consistency) và tốc độ phát triển (Velocity).
    *   **Action:** Em triển khai Monorepo dùng `pnpm workspace`, tách các UI component dùng chung vào package `@tappy/components` và business logic vào `@tappy/core`.
    *   **Result:** Tăng tốc độ phát triển lên 50% khi build app mới, các thay đổi tại library được cập nhật ngay lập tức (Instant feedback) tới tất cả các app mà không cần publish lên NPM.

**🔥 XOÁY SÂU (Drill-down):** "Làm thế nào bạn xử lý được vấn đề **Circular Dependencies** (phụ thuộc vòng) giữa các package trong Monorepo? Ví dụ: `Package A` dùng `B`, nhưng `B` lại vô tình gọi ngược lại một helper trong `A`?"

*   **Trả lời chuyên gia:**
    *   "Đây là vấn đề rất dễ gặp khi team scale lớn. Em sử dụng 2 lớp 'bảo vệ':
        1.  **Kiến trúc Layered**: Ép buộc quy tắc chỉ có `App` mới được phụ thuộc vào `Library`, và `Library` cấp cao (`@tappy/components`) mới được phụ thuộc vào `Library` thấp hơn (`@tappy/core`). Tuyệt đối không có chiều ngược lại.
        2.  **Tooling (Es-lint)**: Em cấu hình rule `import/no-cycle` và sử dụng `nx graph` (hoặc pnpm graph) để visualize các mối quan hệ. Nếu phát hiện vòng lặp, em sẽ tách phần code chung đó ra một package 'base' mới hoặc sử dụng `InjectionToken` trong Angular để thực hiện Dependency Inversion, giúp 'phá' liên kết cứng giữa hai bên."

---

## 💾 PILLAR 2: TẦNG DỮ LIỆU & REACTIVITY (DATABUS & DATASOURCE)

**Câu hỏi Starter:** "Bạn giới thiệu về `DataBusService` như một Centralized Hub. Tại sao bạn không dùng NgRx hay Signals Store cho khỏe, mà lại đi tự build một tầng trung gian phức tạp như vậy?"

*   **Trả lời (STAR):**
    *   **Situation:** Đặc thù của hệ thống PMS là dữ liệu rất lớn và có tính 'stateless' cao (data thay đổi liên tục từ server). NgRx thường quá cồng kềnh với nhiều boilerplate.
    *   **Action:** Em hướng tới sự tối giản bằng cách dùng **Facade Pattern**. `DataBus` quản lý một Map các `DataSource`. Khi component cần data, nó gọi qua `DataBus`.
    *   **Result:** Giảm 40% request dư thừa nhờ cơ chế cache instance DataSource. Code Component sạch hơn vì chỉ quan tâm đến `DataSource.items()`.

**🔥 XOÁY SÂU (Drill-down):** "Trong code của bạn, `ApiDataSource` có cơ chế tự động reload sau mutation. Vậy nếu 2 component cùng dùng chung 1 DataSource instance, làm sao bạn đảm bảo khi 1 bên trigger 'reload', bên còn lại không bị 'flicker' (giật lag) giao diện hoặc bị trigger logic lặp lại?"

*   **Trả lời chuyên gia:**
    *   "Vấn đề này em đã xử lý thông qua trạng thái **`status` của DataSource** (một BehaviorSubject/Signal). Khi một mutation diễn ra (ví dụ: Update Booking), em gọi `reloadAfterMutation()`. 
    *   Bên trong hàm này, trước khi gọi API mới, em giữ lại data cũ trong biến `shadowData`. Lúc này status chuyển sang `loading` nhưng UI vẫn hiển thị data cũ (hoặc skeleton mờ). 
    *   Quan trọng nhất: Em sử dụng toán tử `shareReplay(1)` hoặc Signal memoization để đảm bảo các component subscribe sau không trigger lại request. Mọi thành phần dùng chung instance này sẽ cùng nhận được thông báo 'Reload' đồng thời, đảm bảo tính Single Source of Truth tuyệt đối."

---

## ⚡ PILLAR 3: TỐI ƯU HIỆU NĂNG CAO ĐỘ (TIMELINE 60 FPS)

**Câu hỏi Starter:** "Timeline grid chứa hàng ngàn booking là một bài toán rất nặng về rendering. Bạn đã làm những gì để duy trì mức 60 FPS ổn định?"

*   **Trả lời (STAR):**
    *   **Task:** Hiển thị ma trận phòng khách sạn theo thời gian, hỗ trợ kéo thả (drag & drop) mà không lag.
    *   **Action:** 
        1. Ép buộc **100% OnPush** Change Detection.
        2. Sử dụng **Virtual Scroll** (CDK) cho cả trục ngang và dọc.
        3. Tách logic tính toán vị trí Event ra khỏi render loop bằng cách dùng **Memoization Cache** (`eventPositionsCache`).
    *   **Result:** Ứng dụng mượt mà ngay cả khi chạy trên các máy tính cấu hình trung bình tại khách sạn.

**🔥 XOÁY SÂU (Drill-down):** "Khi scroll đồng bộ giữa header (ngày tháng) và body (danh sách phòng), thường sẽ có hiện tượng 'so le' (jitter) do chênh lệch thời gian render. Bạn xử lý cái Jitter này như thế nào?"

*   **Trả lời chuyên gia:**
    *   "Đúng, đây là lỗi kinh điển. Em đã giải quyết bằng cách kết hợp **`requestAnimationFrame`** và chạy **ngoài Angular Zone**. 
    *   Cụ thể: Em lắng nghe sự kiện scroll từ body trong `NgZone.runOutsideAngular`. Khi user scroll, em dùng `nativeElement.setScrollLeft()` trực tiếp cho header bên trong một callback của `requestAnimationFrame`. 
    *   Việc này giúp đồng bộ hóa việc di chuyển của 2 element ở mức độ hardware-accelerated, trước khi browser thực hiện quá trình 'paint' tiếp theo. Chỉ khi nào cần cập nhật các trạng thái logic (như load thêm data trang mới), em mới quay lại vào Inside Zone để trigger change detection."

*(Tạm dừng để bạn đọc và ngấm 3 Pillar cực nặng này. Nếu bạn thấy ổn, tôi sẽ tiếp tục soạn 3 Pillar tiếp theo bao gồm Dynamic Forms, System Design cho Big Data và Backend Integration)*

---

## 📋 PILLAR 4: PRODUCTIVITY ENGINEERING (DYNAMIC FORM TEMPLATES)

**Câu hỏi Starter:** "Tôi thấy bạn xây dựng một kiến trúc Dynamic Form giúp giảm 50% thời gian code form mới. Bạn có thể giải thích cơ chế hoạt động cốt lõi của `<tap-form-template>` không?"

*   **Trả lời (STAR):**
    *   **Action:** Thay vì viết code HTML/Typescript cho từng form, em định nghĩa form qua một file cấu hình JSON (`FormElement[]`). `FormTemplateComponent` sẽ duyệt qua mảng này và sử dụng một `FormDirective` để render các component tương ứng (`tap-input`, `tap-select`, v.v.)
    *   **Result:** Developer chỉ cần tập trung vào logic data, UI layer được tự động hóa hoàn toàn. Tốc độ bàn giao feature tăng gấp đôi.

**🔥 XOÁY SÂU (Drill-down):** "Làm thế nào bạn giải quyết bài toán **Conditional Validation**? Ví dụ: Field B bắt buộc (required) chỉ khi Field A có giá trị là 'VIP'. Bạn handle logic này ở đâu trong file cấu hình?"

*   **Trả lời chuyên gia:**
    *   "Đây là bài toán về **Reactive Context**. Trong cấu hình của mỗi field, em thêm một thuộc tính `validators`. Thay vì truyền mảng validator tĩnh, em cho phép truyền một function nhận vào `context` (bao gồm giá trị hiện tại của toàn bộ form qua Signals).
    *   Khi người dùng gõ vào Field A, một `effect()` trong `FormDirective` sẽ được trigger để tính toán lại trạng thái của các field phụ thuộc. Em dùng `computeValidators(field, currentFormValues)` để sinh ra danh sách validator mới và dùng `setValidators()` để cập nhật cho `FormControl` tương ứng. Toàn bộ logic này được đóng gói bên trong Directive, developer không cần viết một dòng code validation thủ công nào trong component."

---

## 🌐 PILLAR 5: SYSTEM DESIGN & BIG DATA (MOBIO CONTEXT)

**Câu hỏi Starter:** "Tại Mobio, chúng tôi xử lý hàng triệu bản ghi khách hàng (CDP). Với trải nghiệm làm việc trên tập dữ liệu lớn tại Delmonte, bạn sẽ thiết kế dashboard như thế nào để đảm bảo hiệu suất render?"

*   **Trả lời chuyên gia:**
    *   "Với Big Data trên Frontend, rào cản lớn nhất không phải là JS xử lý chậm mà là **DOM Bottleneck**. Chiến lược của em sẽ gồm 3 lớp:
        1.  **Data Windowing (Virtualization)**: Chỉ render những gì user thấy trên màn hình. Mọi bảng dữ liệu hoặc list đều phải qua `CdkVirtualScroll`.
        2.  **Web Workers**: Chuyển các tác vụ nặng như Filtering, Sorting, Grouping hàng chục ngàn bản ghi xuống Web Worker để không block Main Thread (tránh hiện tượng đóng băng UI).
        3.  **OnPush & Signals**: Đảm bảo Angular chỉ kiểm tra những component có data thực sự thay đổi. Kết hợp với `ChangeDetectionStrategy.OnPush` và `Zoneless` (nếu dùng Angular 19) để đạt hiệu năng tối đa."

**🔥 XOÁY SÂU (Drill-down):** "Nếu backend trả về một cục JSON nặng 50MB chứa toàn bộ data khách hàng, trình duyệt chắc chắn sẽ crash. Bạn sẽ 'deal' với team Backend như thế nào để tối ưu luồng dữ liệu này?"

*   **Trả lời chuyên gia:**
    *   "Em sẽ đề xuất implement **Keyset Pagination (Seek Method)** thay vì Offset Pagination để query DB nhanh hơn. 
    *   Về mặt API, em sẽ yêu cầu hỗ trợ **Field Selection** (chỉ trả về các field cần hiển thị trên Table, các field chi tiết chỉ load khi click vào). 
    *   Đồng thời, em sẽ đề xuất sử dụng **Server-Sent Events (SSE)** hoặc **Websockets** để stream dữ liệu theo từng block nhỏ, giúp Frontend có thể hiển thị dữ liệu 'ngay và luôn' (Progressive Loading) thay vì phải đợi tải xong 50MB."

---

## ☕ PILLAR 6: BACKEND INTEGRATION & FULLSTACK MINDSET

**Câu hỏi Starter:** "Bạn từng làm Fullstack với Java Spring Boot. Kinh nghiệm này giúp ích gì cho bạn khi làm Frontend hiện tại?"

*   **Trả lời chuyên gia:**
    *   "Nó giúp em có cái nhìn 'End-to-End'. Em không chỉ coi API là một cái URL, mà em hiểu cách DB được index, cách Transaction được handle. Điều này giúp em thiết kế **API Contracts** tốt hơn, giảm thiểu số lượng vòng lặp gọi API (N+1 problem) và giúp team trace bug nhanh hơn rất nhiều khi có sự cố."

**🔥 XOÁY SÂU (Drill-down):** "Xung đột thường gặp nhất giữa FE và BE là về cấu trúc dữ liệu. Nếu BE trả về dữ liệu 'rác' hoặc không đúng format đã thỏa thuận, bạn sẽ xử lý như thế nào để App FE không bị sập (white screen)?"

*   **Trả lời chuyên gia:**
    *   "Em sử dụng **Adapter Pattern** tại tầng `ApiDataSource`. Ngay khi data vừa về từ `HttpClient`, nó phải đi qua một lớp 'kiểm soát cửa khẩu'. 
    *   Em dùng **Zod hoặc các Type Guards** để validate schema của JSON. Nếu data thiếu field bắt buộc hoặc sai kiểu dữ liệu, Adapter sẽ tự động gán giá trị mặc định (Default values) và bắn một log cảnh báo về hệ thống giám soát (Sentry). Việc này giúp App FE luôn chạy ổn định dù Backend có lỗi, đồng thời có bằng chứng cụ thể để làm việc lại với team Backend."

---

## 🎯 LỜI KHUYÊN CUỐI CÙNG CHO BUỔI PHỎNG VẤN

1.  **Luôn nhắc đến "Monorepo" và "Standardization":** Mobio thích những người có tư duy xây dựng hệ thống quy chuẩn.
2.  **Tự tin về Signals:** Đây là thế mạnh của bạn so với những ứng viên chỉ biết RxJS truyền thống. Hãy nhấn mạnh việc bạn đã "ép buộc" team dùng Signals ra sao.
3.  **Thái độ quyết liệt:** Khi trả lời các câu hỏi "xoáy", hãy thể hiện rằng bạn là người chủ động đưa ra giải pháp chứ không đợi task.

**Chúc bạn tỏa sáng tại Mobio!**

