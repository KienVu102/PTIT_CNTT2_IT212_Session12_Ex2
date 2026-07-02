# SOFTWARE REQUIREMENTS SPECIFICATION (SRS)
## DỰ ÁN: HỆ THỐNG ĐỊNH DANH ĐIỆN TỬ TRỰC TUYẾN (eKYC) - ABC BANK
**Chuẩn tài liệu:** IEEE Std 830-1998  
**Phiên bản:** v1.0  
**Đối tượng bàn giao:** Đội ngũ Phát triển (Developers) và Đảm bảo chất lượng (QA/Testers)

---

## PHẦN 1: INTRODUCTION (GIỚI THIỆU CHUNG)

### 1.1 Purpose (Mục đích tài liệu)
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) này quy định chi tiết các yêu cầu chức năng, phi chức năng, quy tắc nghiệp vụ và kiến trúc logic cho mô-đun Định danh điện tử khách hàng trực tuyến (eKYC) thuộc hệ thống Ngân hàng số ABC Bank. Tài liệu cung cấp căn cứ kỹ thuật chính xác để đội ngũ lập trình triển khai hệ thống và đội ngũ QA/Tester xây dựng kịch bản kiểm thử (Test Cases).

### 1.2 Scope (Phạm vi hệ thống eKYC)
*   **Hạng mục thuộc phạm vi (In-Scope):**
    *   Mô-đun giao diện ứng dụng di động (Mobile App) thu thập thông tin đăng ký ban đầu và xác thực OTP.
    *   Tích hợp giải pháp OCR tự động nhận diện, phân loại và trích xuất dữ liệu từ Giấy tờ tùy thân (CCCD 12 số, CCCD gắn chip).
    *   Tích hợp giải pháp Liveness Check (chống giả mạo sinh trắc học khuôn mặt sinh sống) và Face Matching (đối chiếu khuôn mặt chụp selfie với ảnh trên CCCD).
    *   Hệ thống kiểm tra điều kiện logic nội bộ ngân hàng và danh sách đen rủi ro rà soát rửa tiền (AML/PEP).
    *   Hệ thống API kết nối tự động gọi Core Banking để khởi tạo mã khách hàng (CIF) và số tài khoản ngân hàng.
    *   Giao diện bảng điều khiển quản trị (Back-office Portal) để kiểm soát viên phê duyệt thủ công các trường hợp nghi vấn (Vùng xám).
*   **Hạng mục ngoài phạm vi (Out-of-Scope):**
    *   Quy trình phát hành thẻ vật lý sau khi mở tài khoản.
    *   Quy trình định danh nâng cao trực tiếp tại quầy giao dịch vật lý.

### 1.3 Definitions, Acronyms (Bảng thuật ngữ)

| Thuật ngữ | Khái niệm đầy đủ | Định nghĩa / Mô tả ý nghĩa |
| :--- | :--- | :--- |
| **eKYC** | Electronic Know Your Customer | Định danh và xác thực danh tính khách hàng bằng phương thức điện tử không cần gặp mặt trực tiếp. |
| **OCR** | Optical Character Recognition | Công nghệ nhận dạng ký tự quang học, chuyển đổi hình ảnh chữ viết trên giấy tờ thành văn bản số. |
| **Liveness Check** | Liveness Detection | Kỹ thuật kiểm tra thực thể sống nhằm xác định hình ảnh khuôn mặt thu thập từ camera là một con người thực tế đang tương tác, không phải ảnh chụp lại hay Deepfake. |
| **Face Matching** | Facial Recognition Matching | Thuật toán so sánh các điểm đặc trưng sinh trắc học giữa hai khuôn mặt để đưa ra tỷ lệ phần trăm trùng khớp. |
| **CIF** | Customer Information File | Tệp thông tin khách hàng, mã định danh duy nhất của mỗi khách hàng trong hệ thống lõi ngân hàng. |
| **Core Banking** | Core Banking System | Hệ thống xử lý các giao dịch tài chính lõi của ngân hàng (Mở tài khoản, gửi tiền, chuyển tiền...). |
| **AML / PEP** | Anti-Money Laundering / Politically Exposed Persons | Chống rửa tiền / Người có ảnh hưởng chính trị thuộc danh sách giám sát rủi ro đặc biệt. |
| **FAR / FRR** | False Acceptance Rate / False Rejection Rate | Tỷ lệ chấp nhận sai (lọt gian lận) / Tỷ lệ từ chối sai (từ chối nhầm khách hàng thật). |

