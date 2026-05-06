# 🎯 TOÀN TẬP 70 CÂU HỎI PHỎNG VẤN CHUYÊN SÂU (DỰA TRÊN PRODDELMONTEUI)

> **Lưu ý:** Tất cả các câu trả lời dưới đây đều sử dụng thuật ngữ, logic và cấu trúc thực tế từ mã nguồn dự án **ProdDelmonteUI**. Hãy đọc kỹ các chi tiết kỹ thuật (tên biến, tên class) để thể hiện sự trung thực và am hiểu sâu sắc về dự án.

---

## 🏗️ PHẦN 1: KIẾN TRÚC DỮ LIỆU & QUẢN LÝ TRẠNG THÁI (DATA LAYER)

**1. DataBusService trong dự án của bạn đóng vai trò gì? Tại sao không dùng Service bình thường?**
- **Trả lời:** `DataBusService` đóng vai trò là một Centralized Data Hub. Thay vì mỗi component tự inject và gọi API service rườm rà, em dùng `DataBusService.api(path)` để lấy về một `ApiDataSource`. Điểm đặc biệt là nó quản lý một `dataSourceCache` (kiểu `Map`). Nếu nhiều component cùng yêu cầu một tập dữ liệu (ví dụ: danh sách Hotel), nó sẽ trả về cùng một instance DataSource, giúp đồng bộ dữ liệu toàn cục mà không cần NGRX.

**2. Giải thích cơ chế tạo Cache Key trong DataBusService?**
- **Trả lời:** Em sử dụng method `createCacheKey(type, path, config)`. Đối với các request có query params, em thực hiện sắp xếp các key của Object query theo bảng chữ cái (alphabetical sort) trước khi nối chuỗi: `parts.push(dynamic-query:${queryString})`. Điều này đảm bảo rằng dù thứ tự truyền params khác nhau, cache key vẫn trùng khớp, tránh việc tạo thừa DataSource.

**3. ApiDataSource trong hệ thống của bạn khác gì so với HttpClient thông thường?**
- **Trả lời:** `ApiDataSource` kế thừa từ `SimpleDataSource` và tích hợp sẵn trạng thái `status` (BehaviorSubject) với các giá trị: `loading`, `loaded`, `error`, `empty`. Nó tự động handle việc gọi API, quản lý dữ liệu trong field `_data` và hỗ trợ `Adapter Pattern` để chuyển đổi dữ liệu thô từ server thành business objects ngay tại tầng data.

**4. "Loader Mode" trong ApiDataSource được dùng khi nào?**
- **Trả lời:** Trong code của em, `options.loader` là một function trả về một `Observable`. Khi được cấu hình, `ApiDataSource` sẽ bỏ qua việc gọi API qua URL thông thường mà thực thi loader này. Điều này cực kỳ hữu ích khi em cần thực hiện các logic fetching phức tạp (như kết hợp dữ liệu từ 2 nguồn) trước khi đẩy vào DataSource.

**5. Giải thích cách hệ thống xử lý "Auto Reload" sau khi đột biến dữ liệu (Mutations)?**
- **Trả lời:** Trong `ApiDataSource`, em có hàm `reloadAfterMutation()`. Khi thực hiện các thao tác `create`, `updateByKey`, hoặc `deleteByKey`, sau khi API thành công, hàm này sẽ tự động gọi `this.clearCache()` và `this.load()` lại chính DataSource đó. Điều này giúp UI luôn đồng bộ với Server mà không cần code logic refresh thủ công ở Component.

**6. Adapter Pattern được ứng dụng như thế nào trong lớp dữ liệu?**
- **Trả lời:** Em dùng `Adapter` interface với method `apply(data)`. Khi `ApiDataSource` nhận data từ API, nếu có `adapter` được cấu hình, nó sẽ gọi `this.options.adapter.apply(keyData)`. Ví dụ: chuyển đổi string date từ API thành `JS Date` object hoặc tính toán thêm các field ảo trước khi data "chạm" tới UI.

