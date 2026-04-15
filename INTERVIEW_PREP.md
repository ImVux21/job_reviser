# 🎯 TỔNG HỢP 70 CÂU HỎI & TRẢ LỜI PHỎNG VẤN (TÂM ĐIỂM PRODDELMONTEUI)

> Tài liệu này được thiết kế để bạn "bảo vệ" những gì đã viết trong CV, tập trung sâu vào hệ thống **ProdDelmonteUI** và vị trí Middle Angular Developer tại Mobio.

---

## 🏛️ PHẦN 1: ANGULAR CORE & MODERN ARCHITECTURE (12 CÂU)

**1. Hệ thống ProdDelmonteUI sử dụng Angular 19, bạn thấy tính năng "Zoneless" có lợi ích gì thực tế?**
- **Trả lời:** Trong dự án, Zoneless giúp loại bỏ overhead của Zone.js. Với bảng Timeline chứa hàng ngàn booking tiles, việc không phải "monkey patch" mọi asyn task giúp giảm giật lag đáng kể. Ta dùng Signals để báo cho Angular biết chính xác phần nào cần render lại.

**2. Tại sao bạn lại chọn Standalone Components cho bộ thư viện `@tappy/components`?**
- **Trả lời:** Standalone giúp các package trong Monorepo độc lập hơn. Khi app PMS cần dùng một `tap-button`, nó không phải import cả một `SharedModule` khổng lồ, giúp Bundle Size giảm đáng kể và tránh lỗi Circular Dependency.

**3. Hàm `inject()` được ứng dụng như thế nào trong ProdDelmonteUI?**
- **Trả lời:** Em dùng `inject()` trong các class field để code gọn hơn. Đặc biệt là dùng trong các Functional Guards hoặc để lấy `BASE_URL`, `API_KEY` từ Injection Tokens mà không cần khai báo rườm rà trong constructor.

**4. Signal-based Inputs (`input()`) giúp gì cho việc tối ưu component của bạn?**
- **Trả lời:** Thay vì dùng `@Input` và `ngOnChanges`, em dùng `input()`. Nó trả về một Signal nên em có thể dùng `computed()` để biến đổi dữ liệu (ví dụ: format date của booking) một cách reactive và hiệu quả hơn.

**5. Khác biệt giữa `signal()` và `linkedSignal()` (v19)?**
- **Trả lời:** `linkedSignal` rất hữu ích khi em muốn một signal tự động reset giá trị khi một signal khác thay đổi. Ví dụ: Khi đổi "Hotel ID", danh sách "Room Selection" cần phải được reset về rỗng một cách đồng bộ.

**6. Tại sao dự án ProdDelmonteUI lại ép buộc 100% OnPush Change Detection?**
- **Trả lời:** Vì đây là hệ thống Enterprise quản lý nhiều dữ liệu. Nếu để Change Detection mặc định, mọi cử động chuột hay scroll đều khiến toàn bộ cây component bị check lại. OnPush đảm bảo chỉ khi data ref thay đổi thì component mới vẽ lại.

**7. Bạn quản lý Dependency Injection trong Monorepo như thế nào?**
- **Trả lời:** Em sử dụng `providedIn: 'root'` cho các core services. Với các service đặc thù cho từng app (như logic đặt phòng riêng), em provide ở cấp app hoặc cấp component để đảm bảo tính đóng gói (encapsulation).

**8. Lợi ích của `resource()` và `httpResource()` trong Angular 19?**
- **Trả lời:** Nó giúp việc gọi API trở nên cực kỳ đơn giản và "reacitve" hơn. App chỉ cần truyền một signal chứa query, `httpResource` tự động trigger request và trả về signals chứa `data`, `isLoading`, `error`. Em đã áp dụng thử nghiệm cho các dashboard đơn giản.

**9. `afterRender` và `afterNextRender` giải quyết bài toán gì cho dự án của bạn?**
- **Trả lời:** Khi cần thao tác trực tiếp với DOM (ví dụ: đo đạc kích thước bảng Timeline để vẽ Grid), em dùng `afterNextRender` để đảm bảo code chỉ chạy ở Browser, tránh lỗi khi chạy SSR hoặc Prerendering.

**10. View Transition API trong Angular 17+ bạn có ứng dụng không?**
- **Trả lời:** Có, em dùng nó để tạo hiệu ứng chuyển cảnh mượt mà khi người dùng di chuyển giữa màn hình "Danh sách phòng" và "Chi tiết đặt phòng", giúp cảm giác ứng dụng giống Native App hơn.