---

## PHẦN 2: OVERALL DESCRIPTION (MÔ TẢ TỔNG QUAN)

### 2.1 Product Perspective (Góc nhìn sản phẩm)
Hệ thống eKYC là một phân hệ dịch vụ tiền sảnh (Front-end Service) chạy độc lập nhưng kết nối chặt chẽ trong hệ sinh thái Ngân hàng số ABC Bank. Hệ thống đóng vai trò cổng tiếp nhận và kiểm soát an toàn dữ liệu khách hàng đầu vào, sau khi tự động thẩm định và định danh đạt yêu cầu sẽ chuyển tiếp dữ liệu có cấu trúc sang hệ thống Middleware API để ra lệnh cho Core Banking tạo tài khoản tức thì mà không cần con người xử lý tại quầy.

### 2.2 User Classes and Characteristics (Đặc điểm người dùng)
*   **Khách hàng vãng lai (Prospective Customer):** Nhóm người dùng đại chúng, thao tác trên thiết bị di động cá nhân (iOS/Android). Đòi hỏi giao diện trực quan, thao tác nhanh gọn, tốc độ phản hồi tính bằng giây. Trình độ công nghệ từ phổ thông đến cao cấp.
*   **Kiểm soát viên rủi ro (Back-office Auditor):** Nhân viên nghiệp vụ phòng Quản lý rủi ro/Vận hành của ngân hàng. Sử dụng hệ thống trên máy tính qua trình duyệt web. Có chuyên môn cao về kiểm tra chứng từ giấy tờ, xử lý nhanh các hồ sơ bị treo ở "Vùng xám".
*   **Quản trị viên hệ thống (System Administrator):** Kỹ sư CNTT của ngân hàng chịu trách nhiệm cấu hình hệ thống, theo dõi log API, giám sát hiệu năng, cấu hình các ngưỡng điểm tin cậy sinh trắc học (`Confidence Score Threshold`).

### 2.3 Constraints (Các giới hạn ràng buộc)
*   **Ràng buộc pháp lý:** Hệ thống phải tuân thủ nghiêm ngặt Nghị định số 13/2023/NĐ-CP của Chính phủ về bảo vệ dữ liệu cá nhân, Thông tư của Ngân hàng Nhà nước Việt Nam quy định về việc mở và sử dụng tài khoản thanh toán bằng phương thức điện tử (eKYC).
*   **Ràng buộc công nghệ thiết bị:** Khách hàng phải sử dụng smartphone có Camera độ phân giải tối thiểu 5.0 MP, có kết nối Internet ổn định (3G/4G/5G/Wifi). Ứng dụng chỉ hỗ trợ hệ điều hành iOS từ phiên bản 14.0 trở lên và Android từ phiên bản 9.0 trở lên.
*   **Ràng buộc an ninh dữ liệu:** Toàn bộ dữ liệu hình ảnh khuôn mặt và ảnh giấy tờ gốc không được phép lưu tạm ở bộ nhớ đệm (Cache) cục bộ của thiết bị di động sau khi kết thúc phiên giao dịch eKYC.

---

## PHẦN 3: SPECIFIC FUNCTIONAL REQUIREMENTS (YÊU CẦU CHỨC NĂNG CHI TIẾT)

### 3.1 Module 01: Đăng ký tài khoản & Xác thực OTP
*   **Mô tả:** Hệ thống tiếp nhận thông tin khởi tạo gồm Số điện thoại và Email của khách hàng, tiến hành gửi mã xác thực để xác minh quyền sở hữu thiết bị đầu cuối.
*   **Tác nhân:** Khách hàng vãng lai, Hệ thống SMS Gateway.
*   **Tiền điều kiện:** Người dùng đã cài đặt thành công ứng dụng di động ABC Bank và có kết nối mạng mạng hoạt động ổn định.
*   **Hậu điều kiện:** Số điện thoại của khách hàng được xác thực trạng thái hoạt động hợp pháp, phiên làm việc (Session) của khách hàng được khởi tạo an toàn để chuẩn bị chuyển sang bước eKYC tiếp theo.
*   **Luồng xử lý chi tiết:**