**7. Tại sao entityKey trong hệ thống của bạn lại mặc định là "code"?**
- **Trả lời:** Do quy chuẩn của hệ thống Backend (Stateless), các thực thể thường được định danh bằng mã (code) duy nhất thay vì ID tăng dần. Tuy nhiên, em thiết kế `ApiDataSource` linh hoạt để có thể override qua `options.entityKey`, ví dụ như dùng `id` cho một số module cũ.

**8. Cách xử lý Pagination với LazyDataSource?**
- **Trả lời:** Em dùng `LazyDataSource` hỗ trợ `pageSize`. Nó không load toàn bộ mà chỉ load theo từng "trang" khi UI yêu cầu. Trạng thái pagination được quản lý tập trung trong DataSource, giúp các component như Table hay Scroll có thể tương tác dễ dàng qua các method `loadNext()`, `loadPrevious()`.

**9. Tại sao bạn dùng `untracked()` trong Effect khi khởi tạo DataSource?**
- **Trả lời:** Đây là kỹ thuật tối ưu trong Angular 18/19. Khi khởi tạo DataSource trong một `effect()`, vì việc subcribe vào status của DS sẽ trigger ghi vào các Signals (`flattenedItems`, `eventLookup`), nếu không dùng `untracked()`, các thay đổi này sẽ làm `effect` tự kích hoạt lại vô tận (infinite loop).

**10. "Shadow Data" trong ApiDataSource dùng để làm gì?**
- **Trả lời:** Em dùng `this.shadowData = Array(this.options.count).fill({})`. Khi UI đang ở trạng thái `loading`, shadow data này giúp em render ra các "Skeleton rows" với số lượng chính xác như dữ liệu thật sắp về, tạo cảm giác mượt mà hơn cho người dùng (Skeleton Screen pattern).

---

## ⚡ PHẦN 2: HIỆU NĂNG ĐỈNH CAO & TIMELINE COMPONENT (60 FPS)

**11. TimelineComponent trong dự án rất phức tạp (34KB), bạn đã tối ưu nó như thế nào?**
- **Trả lời:** Thành công lớn nhất là đạt mức 60 FPS khi scroll. Em áp dụng 3 tầng Cache chuyên biệt: `eventsInCellCache` (chứa dữ liệu sự kiện theo ô), `eventPositionsCache` (vị trí hiển thị), và `titleRenderCache` (quyết định ô nào được hiển thị title để tránh trùng lặp khi sự kiện kéo dài qua nhiều ô).

**12. Bạn xử lý việc tính toán vị trí Event trong Timeline như thế nào để không gây lag?**
- **Trả lời:** Em sử dụng `TimelineUnitStrategy` (Abstract Class). Tùy vào view (Daily hay Hourly), strategy sẽ tính toán chính xác `getEventKey` và `getEventPosition`. Kết quả tính toán này được memoize ngay trong component, chỉ tính lại khi `timelineData` hoặc `unitsRange` thay đổi thực sự.

**13. Giải thích kỹ thuật "Manual Scroll Synchronization" bạn đã làm?**
- **Trả lời:** Trong Timeline, em có CdkVirtualScrollViewport và một Table header/container riêng biệt. Để đồng bộ scroll dọc giữa chúng mà không bị hiện tượng "so le" (jitter), em dùng `ResizeObserver` để theo dõi chiều cao và `elementScrolled()` của viewport để cập nhật `scrollTop` cho table container bằng tay, bọc trong `NgZone.runOutsideAngular` để tránh trigger change detection liên tục.

**14. Tại sao lại cần `forceViewportRender()` với các thủ thuật scrollToOffset(1) rồi (0)?**
- **Trả lời:** Đây là một "hack" kỹ thuật cho CdkVirtualScroll. Đôi khi dữ liệu thay đổi nhưng viewport không tự nhận diện để render lại các items đang hiển thị. Việc scroll nhẹ 1px rồi trả về 0px sẽ ép Virtual Scroll tính toán lại vùng render (viewport), đảm bảo UI luôn hiển thị đúng data mới nhất.