**11. Server-Side Rendering (SSR) có được áp dụng trong ProdDelmonteUI không?**
- **Trả lời:** Hiện tại chủ yếu là Client-side render vì là công cụ quản trị nội bộ. Tuy nhiên, kiến trúc Monorepo đã sẵn sàng cho `@angular/ssr` nếu cần tối ưu SEO cho trang login/landing.

**12. Làm sao để đảm bảo Component trong thư viện `@tappy` có tính tái sử dụng cao?**
- **Trả lời:** Tuân thủ quy tắc: Nhận data qua `input()`, báo sự kiện qua `output()`, không chứa logic nghiệp vụ (Dumb Component). Sử dụng Content Projection (`ng-content`) để cho phép bên ngoài tùy biến layout.

---

## ⚡ PHẦN 2: RXJS & REACTIVE DATA FLOW (10 CÂU)

**13. Trong ProdDelmonteUI, làm sao bạn xử lý việc người dùng gõ tìm kiếm booking liên tục?**
- **Trả lời:** Em sử dụng `searchSignal` kết hợp `toObservable()`, sau đó dùng toán tử `debounceTime(300)` và `distinctUntilChanged()`, cuối cùng là `switchMap()` để gọi API. Điều này đảm bảo chỉ request cuối cùng được thực thi.

**14. Giải thích cách bạn sử dụng `DataBusService` để làm Caching Layer?**
- **Trả lời:** Khi một API được gọi, `DataBusService` sẽ lưu kết quả vào một `Map` (Subject hoặc Signal). Nếu các component khác yêu cầu cùng một dữ liệu đó trong một khoảng thời gian ngắn, Service sẽ trả về dữ liệu cache thay vì gọi lại server, giúp giảm 40% request thừa.

**15. Bạn xử lý lỗi API (Error Handling) tập trung như thế nào?**
- **Trả lời:** Em viết một `ErrorService` và kết hợp với `HttpInterceptor`. Mọi lỗi 4xx, 5xx đều được catch, hiển thị thông báo qua `tap-toast` và log lỗi về hệ thống tập trung.

**16. Khi nào bạn bắt buộc phải dùng RxJS thay vì Signals trong dự án này?**
- **Trả lời:** Khi cần xử lý các bài toán thời gian (timing) như `interval` cập nhật trạng thái phòng 30s một lần, khi cần `retry` request gặp lỗi, hoặc khi cần kết hợp nhiều luồng dữ liệu async phức tạp bằng `combineLatest`.

**17. `shareReplay(1)` có ý nghĩa gì trong API service của bạn?**
- **Trả lời:** Giúp "multicast" một luồng dữ liệu. Ví dụ, thông tin "Cấu hình khách sạn" chỉ cần gọi 1 lần, các component khác subcribe sau đó sẽ nhận được ngay kết quả cuối cùng mà không trigger thêm HTTP request.

**18. Làm sao để ngăn chặn Memory Leak khi sử dụng RxJS trong hệ thống PMS?**
- **Trả lời:** Em sử dụng `takeUntilDestroyed()` từ `rxjs-interop`. Nó tự động unsubscribe khi component bị destroy, rất an toàn và sạch sẽ hơn việc lưu subscription thủ công.

**19. Sự khác biệt giữa `Subject` và `Signal` trong việc phát thông báo (Event)?**
- **Trả lời:** `Subject` phát sự kiện cho các subscriber tại thời điểm xảy ra. `Signal` giữ trạng thái. Với các sự kiện như "Đã thanh toán thành công", em dùng `Subject`. Với trạng thái "Đang loading", em dùng `Signal`.

**20. Bạn dùng toán tử nào để gộp kết quả từ nhiều API (ví dụ: Thông tin phòng + Thông tin khách)?**
- **Trả lời:** Em dùng `forkJoin` nếu muốn đợi tất cả xong mới xử lý, hoặc `combineLatest` nếu muốn UI update ngay khi bất kỳ nguồn nào có data (kết hợp với loading skeleton).

**21. Làm thế nào để handle việc Token hết hạn giữa lúc đang call nhiều API song song?**
- **Trả lời:** Trong Interceptor, em dùng một biến flag `isRefreshing` và một `Subject` để "xếp hàng" các request đang chờ. Khi có token mới, em phát tín hiệu để các request đó chạy tiếp qua `switchMap`.