| Bước | Hành động của Tác nhân | Phản hồi của Hệ thống | Luồng ngoại lệ / Lỗi |
| :--- | :--- | :--- | :--- |
| 1 | Khách hàng nhập Số điện thoại và Email hợp lệ vào phom ứng dụng, bấm "Tiếp tục". | Hệ thống kiểm tra định dạng dữ liệu chữ đầu vào. Thực hiện kết nối và tạo một mã số ngẫu nhiên gồm 6 ký tự số. | Nếu SĐT đã đăng ký tài khoản trước đó, hệ thống báo lỗi "Số điện thoại này đã tồn tại trong hệ thống, vui lòng Đăng nhập". |
| 2 | Hệ thống SMS Gateway gửi tin nhắn chứa mã OTP tới số điện thoại khách hàng. | Ứng dụng chuyển sang màn hình đếm ngược thời gian nhập OTP (120 giây). | Nếu kết nối đến hệ thống Gateway bị ngắt, ứng dụng báo lỗi: "Hệ thống bận, vui lòng thử lại sau". |
| 3 | Khách hàng nhận tin nhắn SMS, điền chuỗi 6 ký tự số vào ô nhập liệu OTP trên app. | Hệ thống đối chiếu mã khách hàng nhập vào với mã đã lưu trữ trên bộ nhớ đệm máy chủ Server. | Nếu nhập sai mã số, hệ thống báo lỗi "Mã xác thực không chính xác, vui lòng kiểm tra lại". Cho phép bấm gửi lại OTP sau khi hết thời gian đếm ngược. |
| 4 | Kết quả đối chiếu chính xác. | Hệ thống đánh dấu trạng thái SĐT là `VERIFIED` và chuyển người dùng sang màn hình Chụp CCCD. | |

### 3.2 Module 02: Upload & Đọc thông tin CCCD (OCR)
*   **Mô tả:** Hướng dẫn người dùng đưa thẻ căn công dân (CCCD) trước camera, tự động chụp hình ảnh chất lượng cao và chuyển giao sang lõi thuật toán OCR để số hóa dữ liệu dạng text.
*   **Tác nhân:** Khách hàng vãng lai, Dịch vụ lõi OCR.
*   **Tiền điều kiện:** Số điện thoại đã được xác thực hoàn tất ở Module 01. Khách hàng cấp quyền cho ứng dụng truy cập vào phần cứng Camera của thiết bị.
*   **Hậu điều kiện:** Trích xuất đầy đủ và lưu trữ tạm thời các trường văn bản từ giấy tờ tùy thân của khách hàng vào bộ nhớ phiên của Server.
*   **Luồng xử lý chi tiết:**

| Bước | Hành động của Tác nhân | Phản hồi của Hệ thống | Luồng ngoại lệ / Lỗi |
| :--- | :--- | :--- | :--- |
| 1 | Khách hàng đưa mặt trước của thẻ CCCD vào khung hình chữ nhật hướng dẫn trên màn hình. | Ứng dụng tự động nhận diện độ nét, độ sáng, tiến hành quét hình ảnh mà không cần bấm nút (Auto-crop). | Nếu ảnh chụp bị che khuất góc hoặc không nằm trọn trong khung hình, app thông báo nhắc nhở "Hãy đưa toàn bộ giấy tờ vào trong khung". |
| 2 | Khách hàng lật mặt sau của thẻ CCCD và lặp lại thao tác tương tự theo hướng dẫn. | Hệ thống kiểm tra chất lượng ảnh mặt sau của giấy tờ. | |
| 3 | Người dùng xác nhận gửi cả hai hình ảnh mặt trước và mặt sau lên Server xử lý. | Hệ thống gửi dữ liệu nhị phân ảnh qua API đến dịch vụ lõi OCR. Dịch vụ OCR trả về chuỗi JSON thông tin cá nhân. | Nếu phát hiện giấy tờ photocopy, scan mờ hoặc có dấu hiệu làm giả chỉnh sửa ảnh, hệ thống hiển thị mã lỗi: "Giấy tờ không hợp lệ, vui lòng dùng giấy tờ gốc". |
| 4 | Người dùng xem lại bảng kết quả hiển thị thông tin trích xuất trên app. | Ứng dụng cho phép người dùng tự sửa đổi thủ công một số trường thông tin bị sai sót do chữ mờ (như Quê quán) trừ trường Số CCCD, Họ tên, Ngày sinh. | Nếu thông tin trích xuất hiển thị cho thấy khách hàng chưa đủ 18 tuổi, hệ thống lập tức chấm dứt phiên đăng ký: "Rất tiếc, dịch vụ chỉ áp dụng cho khách hàng từ đủ 18 tuổi trở lên". |