**15. Bạn ứng dụng `ChangeDetectorRef` vào đâu trong Timeline?**
- **Trả lời:** Mặc dù dùng 100% Signals, nhưng trong các hàm xử lý Drag & Drop (sử dụng native events bên ngoài Angular Zone), em dùng `detector.markForCheck()` hoặc `detectChanges()` để ép Angular cập nhật UI ngay lập tức sau khi các tính toán vị trí bằng tay hoàn tất.

**16. "Title Position Map" giúp ích gì cho việc hiển thị bảng Timeline?**
- **Trả lời:** Với các booking kéo dài 3-4 ngày, nếu ô nào cũng hiện tên khách thì UI sẽ rất rối. Em xây dựng `prepareTitlePositions()` để tính toán duy nhất 1 ô (thường là ô 'start' hoặc ô hiển thị đầu tiên trong viewport) để render title, các ô còn lại chỉ render background màu.

**17. `NgZone.runOutsideAngular` được dùng cụ thể ở đâu?**
- **Trả lời:** Em dùng trong `onCellMouseMove` khi người dùng đang "rê" một sự kiện (Drag). Việc tính toán `hoveredCell` và update `iconEventPosition` xảy ra liên tục (mỗi 10-20ms). Chạy ngoài Zone giúp tránh hàng trăm lượt Change Detection vô nghĩa, chỉ khi kết thúc Drag (Drop) em mới chạy vào Zone để cập nhật state cuối.

**18. Cách bạn xử lý Column Resize cho một bảng Timeline khổng lồ?**
- **Trả lời:** Em viết một `ColumnResizeService` riêng. Khi user kéo rộng một cột, service này sẽ update một `Map` chứa widths. Component dùng một `computed()` signal đặt tên là `columnsWithWidth` để tính toán lại style cho tất cả các ô. Việc dùng `Map` và `computed` giúp UI update cực nhanh mà không cần render lại toàn bộ component.

**19. `Abstract Class TimelineComponent` giải quyết bài toán đa dạng hóa view như thế nào?**
- **Trả lời:** Em định nghĩa các method abstract như `strategy`. App PMS có 2 view chính: Daily (quản lý theo ngày) và Hourly (quản lý theo giờ). Mỗi view sẽ kế thừa `TimelineComponent` và cung cấp một `strategy` riêng, giúp tái sử dụng 90% logic hiển thị và interaction phức tạp.

**20. TrackBy trong Timeline của bạn được cấu hình như thế nào?**
- **Trả lời:** Em dùng 3 tầng trackBy: `trackByRowId` (cho hàng), `trackByUnit` (cho cột ngày/giờ), và `trackByColumnKey` (cho các cột thông tin phụ). Đây là yếu tố sống còn để Angular không "re-paint" lại cả bảng khi chỉ có một vài booking bị thay đổi trạng thái.

---

## 📋 PHẦN 3: HỆ THỐNG FORM ĐỘNG & UI COMPONENTS (CONFIG-DRIVEN)

**21. FormTemplateComponent nhận cấu hình như thế nào để render dynamic?**
- **Trả lời:** Component nhận input `fields()` kiểu `FormElement[]`. Em hỗ trợ nhiều loại element: `fieldset` (nhóm field), `tabs` (chia tab), và `columns` (chia cột). Logic render được xử lý đệ quy qua `resolveFieldsWithContext` để đảm bảo context được truyền sâu nhất tới từng field.

**22. Giải thích cơ chế "Context Merging" để ẩn/hiện field động?**
- **Trả lời:** Đây là phần em tâm đắc nhất. Trong hàm `isFieldHidden(field)`, em gộp base `context()` với live `_formValues()` (lấy từ Signal của form đang gõ). Nhờ đó, em có thể viết logic: "Ẩn field B nếu field A đang có giá trị là 'X'". Toàn bộ logic này được xử lý reactive qua Signal.

**23. `conditionalConfig` giúp bạn xử lý các mode Create/Update/View như thế nào?**
- **Trả lời:** Thay vì viết 3 cái form, em dùng 1 form duy nhất. Mỗi field có một `conditionalConfig`. Ví dụ: Ở mode `view`, em override `disabled: true`, ở mode `create`, em override `initValue: 'Default'`. Hàm `resolveFieldWithContext` sẽ tự động "merge" các cấu hình này dựa trên `context.mode`.