**22. Higher-order Observable nào em dùng nhiều nhất? Tại sao?**
- **Trả lời:** `switchMap`. Vì trong PMS, người dùng thường đổi hotel hoặc ngày tháng liên tục, em cần hủy bỏ các request của hotel/ngày cũ ngay lập tức để tránh sai lệch dữ liệu trên UI.

---

## 🛡️ PHẦN 3: BẢO MẬT & AUTHENTICATION (JWT/API KEYS) (10 CÂU)

**23. Mô tả luồng xử lý Authentication (Xác thực) trong hệ thống ProdDelmonteUI?**
- **Trả lời:** Hệ thống sử dụng cơ chế Stateless. Sau khi login, Client nhận về một Token. Em sử dụng `AuthInterceptor` để tự động đính kèm `X-API-Key` và các thông tin định danh như `jboss_location` vào header của mọi request đi tới Server.

**24. Tại sao lại dùng Interceptor thay vì đính kèm header thủ công trong mỗi Service?**
- **Trả lời:** Để đảm bảo tính tập trung (Centralized). Nếu sau này Server đổi key hay cần thêm header mới (ví dụ: `App-Version`), em chỉ cần sửa duy nhất ở `AuthInterceptor`.

**25. Bạn lưu trữ Token/API Key ở đâu để đảm bảo an toàn?**
- **Trả lời:** Em lưu vào `SessionStorage` để đảm bảo khi đóng tab/trình duyệt dữ liệu sẽ bị xóa, tránh rủi ro nếu người dùng dùng máy công cộng. Với các thông tin nhạy cảm hơn, em kiến nghị dùng `HttpOnly Cookie`.

**26. `X-API-Key` trong interceptor của bạn được lấy từ đâu?**
- **Trả lời:** Nó được inject vào thông qua `API_KEY` token (InjectionToken). Giá trị này thường được cung cấp từ tầng cấu hình môi trường (Environment) của App chính khi khởi tạo Monorepo.

**27. Làm thế nào để xử lý lỗi 401 (Unauthorized) một cách thông minh?**
- **Trả lời:** Trong Interceptor, em check nếu `error.status === 401`, em sẽ tự động logout người dùng hoặc gọi xóa session và điều hướng về trang `/login` kèm theo `returnUrl`.

**28. XSS (Cross-Site Scripting) là gì và dự án của bạn phòng chống nó thế nào?**
- **Trả lời:** XSS là tấn công chèn mã độc vào web. Angular mặc định sanitize dữ liệu. Tuy nhiên, em luôn nhắc team tránh dùng `innerHTML`. Nếu bắt buộc phải dùng (ví dụ: hiển thị nội dung email khách hàng), em dùng `DomSanitizer` để lọc mã độc.

**29. CSRF (Cross-Site Request Forgery) bạn xử lý thế nào?**
- **Trả lời:** Vì dùng Token đính kèm ở Header (`X-API-Key`), nên hệ thống mặc định đã chống được CSRF (do Browser không tự động đính kèm custom headers khi hacker giả mạo form submit).

**30. Tại sao hệ thống lại cần gửi thêm header `jboss_location`?**
- **Trả lời:** Đây là đặc thù của kiến trúc Microservices/Distributed bên Backend. Header này giúp Gateway hoặc Load Balancer biết cần định tuyến request đến cụm server vật lý nào tương ứng với chi nhánh khách sạn đó.

**31. Stateless Authentication có ưu điểm gì cho hệ thống PMS Monorepo?**
- **Trả lời:** Giúp Server không phải lưu trữ Session, dễ dàng mở rộng (Scalability). Dù App là PMS hay POS, chỉ cần có Key/Token hợp lệ là có thể truy cập tài nguyên, rất phù hợp với mô hình Monorepo nhiều apps.

**32. Làm sao để ẩn/hiện các nút (Ví dụ: Nút Xóa Booking) dựa trên quyền hạn của JWT/Token?**
- **Trả lời:** Em viết một `AuthService` để decode thông tin quyền hạn (Permissions) và tạo một Structural Directive `*tapPermission="'DELETE_BOOKING'"` để xóa bỏ element khỏi DOM nếu User không có quyền.

---

## 🚀 PHẦN 4: PERFORMANCE & LARGE DATA HANDLING (10 CÂU)

**33. Bí quyết để bảng Timeline trong ProdDelmonteUI đạt mức 60 FPS là gì?**
- **Trả lời:** Ngoài OnPush, em sử dụng kỹ thuật "Canvas Rendering" hoặc "Virtual Scrolling" cho các khu vực quá dày đặc dữ liệu. Đặc biệt là dùng `trackBy` để Angular không vẽ lại các ô booking cũ khi chỉ có 1 ô mới được thêm vào.