### 3.3 Module 03: Xác thực khuôn mặt sinh trắc học (Liveness Check & Face Matching)
*   **Mô tả:** Hệ thống thực hiện ghi hình chuyển động của người dùng để chống gian lận bằng hình ảnh tĩnh, sau đó tiến hành so sánh đối chiếu khuôn mặt người thật với ảnh chân dung trên thẻ CCCD để tính điểm tương đồng.
*   **Tác nhân:** Khách hàng vãng lai, Dịch vụ nhận diện khuôn mặt sinh trắc học.
*   **Tiền điều kiện:** Dữ liệu text và hình ảnh CCCD đã được xử lý và ghi nhận thành công ở Module 02.
*   **Hậu điều kiện:** Trả về kết quả đánh giá thực thể sống (`Liveness_Status: PASSED/FAILED`) và điểm số trùng khớp sinh trắc học (`Face_Matching_Score: 0% - 100%`).
*   **Luồng xử lý chi tiết:**

| Bước | Hành động của Tác nhân | Phản hồi của Hệ thống | Luồng ngoại lệ / Lỗi |
| :--- | :--- | :--- | :--- |
| 1 | Khách hàng đưa khuôn mặt vào giữa khung tròn camera selfie theo hướng dẫn trên màn hình. | Ứng dụng kích hoạt Camera trước, điều chỉnh độ sáng màn hình tự động phù hợp. | Nếu camera không phát hiện thấy khuôn mặt trong khung hình sau 10 giây, app báo lỗi "Không tìm thấy khuôn mặt, vui lòng thử lại". |
| 2 | Khách hàng thực hiện các hành động ngẫu nhiên theo lệnh của app (ví dụ: Quay sang trái, chớp mắt, quay sang phải, mỉm cười). | Thuật toán Liveness Check chạy trực tiếp để phân tích chuyển động cơ mặt và các điểm ảnh theo chiều sâu để xác thực thực thể người đang sống. | Nếu phát hiện người dùng sử dụng mặt nạ in 3D, video phát lại trên máy tính bảng hoặc ảnh tĩnh, hệ thống chặn tức thì và báo lỗi "Phát hiện gian lận sinh trắc học, phiên bị từ chối". |
| 3 | Sau khi vượt qua Liveness Check, hệ thống tự động lưu hình ảnh chụp selfie sắc nét nhất. | Hệ thống chạy thuật toán Face Matching, so sánh ảnh selfie thu được với ảnh chân dung cắt ra từ thẻ CCCD thu thập ở Module 02. | Nếu điểm tương đồng khuôn mặt `Face_Matching_Score` < 75%, hệ thống thông báo lỗi: "Khuôn mặt không trùng khớp với giấy tờ, vui lòng thực hiện lại cẩn thận". |
| 4 | Quá trình đối chiếu hoàn tất. | Hệ thống ghi nhận các thông số điểm số cụ thể gửi về máy chủ Server trung tâm. | |

### 3.4 Module 04: Đối chiếu điều kiện và Tự động kích hoạt tài khoản
*   **Mô tả:** Hệ thống tự động phân loại xử lý hồ sơ dựa trên các điểm số tin cậy thu được để ra quyết định duyệt mở tài khoản tự động (Zero-manual) hoặc chuyển giao duyệt thủ công.
*   **Tác nhân:** Hệ thống eKYC (System backend), Hệ thống Quản trị rủi ro (Risk & Compliance System), Hệ thống Core Banking, Kiểm soát viên rủi ro (Back-office Auditor).
*   **Tiền điều kiện:** Khách hàng đã hoàn thành trọn vẹn và hợp lệ tất cả các bước xác thực thông tin tại Module 01, 02 và 03.
*   **Hậu điều kiện:** Khách hàng được cấp Số tài khoản ngân hàng mới tự động HOẶC hồ sơ được chuyển vào danh sách chờ phê duyệt thủ công.
*   **Luồng xử lý chi tiết:**