**24. Tại sao bạn lại dùng `viewChild(FormDirective)` thay vì `@ViewChild`?**
- **Trả lời:** Em dùng `viewChild` (signal-based) của Angular 17+. Nó cho phép em dùng `computed()` hoặc `effect()` để theo dõi khi nào FormDirective thực sự khả dụng (không bị null) và trigger các logic đồng bộ dữ liệu ngay lập tức mà không cần `ngAfterViewInit`.

**25. Cách bạn đồng bộ giá trị Form ngược lại Model (`this.value`)?**
- **Trả lời:** Em dùng một `effect()` gọi là `formValueSyncEffect`. Nó lắng nghe sự kiện `valueChanged` từ `FormDirective`, sau đó dùng `this.value.update(...)` để cập nhật model. Việc dùng `update` của Signal giúp đảm bảo tính immutable và các component cha nhận được notification ngay lập tức.

**26. Làm sao để hiển thị tất cả lỗi validation của form qua một cái Toast thông báo (Notification)?**
- **Trả lời:** Em viết hàm `showValidationErrors()`. Nó duyệt qua tất cả `fields()` của `FormDirective`, check `control.invalid`, sau đó dùng `TranslateService` để lấy label và message lỗi, gộp thành một list string và gửi qua `NotificationService`. Người dùng sẽ thấy một popup tổng hợp tất cả lỗi cần sửa.

**27. `ControlValueAccessor` (CVA) được triển khai như thế nào cho các Custom Input?**
- **Trả lời:** Mọi component trong `@tappy/components` (như `tap-select`, `tap-datepicker`) đều implement CVA. Em cung cấp các method `writeValue`, `registerOnChange`, `registerOnTouched`. Điều này giúp `FormTemplate` có thể quản lý chúng bằng `Reactive Forms` một cách đồng nhất thông qua `key` của field.

**28. "Init Value Effect" giải quyết bài toán gì khi tạo mới dữ liệu?**
- **Trả lời:** Khi mở form ở mode `create`, em cần điền các giá trị mặc định. `initValueEffect` sẽ duyệt qua config các field, gọi hàm `resolveInitValue` (có thể là một giá trị cứng hoặc một function tính toán theo context) để set giá trị khởi tạo cho `this.value`.

**29. Tại sao bạn lại dùng `model()` signal cho input `value`?**
- **Trả lời:** Dùng `model()` cho phép Two-way binding cực kỳ mạnh mẽ và clean. Component cha có thể truyền `[(value)]="myData"`, và mọi thay đổi bên trong `FormTemplate` (do user gõ) sẽ tự động sync ngược ra `myData` mà không cần code `EventEmitter` thủ công.

**30. Phân biệt `FormDirective` và `FormTemplateComponent` trong dự án của bạn?**
- **Trả lời:** `FormDirective` chứa logic lõi về quản lý `FormGroup` và danh sách các `FormField`. `FormTemplateComponent` là lớp UI bao ngoài, chịu trách nhiệm render layout (tabs, fieldsets) và xử lý tương tác cấp cao (Submit, Cancel, Notification).

---

## 🔒 PHẦN 4: BẢO MẬT & AUTHENTICATION (SECURITY)

**31. AuthInterceptor của bạn thực hiện những nhiệm vụ gì cụ thể?**
- **Trả lời:** Nó đính kèm 3 headers quan trọng vào mọi request: `X-API-Key` (lấy từ InjectionToken `API_KEY`), `jboss_location` (để định tuyến server chi nhánh), và `Access-Control-Allow-Origin`. Nó dùng `req.clone()` để đảm bảo tính bất biến của request gốc.

**32. Tại sao hệ thống lại dùng SessionStorage thay vì LocalStorage để lưu Token?**
- **Trả lời:** Để tăng cường bảo mật cấp độ Tab. Nếu user mở link trong tab ẩn danh hoặc đóng trình duyệt, session sẽ bị hủy ngay lập tức. Đặc thù của PMS là dữ liệu lưu trú khách hàng cực kỳ nhạy cảm, nên việc giới hạn phạm vi lưu trữ là bắt buộc.