**34. Virtual Scrolling trong dự án của bạn hoạt động như thế nào?**
- **Trả lời:** Em dùng `CdkVirtualScrollViewport`. Nó chỉ render khoảng 20-30 dòng dữ liệu nằm trong tầm mắt người dùng. Khi scroll, nó tái sử dụng các thẻ HTML đó và chỉ thay đổi dữ liệu bên trong.

**35. Khi nào bạn sử dụng `ChangeDetectorRef.detectChanges()`?**
- **Trả lời:** Hầu như không dùng nếu đã có Signals. Tuy nhiên, đôi khi cần update UI ngay lập tức sau một thao tác DOM bên ngoài vùng quản lý của Angular (như library bên thứ 3), em sẽ gọi nó.

**36. Web Workers được ứng dụng như thế nào cho bài toán tính toán giá phòng?**
- **Trả lời:** Nếu việc tính toán giá phức tạp (nhiều quy tắc thuế, phí, tour...) gây treo UI, em đẩy logic đó vào Web Worker để chạy ở thread riêng, trả kết quả về cho UI thread hiển thị.

**37. Lazy Loading và Preloading trong ProdDelmonteUI được thiết kế ra sao?**
- **Trả lời:** Các module lớn như "Kế toán", "Báo cáo" được Lazy Load. Sau khi trang chính load xong, em bật `QuicklinkStrategy` để tải trước các trang mà người dùng có khả năng click vào tiếp theo.

**38. Tối ưu hóa Bundle Size cho bộ thư viện `@tappy` bằng cách nào?**
- **Trả lời:** Đảm bảo code mang tính "Tree-shakable". Không import cả thư viện `lodash` mà chỉ import cụ thể function cần dùng. Sử dụng Esbuild để minify code hiệu quả nhất.

**39. Bạn xử lý các mảng dữ liệu lớn (Array transformation) như thế nào cho mượt?**
- **Trả lời:** Sử dụng các phương thức immutable (`map`, `filter`) kết hợp với `computed()` signal để kết quả được cache và chỉ tính toán lại khi mảng gốc thực sự thay đổi.

**40. `trackBy` quan trọng thế nào trong dự án PMS?**
- **Trả lời:** Rất quan trọng! Nếu không có `trackBy`, mỗi lần update trạng thái một phòng (ví dụ: chuyển từ Trống sang Có khách), Angular sẽ xóa trắng và vẽ lại cả nghìn ô phòng, gây giật màn hình.

**41. Bạn làm gì khi Angular DevTools báo Change Detection chạy quá nhiều?**
- **Trả lời:** Em sẽ kiểm tra xem có hàm nào đang được gọi trực tiếp trong template không (ví dụ: `{{ getPrice() }}`). Nếu có, em chuyển nó sang `computed()` signal hoặc `Pipe`.

**42. Lợi ích của việc dùng CSS Grid/Flexbox so với Table truyền thống cho Timeline?**
- **Trả lời:** GPU có thể tối ưu việc render Grid/Flexbox tốt hơn, giúp việc scroll ngang/dọc bảng Room-Map mượt mà hơn nhiều so với `<table>` nguyên bản.

---

## 📝 PHẦN 5: FORMS, DIRECTIVES & PIPES (10 CÂU)

**43. Tại sao ProdDelmonteUI lại ưu tiên Dynamic Forms (Config-driven)?**
- **Trả lời:** Vì PMS có hàng trăm loại form khai báo (thông tin khách, thông tin phòng, dịch vụ đi kèm). Việc xây dựng một Component `<tap-form-template>` nhận JSON config giúp team giảm 50% thời gian code UI và đảm bảo tính nhất quán.

**44. `ControlValueAccessor` (CVA) được ứng dụng ở đâu trong thư viện Tappy?**
- **Trả lời:** Em ứng dụng cho tất cả các custom inputs như `tap-select`, `tap-datepicker`. Điều này cho phép các app sử dụng chúng với `Reactive Forms` (formControlName) một cách tự nhiên.

**45. Làm sao để validate một Form phức tạp (ví dụ: Ngày trả phòng phải sau ngày nhận phòng)?**
- **Trả lời:** Em viết một `ValidatorFunction` ở cấp `FormGroup` (Cross-field validation), lấy giá trị của cả 2 field để so sánh và trả về lỗi nếu không hợp lệ.