| Bước | Hành động của Tác nhân / Hệ thống | Phản hồi của Hệ thống Backend | Luồng ngoại lệ / Lỗi |
| :--- | :--- | :--- | :--- |
| 1 | Hệ thống tự động gửi yêu cầu kiểm tra dữ liệu khách hàng sang Hệ thống Quản trị rủi ro nội bộ để quét danh sách Blacklist/AML/PEP. | Hệ thống Quản trị rủi ro trả về trạng thái danh sách (`Clean` hoặc `Flagged`). | Nếu trạng thái trả về là `Flagged` (Khách hàng thuộc danh sách đen cấm giao dịch tài chính), hệ thống từ chối mở tài khoản và hiển thị thông báo chung chung: "Đăng ký không thành công, vui lòng liên hệ quầy giao dịch gần nhất". |
| 2 | Hệ thống kiểm tra bộ quy tắc logic nghiệp vụ (`Business Rules Validation`). | Phân loại hồ sơ khách hàng dựa trên điểm số: <br> - **Trường hợp 1 (Happy Path - Duyệt tự động):** Nếu `OCR_Confidence` $\ge 98\%$ VÀ `Face_Matching` $\ge 95\%$ VÀ Rủi ro = `Clean`. <br> - **Trường hợp 2 (Vùng xám - Duyệt tay):** Nếu điểm `Face_Matching` nằm trong khoảng từ $75\%$ đến $94\%$. | **Xử lý Trường hợp 2 (Luồng rẽ nhánh duyệt tay):** Hệ thống tạo một bản ghi trạng thái `PENDING_REVIEW`, chuyển hồ sơ vào hàng đợi xử lý trên Web Portal của Kiểm soát viên. Khách hàng nhận được thông báo trên app: "Hồ sơ của bạn đang được xử lý thẩm định, kết quả sẽ gửi trong vòng 15 phút". Kiểm soát viên xem xét hình ảnh, bấm duyệt (`APPROVED`) hoặc từ chối (`REJECTED`) thủ công bằng nút bấm. |
| 3 | Hệ thống thực hiện lệnh gọi API đồng bộ sang Hệ thống Core Banking của ngân hàng. | Core Banking tiếp nhận payload dữ liệu, tự động khởi tạo Hồ sơ thông tin khách hàng (CIF) và mở Số tài khoản mới dựa trên dãy số định sẵn của ABC Bank. | Nếu API kết nối Core Banking bị timeout hoặc lỗi mạng nội bộ, hệ thống tự động lưu vào hàng đợi xếp hàng thử lại (Retry Queue) tối đa 5 lần trong chu kỳ. |
| 4 | Core Banking gửi thông tin tài khoản mới tạo thành công về cho Hệ thống eKYC. | Hệ thống gửi Email xác nhận và tin nhắn SMS chào mừng đến khách hàng chứa Số tài khoản mới và thông tin mật khẩu đăng nhập ứng dụng lần đầu. Trạng thái hồ sơ đổi thành `SUCCESS`. | |

---

## PHẦN 4: NON-FUNCTIONAL REQUIREMENTS (YÊU CẦU PHI CHỨC NĂNG)