**33. Bạn xử lý việc định tuyến động qua `jboss_location` như thế nào?**
- **Trả lời:** Giá trị này được xác định ngay khi user chọn chi nhánh khách sạn lúc đăng nhập. Nó được lưu vào một `Context` tập trung. `AuthInterceptor` inject context này và đính vào header. Phía Backend sẽ dựa vào header này để truy vấn đúng Database của chi nhánh đó.

**34. Làm thế nào để Redirect user về trang Login khi Token hết hạn (401)?**
- **Trả lời:** Trong `ErrorService`, em handle `HttpErrorResponse`. Nếu `error.status === 401`, em sẽ gọi `AuthService.logout()` để xóa sạch `SessionStorage` và dùng `Router.navigate(['/login'])` kèm theo `queryParams: { returnUrl: currentPath }`.

**35. Stateless Authentication có nghĩa là gì trong ngữ cảnh dự án Delmonte?**
- **Trả lời:** Nghĩa là Server không lưu giữ bất kỳ trạng thái nào về phiên làm việc của Client (No Session ID). Mọi thông tin cần thiết để định danh và phân quyền đều nằm trong Token/API Key gửi kèm mỗi request. Điều này giúp hệ thống Monorepo dễ dàng tích hợp thêm các App mới (như App dọn phòng, App nhà hàng) mà không cần chung session.

**36. Bạn bảo vệ hệ thống khỏi Clickjacking như thế nào?**
- **Trả lời:** Ngoài việc Backend set header `X-Frame-Options: DENY`, phía Angular em kiểm tra nếu app bị load bên trong một `<iframe>` không hợp lệ, em sẽ thực hiện phá vỡ frame đó hoặc ngưng render.

**37. Injection Tokens (`API_KEY`, `BASE_URL`) giúp ích gì cho bảo mật và cấu hình?**
- **Trả lời:** Nó giúp tách biệt code logic và thông tin cấu hình nhạy cảm. Em có thể provide các giá trị khác nhau cho môi trường Dev/Staging/Prod mà không cần sửa code service, giúp tránh việc vô tình commit API Key thật lên Github.

**38. Cách bạn handle việc phân quyền (Role-based Access Control) trên UI?**
- **Trả lời:** Em sử dụng thông tin quyền hạn được lưu trong User Context. Em viết một directive `*tapPermission`. Directive này sẽ kiểm tra nếu quyền của user không khớp với yêu cầu, nó sẽ dùng `ViewContainerRef.clear()` để xóa sạch element đó khỏi DOM, chứ không chỉ là ẩn bằng CSS (`display:none`).

**39. Bạn có sử dụng HttpOnly Cookies không? Tại sao?**
- **Trả lời:** Với hệ thống PMS hiện tại, team đang dùng Header-based (X-API-Key) vì tính linh hoạt cho Monorepo. Tuy nhiên, em có đề xuất chuyển sang HttpOnly Cookies cho các phiên bản sau để ngăn chặn triệt để việc Javascript (XSS) có thể đọc được Token.

**40. Cơ chế "Stateless" có gây khó khăn gì khi làm Real-time (Socket) không?**
- **Trả lời:** Có một chút rắc rối vì Socket cần bắt tay (handshake). Em đã giải quyết bằng cách gửi Token qua tham số query trong lúc khởi tạo kết nối Socket lần đầu, sau đó Server sẽ xác thực và duy trì kết nối đó.

---

## 🚀 PHẦN 5: REACTIVE LOGIC & ANGULAR v19 (MODERN PATTERNS)

**41. Bạn ứng dụng `computed()` signal vào bài toán thực tế nào?**
- **Trả lời:** Em dùng rất nhiều. Ví dụ: `resolvedFields` trong FormTemplate. Nó tự động tính toán lại danh sách field cần hiển thị mỗi khi `fields()` (cấu hình) hoặc `context()` (trạng thái form) thay đổi. Nhờ `computed`, việc tính toán này được cache và chỉ chạy lại khi cần thiết.