**46. `HostBinding` và `HostListener` bạn dùng khi nào trong `@tappy/components`?**
- **Trả lời:** Dùng để tạo các hiệu ứng tương tác. Ví dụ: `HostBinding('class.active')` để tự động highlight component, `HostListener('keydown')` để điều hướng bằng phím mũi tên trong menu.

**47. Lợi ích của "Pure Pipe" trong việc format tiền tệ ở PMS?**
- **Trả lời:** `tapCurrency` pipe là pure. Khi người dùng scroll bảng kê, hàng nghìn ô tiền không bị tính toán lại liên tục, chỉ khi giá trị tiền tại ô đó thay đổi thì Pipe mới chạy.

**48. Directive Composition API giúp bạn tái sử dụng code thế nào?**
- **Trả lời:** Em có các directive như `TooltipDirective`, `ClickOutsideDirective`. Thay vì kế thừa phức tạp, em dùng `hostDirectives` để gán chúng vào bất kỳ component nào một cách linh hoạt.

**49. Cách xử lý khi một Form cực lớn bị chậm khi gõ?**
- **Trả lời:** Sử dụng `updateOn: 'blur'` thay vì 'change' mặc định để Angular chỉ validate khi người dùng gõ xong và chuyển sang field khác.

**50. Làm sao để truyền dữ liệu từ Directive về Component chứa nó?**
- **Trả lời:** Em sử dụng `@Output` (hoặc `output()`) trong Directive, hoặc inject chính Component đó vào constructor của Directive để gọi method trực tiếp.

**51. AsyncPipe quan trọng thế nào trước khi có Signals?**
- **Trả lời:** Nó giúp tự động subscribe/unsubscribe Observable và trigger Change Detection. Hiện tại em dần chuyển sang dùng Signals để code tường minh hơn.

**52. Bạn tạo một Custom Validator để check Format mã Voucher khách sạn thế nào?**
- **Trả lời:** Viết một function nhận vào `AbstractControl`, dùng Regex để kiểm tra và trả về `ValidationErrors | null`.

---

## 🏗️ PHẦN 6: ARCHITECTURE, MONOREPO & WORKFLOW (10 CÂU)

**53. Tại sao bạn dùng `pnpm` thay vì `npm` hay `yarn` cho Monorepo này?**
- **Trả lời:** Tốc độ và tiết kiệm bộ nhớ. Với Monorepo chứa hàng chục package, pnpm chỉ lưu một bản copy thư viện duy nhất cho toàn bộ máy, giúp cài đặt dependencies nhanh gấp 3 lần.

**54. Bạn giải quyết bài toán "Dependency Hell" giữa các app trong Monorepo thế nào?**
- **Trả lời:** Sử dụng `peerDependencies` cho thư viện `@tappy` và ép buộc các app phải dùng chung một phiên bản Angular bằng cách khai báo version ở Root `package.json`.

**55. Quy chuẩn commit (Husky, Commitlint) trong dự án giúp ích gì cho bạn?**
- **Trả lời:** Đảm bảo mọi commit đều phải theo chuẩn (ví dụ: `feat: add room table`). Nó giúp việc tự động sinh `CHANGELOG.md` và trace lại lịch sử sửa lỗi cực kỳ dễ dàng.

**56. Bạn quản lý trạng thái global (State Management) như thế nào mà không dùng NGRX?**
- **Trả lời:** Em dùng **Signal-based Services** kết hợp Facade Pattern. Với em, NGRX quá nặng nề cho các app quản trị. Một Service chứa `signal` là đủ để quản lý state bền vững và dễ hiểu.

**57. "Config-driven Architecture" trong dự án Delmonte có nhược điểm gì không?**
- **Trả lời:** Nhược điểm là việc debug sẽ khó hơn nếu JSON config quá phức tạp. Cách khắc phục là em viết các JSON Schema và dùng TypeScript Interfaces để validate config ngay lúc code.

**58. Bạn tổ chức thư mục (Folder Structure) trong Monorepo như thế nào?**
- **Trả lời:** Chia làm `packages/main` (các app chính), `packages/tappy` (UI library), và `packages/core` (logic dùng chung như API, Auth, Utils).

**59. Khi nào bạn quyết định tách một logic ra thành một `package` riêng thay vì để trong app?**
- **Trả lời:** Khi logic đó có khả năng được dùng ở 2 app trở lên (ví dụ: logic tính toán Refund tiền phòng dùng chung cho cả PMS và POS).