### 4.1 Security Requirements (Yêu cầu bảo mật dữ liệu)
*   **An toàn kênh truyền dẫn:** Toàn bộ dữ liệu tương tác giữa ứng dụng Mobile App và máy chủ API Gateway của ngân hàng bắt buộc phải được mã hóa mã hóa an toàn bằng giao thức bảo mật lớp truyền tải **HTTPS/TLS 1.3**. Áp dụng kỹ thuật gắn chặt chứng chỉ số SSL Pinning trên ứng dụng di động để ngăn chặn hoàn toàn các cuộc tấn công nghe lén, giả mạo ở giữa (Man-in-the-middle attacks).
*   **Bảo mật dữ liệu lưu trữ:** Các dữ liệu cá nhân nhạy cảm của người dùng bao gồm: Họ tên, Số điện thoại, Số căn cước công dân và hình ảnh chân dung sinh trắc học khi lưu trữ vĩnh viễn trong Cơ sở dữ liệu (Database) bắt buộc phải được mã hóa bằng thuật toán mã hóa đối xứng mạnh **AES-256 bits** ở cấp độ trường dữ liệu. Khóa mật mã (Encryption Keys) phải được quản lý tập trung và an toàn tuyệt đối trong mô-đun bảo mật phần cứng chuyên dụng (HSM).
*   **Tiêu chuẩn tuân thủ:** Toàn bộ cấu trúc hạ tầng hệ thống kỹ thuật phục vụ lưu trữ và truyền tải thông tin định danh eKYC phải tuân thủ nghiêm ngặt theo các tiêu chuẩn an toàn bảo mật dữ liệu quốc tế như **PCI-DSS** và các thông tư hướng dẫn về bảo mật hệ thống thông tin cấp độ 3 của Ngân hàng Nhà nước.

### 4.2 Performance Requirements (Yêu cầu hiệu năng)
*   **Thời gian đáp ứng dịch vụ (Response Time):**
    *   Thời gian xử lý của mô-đun trích xuất OCR (kể từ lúc hệ thống nhận đủ ảnh hai mặt giấy tờ cho đến khi trả ra kết quả text dữ liệu dạng cấu trúc JSON trên màn hình app) không được vượt quá **2.0 giây**.
    *   Thời gian xử lý thuật toán đối chiếu sinh trắc học và kiểm tra Liveness Check không được vượt quá **3.0 giây** cho một giao dịch eKYC.
    *   Tổng thời gian của toàn bộ quy trình từ lúc khách hàng gửi hồ sơ hoàn chỉnh ở bước cuối cùng cho đến khi hệ thống Core Banking xử lý xong cấp số tài khoản tự động (Trường hợp Happy Path) phải nằm trong giới hạn dưới **5.0 giây**.
*   **Khả năng chịu tải (Throughput & Scalability):** Hệ thống tại kiến trúc máy chủ trung tâm phải chịu tải hoạt động ổn định với hiệu năng tối thiểu đạt **500 CCU** (Concurrent Users - Số người dùng thực hiện thao tác eKYC đồng thời tại một thời điểm) mà không làm tăng thời gian phản hồi API quá quy định. Hệ thống phải có khả năng tự động mở rộng tài nguyên (Auto-scaling) trên hạ tầng Cloud/On-premise khi lượng tải CCU tăng đột biến vượt ngưỡng cấu hình 80% dung lượng CPU/RAM của máy chủ.

### 4.3 Availability & Reliability Requirements (Tính sẵn sàng và độ tin cậy)
*   **Chỉ số hoạt động (Availability):** Cam kết tính sẵn sàng của toàn hệ thống phục vụ luồng eKYC đạt chỉ số tối thiểu **SLA 99.9%** hoạt động liên tục 24/7/365 ngày (kể cả thời gian nghỉ lễ, tết), cho phép thời gian downtime bảo trì hệ thống theo kế hoạch tối đa không quá 8.76 giờ trong suốt một năm hoạt động.
*   **Độ chính xác kỹ thuật (Reliability metrics):**
    *   Tỷ lệ hệ thống nhận diện từ chối sai lệch đối với khách hàng có giấy tờ thật và khuôn mặt thật hoàn toàn hợp lệ (**FRR - False Rejection Rate**) phải kiểm soát ở mức nhỏ hơn **2%**.
    *   Tỷ lệ hệ thống nhận diện sai lệch chấp nhận cho các hành vi dùng giấy tờ giả mạo, khuôn mặt photo giả mạo vượt qua được hệ thống định danh (**FAR - False Acceptance Rate**) phải kiểm soát ở mức cực kỳ khắt khe nhỏ hơn **0.001%**.

---

## PHẦN 5: VISUAL DIAGRAM (SƠ ĐỒ TRỰC QUAN)

### 5.1 Use Case Diagram mô tả luồng thao tác eKYC
<img width="762" height="757" alt="image" src="https://github.com/user-attachments/assets/db43673e-f50d-4abc-bd06-6672dbe73180" />