**42. `input()` signal (v17.1+) thay thế `@Input()` truyền thống như thế nào?**
- **Trả lời:** `input()` trả về một Signal. Thay vì dùng `ngOnChanges` cực kỳ tốn hiệu năng để theo dõi biến đổi, em dùng `effect()` hoặc `computed()` để phản ứng lại giá trị mới. Code trở nên tường minh (declarative) hơn rất nhiều.

**43. Sự khác biệt giữa `toSignal()` và `toObservable()`? Bạn dùng chúng ở đâu?**
- **Trả lời:** `toSignal()` em dùng khi muốn biến một luồng API (Observable) thành một biến Signal để hiển thị lên template mà không cần `| async`. `toObservable()` em dùng khi muốn tận dụng các toán tử RxJS (như `debounceTime`) trên một Signal (như input tìm kiếm).

**44. Tại sao bạn lại chọn kiến trúc "Facade Pattern" cho các module lớn?**
- **Trả lời:** Với module Timeline, em có `TimelineFacadeService`. Nó là điểm tiếp xúc duy nhất giữa Component và logic phức tạp. Nó ẩn đi việc tương tác với `DataSource`, `InteractionService`, và `UnitStrategy`. Component chỉ việc gọi Facade và lắng nghe các Signals kết quả.

**45. Cách xử lý lỗi lồng nhau trong RxJS (ví dụ: Call API 1 xong lấy data call API 2)?**
- **Trả lời:** Em tuyệt đối không "nest subscribe". Em sử dụng `switchMap` hoặc `concatMap`. Ví dụ: lấy danh sách Hotel trước, sau đó dùng `switchMap` để lấy danh sách Room tương ứng với Hotel đó.

**46. Bạn giải quyết bài toán "Race Condition" khi call API liên tục như thế nào?**
- **Trả lời:** `switchMap` là "cứu tinh". Mỗi khi một request mới được phát ra, `switchMap` sẽ tự động unsubscribe (hủy bỏ) request cũ đang chạy. Điều này cực kỳ quan trọng khi user click nhanh giữa các tab "Phòng đơn", "Phòng đôi".

**47. `takeUntilDestroyed()` giúp bạn quản lý memory leak như thế nào?**
- **Trả lời:** Đây là helper mới của Angular. Em không cần tạo `Subject` và `ngOnDestroy` rườm rà nữa. Chỉ cần `.pipe(takeUntilDestroyed())`, Angular sẽ tự động dọn dẹp subscription khi component bị destroy.

**48. Logic xử lý "Retry" API khi gặp lỗi mạng chập chờn?**
- **Trả lời:** Em dùng toán tử `retry()`. Em cấu hình retry 3 lần, mỗi lần cách nhau 2 giây (exponential backoff) trước khi chính thức báo lỗi lên UI qua `status: 'error'`.

**49. Cách bạn biến đổi dữ liệu (Transforming Data) hiệu quả nhất trong Angular?**
- **Trả lời:** Em kết hợp `Adapter Pattern` ở tầng Data và `Pipe` (Pure) ở tầng UI. Data về sẽ được Adapter chuẩn hóa, còn các việc nhỏ nháy như format tiền, format ngày thì để Pipe lo để tối ưu hiệu năng render.

**50. Bạn làm gì nếu một Signal Effect chạy quá nhiều lần không cần thiết?**
- **Trả lời:** Em dùng `untracked()` cho các phần dữ liệu mà em muốn đọc giá trị nhưng không muốn nó trở thành dependency của effect. Hoặc em sẽ tách nhỏ effect đó ra để mỗi effect chỉ chịu trách nhiệm cho một nhóm logic nhỏ.

---

## 🛠️ PHẦN 6: MONOREPO, TOOLING & WORKFLOW

**51. Tại sao dự án của bạn lại chia thành nhiều packages (`main`, `tappy`, `core`)?**
- **Trả lời:** Để phục vụ bài toán mở rộng (Scalability). Package `tappy` là UI Library dùng chung, `core` chứa logic cốt lõi. Khi Mobio muốn build thêm một App cho mobile bằng Angular, em có thể tái sử dụng ngay 80% code từ `tappy` và `core` mà không cần copy.