**60. Bạn sử dụng `nx` hay chỉ dùng `pnpm workspace` thuần? Tại sao?**
- **Trả lời:** Hiện tại chủ yếu dùng `pnpm workspace` kết hợp các scripts tự viết. Nếu dự án phình to hơn, em sẽ đề xuất dùng `Nx` để tận dụng tính năng build cache và graph analysis.

**61. Làm sao để đảm bảo "Public API" của thư viện `@tappy` luôn sạch sẽ?**
- **Trả lời:** Sử dụng file `public_api.ts`. Chỉ export những gì cần thiết cho bên ngoài dùng, các logic xử lý bên trong (internal) sẽ được giấu kín để tránh việc app phụ thuộc quá sâu vào chi tiết triển khai.

**62. Cách bạn quản lý các Assets (ảnh, icon) dùng chung cho toàn bộ Monorepo?**
- **Trả lời:** Em tạo một package `themes` hoặc `assets`, sau đó config `assets` trong `angular.json` để copy từ thư mục chung đó vào từng app khi build.

---

## 🤝 PHẦN 7: INTERVIEW ATTITUDE, AI & SOFT SKILLS (8 CÂU)

**63. Bạn dùng AI Tools (Cursor/Copilot) để làm gì mà vẫn đảm bảo mình không bị "lười"?**
- **Trả lời:** Em dùng AI để sinh Unit Test nhanh, viết boilerplate code (như interfaces, mock data). Tuy nhiên, em luôn là người quyết định kiến trúc cuối cùng. AI giúp em rảnh tay để tập trung vào các bài toán logic khó hơn.

**64. Quy trình Code Review của bạn tại team là gì?**
- **Trả lời:** Sau khi tạo PR, em sẽ check: 1. Code có chạy đúng spec không? 2. Có dùng `OnPush` chưa? 3. Có rò rỉ RxJS không? 4. Variable naming có rõ nghĩa không? Em luôn review với tâm thế xây dựng, không chỉ trích.

**65. Bạn làm gì khi có xung đột kỹ thuật (Technical Conflict) với đồng nghiệp?**
- **Trả lời:** Em sẽ đề xuất làm một buổi "Mini Tech-Talk" hoặc Demo nhỏ. Dữ liệu và Performance thực tế sẽ là câu trả lời tốt nhất. Nếu vẫn không đồng nhất, em sẽ tuân theo quyết định của Lead nhưng vẫn bảo lưu ý kiến phục vụ cho việc cải tiến sau này.

**66. Tại sao bạn lại muốn gia nhập Mobio? Bạn biết gì về sản phẩm CDP của chúng tôi?**
- **Trả lời:** Em ấn tượng với việc Mobio xử lý dữ liệu khách hàng khổng lồ và cá nhân hóa marketing. Với kinh nghiệm tối ưu Angular cho Large Datasets và Monorepo, em tin mình sẽ giúp Mobio nâng tầm trải nghiệm mượt mà cho các sản phẩm SaaS của công ty.

**67. Khó khăn lớn nhất bạn từng gặp trong dự án ProdDelmonteUI là gì?**
- **Trả lời:** Đó là lúc chuyển đổi từ kiến trúc cũ sang Signals/OnPush trong khi app vẫn đang phải chạy. Em đã phải làm việc này cuốn chiếu từng module, viết bổ sung unit test để đảm bảo không bị regression lỗi.

**68. Bạn học kiến thức Angular mới từ đâu?**
- **Trả lời:** Em theo dõi blog chính thức của Angular (angular.dev), các kênh YouTube như *Angular University* và các repository mẫu của các GDE (Google Developer Experts).

**69. Bạn có sẵn sàng làm việc với các framework khác ngoài Angular không?**
- **Trả lời:** Có, nền tảng của em là TypeScript và Logic lập trình. Dù em đam mê Angular nhất, nhưng nếu dự án yêu cầu React hay NestJS (Backend), em hoàn toàn có thể thích nghi nhanh nhờ tư duy hệ thống sẵn có.

**70. Bạn kỳ vọng gì ở một môi trường làm việc lý tưởng?**
- **Trả lời:** Một môi trường coi trọng chất lượng code (Code Quality), cởi mở với công nghệ mới (như AI, Signals) và có lộ trình thăng tiến kỹ thuật rõ ràng.

---
*Chúc bạn tự tin tỏa sáng tại Mobio!*