**52. Pnpm workspace giúp bạn quản lý dependencies như thế nào?**
- **Trả lời:** Nó cho phép em định nghĩa các local dependencies dạng `workspace:*`. Ví dụ trong app `pms`, em khai báo dùng `@tappy/core: workspace:*`. Pnpm sẽ tự động link code từ package core sang mà không cần build hay publish lên npm registry.

**53. Bản chất của "Public API" (`public_api.ts`) trong thư viện UI của bạn?**
- **Trả lời:** Nó là một cái "phễu". Chỉ những component, service nào em khai báo export trong đó thì các App ngoài mới thấy và dùng được. Điều này giúp em có thể sửa thoải mái các logic bên trong (internal) mà không sợ làm "gãy" code của các team khác đang dùng thư viện.

**54. Bạn sử dụng Husky và Commitlint để làm gì?**
- **Trả lời:** Để ép cả team vào kỷ luật. Header commit phải đúng format (`type: message`). Husky sẽ chạy `lint` và `test` trước khi cho phép `git push`. Điều này giúp đảm bảo code trên branch chính luôn "sạch" và pass tất cả các test cơ bản.

**55. Quy trình CI/CD của bạn (với tư cách Frontend) có gì đặc biệt?**
- **Trả lời:** Mỗi khi em đẩy code, hệ thống CI sẽ tự động build các package theo đúng thứ tự phụ thuộc (dependencies graph), sau đó chạy bộ test của library trước khi build app chính. Nó giúp phát hiện lỗi "vỡ layout" sớm nếu em lỡ sửa code trong package dùng chung.

**56. Bạn quản lý Versioning cho thư viện `@tappy` như thế nào?**
- **Trả lời:** Em dùng `changesets`. Mỗi PR sẽ đi kèm một file changeset mô tả thay đổi (patch, minor, major). Khi merge, hệ thống tự động bump version và generate changelog, giúp các thành viên khác biết có gì mới để cập nhật.

**57. Cách bạn giải quyết mâu thuẫn Version của các thư viện bên thứ 3 (ví dụ: `lodash`) trong Monorepo?**
- **Trả lời:** Em dùng tính năng `pnpm.overrides` để ép toàn bộ Monorepo chỉ dùng duy nhất 1 version của lodash. Điều này tránh việc bundle bị phình to do chứa nhiều bản copy của cùng một thư viện.

**58. Lợi ích của việc dùng GitHub Actions trong workflow của bạn?**
- **Trả lời:** Em tự viết các workflow để tự động deploy bản "Preview" mỗi khi có Pull Request. Designer và PO có thể vào check UI ngay lập tức mà không cần đợi em deploy lên môi trường Staging.

**59. Bạn cấu hình Esbuild cho Angular như thế nào?**
- **Trả lời:** Từ Angular 17, em chuyển qua `application` builder (dùng Esbuild & Vite). Tốc độ build nhanh gấp 5-10 lần so với Webpack cũ. Em cấu hình trong `angular.json` để tận dụng tính năng build song song của Monorepo.

**60. Cách bạn viết tài liệu (Documentation) cho bộ thư viện UI?**
- **Trả lời:** Em dùng chính các app "Demo" trong Monorepo. Mỗi component trong `tappy` sẽ có một trang demo đi kèm mô tả các `Input/Output` và code mẫu (`Live Coding`), giúp các developer khác vào là biết cách dùng ngay.

---

## 🤝 PHẦN 7: KỸ NĂNG MỀM & BÀI TOÁN THỰC TẾ TẠI MOBIO

**61. Bạn làm gì khi nhận được một thiết kế Figma cực kỳ phức tạp từ Designer?**
- **Trả lời:** Em sẽ không bắt tay vào code ngay. Em sẽ phân tích và phản hồi (feedback) lại designer: "Chỗ này có thể dùng component X có sẵn", "Chỗ kia nếu làm vậy sẽ ảnh hưởng tới hiệu năng scroll". Em luôn hướng tới sự cân bằng giữa tính thẩm mỹ và tính khả thi kỹ thuật.

**62. Một ví dụ về việc bạn tối ưu code sau khi nhận Review từ đồng nghiệp?**
- **Trả lời:** Có lần em viết một logic tính toán giá phòng lồng nhau 3 vòng lặp. Đồng nghiệp đã gợi ý em chuyển sang dùng `Map` để giảm độ phức tạp từ `O(n^3)` xuống `O(n)`. Em đã tiếp thu, sửa lại và từ đó luôn chú ý tới hiệu suất thuật toán thay vì chỉ code cho chạy được.

**63. Bạn xử lý áp lực Deadline khi dự án Delmonte đến ngày bàn giao như thế nào?**
- **Trả lời:** Em áp dụng quy tắc "MVP trước, Polish sau". Em sẽ tập trung hoàn thiện luồng nghiệp vụ chính (Happy Path) để đảm bảo hệ thống chạy được. Các hiệu ứng animation hay tối ưu nhỏ nhặt sẽ được đưa vào TODO list để làm ngay sau đó.

**64. Tại sao một Middle Developer như bạn lại cần quan tâm đến UX (Trải nghiệm người dùng)?**
- **Trả lời:** Vì code dù tốt đến đâu mà user thấy khó dùng thì vẫn là thất bại. Ví dụ: Em chủ động thêm `loading skeleton` và `disable submit button` khi đang call API để tránh user click nhầm nhiều lần. Đó là những chi tiết nhỏ nhưng thể hiện sự chuyên nghiệp.

**65. Bạn đã bao giờ từ chối một yêu cầu tính năng (Feature Request) từ PO chưa?**
- **Trả lời:** Có, nếu tính năng đó làm phá vỡ kiến trúc Core của Monorepo. Thay vì từ chối thẳng, em sẽ đề xuất một giải pháp "Workaround" hoặc đề xuất tách nó thành một module độc lập để không ảnh hưởng tới các App khác.

**66. Cách bạn mentor hoặc hỗ trợ các thành viên Junior trong team?**
- **Trả lời:** Em không code hộ. Em sẽ chỉ lỗi, giải thích *tại sao* lỗi đó xảy ra và cung cấp các keyword/link tài liệu để bạn tự tìm hiểu. Em cũng thường xuyên tổ chức các buổi "Quick knowledge sharing" 15 phút về các tính năng mới của Angular 19.

**67. Bạn hiểu gì về văn hóa Agile/Scrum và áp dụng nó như thế nào?**
- **Trả lời:** Team em chạy Sprint 2 tuần. Em luôn chủ động cập nhật trạng thái Task trên Jira hàng ngày. Trong buổi Retro, em luôn thẳng thắn đóng góp các vấn đề về quy trình build/deploy đang chậm để team cùng cải tiến.

**68. Nếu được chọn một thứ để cải thiện trong ProdDelmonteUI bây giờ, bạn sẽ chọn gì?**
- **Trả lời:** Em sẽ tăng độ bao phủ của `Unit Test` lên trên 80%. Hiện tại hệ thống đang chạy rất tốt nhưng với độ phức tạp ngày càng tăng, unit test sẽ là "bảo hiểm" tốt nhất để team tự tin refactor code trong tương lai.

**69. Bạn nghĩ thách thức lớn nhất khi làm việc tại Mobio là gì?**
- **Trả lời:** Đó là việc xử lý khối lượng dữ liệu khổng lồ (Big Data) trên giao diện web. Với trải nghiệm tối ưu Timeline cho hàng vạn booking, em tự tin mình có phương pháp để "trị" các bài toán render nặng của Mobio.

**70. Bạn định vị mình ở đâu trong 2 năm tới?**
- **Trả lời:** Em muốn trở thành một **Senior/Technical Lead** am hiểu sâu về Frontend Infrastructure. Tham gia vào việc xây dựng các bộ Design System chuẩn chỉnh và tối ưu hóa workflow phát triển cho các dự án quy mô lớn.

---
*Hy vọng bộ câu hỏi này sẽ giúp bạn tự tin chinh phục Mobio!*
