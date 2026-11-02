# UniJob – Nền tảng việc làm part-time cho sinh viên

Đồ án **Chuyên đề tốt nghiệp 2** – Chuyên ngành Công nghệ phần mềm – Trường Đại học Văn Lang.
GVHD: ThS. Nguyễn Văn Trung.

UniJob là một marketplace hai chiều kết nối **sinh viên** tìm việc làm thêm với **nhà tuyển dụng** (doanh nghiệp, cửa hàng hoặc cá nhân).
Luồng nghiệp vụ chính: **đăng tin → ứng tuyển → duyệt → hoàn thành → đánh giá hai chiều**. Hệ thống có 3 vai trò: Sinh viên, Nhà tuyển dụng, Quản trị viên.

Phần mở rộng về công nghệ dữ liệu, dùng bộ dữ liệu **VietJobs** (48.092 tin):
- Pipeline làm sạch dữ liệu và thống kê thị trường lao động bằng Python/pandas
- **Gợi ý việc làm** cho sinh viên theo hướng nội dung (TF-IDF + cosine)
- **Phân loại ngành tự động** từ tiêu đề tin bằng Multinomial Naive Bayes (accuracy 62,7%, top-3 87,8%, baseline 15,9%)
- **Gợi ý mức lương** khi đăng tin (P25/trung vị/P75 theo ngành, quy đổi theo giờ/ca)
- Phân tích **skill-gap** (kỹ năng còn thiếu so với tin) và xếp hạng ứng viên theo độ phù hợp
- Dashboard thống kê và xuất CSV cho quản trị viên

Các chức năng kiểu trang tuyển dụng lớn (bổ sung):
- **Danh bạ công ty** + trang công ty (việc đang tuyển, đánh giá), **theo dõi công ty yêu thích** và nhận thông báo khi công ty đăng tin mới
- **Lọc nâng cao**: làm thứ 7 (tự suy ra từ giờ làm), gợi ý AI (xếp theo độ phù hợp hồ sơ), danh mục nghề kèm số tin,
  kinh nghiệm (8 mức), lĩnh vực công ty, lĩnh vực công việc, loại công ty (đã xác minh), mức lương (khoảng có sẵn, tự nhập, thỏa thuận —
  lương giờ/ca/ngày được quy đổi về tháng), cấp bậc, loại hình làm việc, đăng trong vòng N ngày, việc gấp, công ty đang theo dõi; **Lưu bộ lọc**
- **Thông báo việc làm** (job alert): lưu tiêu chí tìm kiếm, tự báo khi có tin mới khớp
- **Tra cứu lương** theo ngành × kinh nghiệm × tỉnh/thành (P25 – trung vị – P75) từ dữ liệu VietJobs
- **Cẩm nang nghề nghiệp**, trang Giới thiệu, Điều khoản, Quyền riêng tư, Trợ giúp
- **Tạo CV** tự động từ hồ sơ, công việc đã hoàn thành và đánh giá (in/lưu PDF)
- **Tìm ứng viên** cho nhà tuyển dụng (xếp theo độ phù hợp với tin) và **mời ứng tuyển**
- **Việc làm tương tự** ở trang chi tiết tin

## Cấu trúc thư mục

```
unijob/
├── backend/            Spring Boot 3.3 (Java 17), Spring Security + JWT, JPA, PostgreSQL
│   ├── src/main/java/vn/edu/vlu/unijob/
│   │   ├── config/       SecurityConfig, WebConfig, DataSeeder
│   │   ├── security/     JwtService, JwtAuthFilter, CurrentUser
│   │   ├── entity/       User, StudentProfile, EmployerProfile, JobPost, Application, Review, ...
│   │   ├── repository/   Spring Data JPA
│   │   ├── service/      nghiệp vụ + RecommendationService, CategoryClassifier, MarketInsightService
│   │   ├── controller/   REST API
│   │   └── dto/, exception/, util/
│   ├── src/main/resources/data/   market_stats.json, category_model.json, seed_jobs.json
│   └── src/test/        unit test + integration test (MockMvc + H2)
├── frontend/           ReactJS 19 (bundle bằng esbuild), giao diện responsive
│   └── demo/           bản demo 1 file HTML, backend giả lập trong trình duyệt
├── data-pipeline/      pipeline.py: làm sạch VietJobs, EDA, huấn luyện Naive Bayes, xuất dữ liệu
├── docs/               sơ đồ (Mermaid/Graphviz), ảnh chụp màn hình, công cụ phụ trợ
└── docker-compose.yml
```

## Cách chạy

### Cách 1 – Docker (nhanh nhất)
```bash
cd frontend && npm install && npm run build:embed && cd ..   # nhúng giao diện vào backend
docker compose up --build
```
Mở http://localhost:8080

### Cách 2 – Chạy thủ công
Yêu cầu: JDK 17+, Maven 3.9+, Node.js 18+, PostgreSQL 14+ (hoặc dùng profile H2).

```bash
# 1. CSDL
createdb unijob   # user/pass mặc định unijob/unijob (đổi qua biến DB_URL, DB_USERNAME, DB_PASSWORD)

# 2. Backend (lần chạy đầu tự tạo bảng và nạp dữ liệu mẫu)
cd backend
mvn spring-boot:run
#   không có PostgreSQL thì chạy:  mvn spring-boot:run -Dspring-boot.run.profiles=h2

# 3. Frontend (cửa sổ khác)
cd frontend
npm install
npm run dev        # http://localhost:5173, gọi API tại http://localhost:8080
```

MySQL: thêm dependency `com.mysql:mysql-connector-j`, rồi đặt
`DB_URL=jdbc:mysql://localhost:3306/unijob`.

### Cách 3 – Bản demo không cần backend
```bash
cd frontend
npm install
npm run build:demo     # tạo frontend/demo/unijob-demo.html
```
Mở `frontend/demo/unijob-demo.html` bằng trình duyệt (hoặc đưa lên GitHub Pages/Netlify) là dùng được ngay.
Bản này giả lập backend trong trình duyệt (`frontend/demo/setup.js`): chặn các lời gọi `/api/...` và xử lý
theo đúng hợp đồng API, quy tắc trạng thái, gợi ý việc làm, phân loại ngành và gợi ý lương như bản Spring Boot,
với cùng dữ liệu mẫu. Dữ liệu chỉ nằm trong bộ nhớ trình duyệt nên tải lại trang sẽ về dữ liệu ban đầu;
không hỗ trợ tải CV và xuất CSV. Dùng để trình diễn nhanh khi không có máy chủ.

### Kiểm thử
```bash
cd backend
mvn test     # unit test UT-01 … UT-12 + integration test TC-01 … TC-26
```

### Chạy lại pipeline dữ liệu
```bash
pip install pandas numpy matplotlib
cd data-pipeline
python pipeline.py --input /đường/dẫn/VietJobs.csv
```
Script ghi `market_stats.json`, `category_model.json`, `seed_jobs.json` vào `backend/src/main/resources/data` và lưu biểu đồ vào `data-pipeline/output/charts`.

## Tài khoản demo

| Vai trò | Email | Mật khẩu |
|---|---|---|
| Sinh viên | sv@unijob.vn | Sinhvien@123 |
| Nhà tuyển dụng | ntd@unijob.vn | Tuyendung@123 |
| Quản trị viên | admin@unijob.vn | Admin@123 |

Ngoài ra có 28 sinh viên `sinhvien1..28@unijob.vn` và 7 nhà tuyển dụng khác (cùng mật khẩu theo vai trò).

## API chính

| Phương thức | Đường dẫn | Mô tả | Quyền |
|---|---|---|---|
| POST | /api/auth/register, /api/auth/login | Đăng ký / đăng nhập, trả JWT | Công khai |
| GET | /api/jobs?q=&category=&city=&jobType=&minSalary=&sort= | Tìm kiếm, lọc tin | Công khai |
| GET | /api/jobs/{id} | Chi tiết tin (+ skill-gap nếu là SV) | Công khai |
| POST/PUT/DELETE | /api/jobs, /api/jobs/{id} | Đăng / sửa / xóa tin | NTD |
| PATCH | /api/jobs/{id}/status | OPEN, FILLED, COMPLETED, CLOSED | NTD chủ tin |
| POST | /api/jobs/{id}/apply | Ứng tuyển | SV |
| GET | /api/jobs/{id}/applications | Danh sách ứng viên (xếp theo điểm phù hợp) | NTD chủ tin |
| PATCH | /api/applications/{id}/decision, /withdraw | Duyệt/từ chối, rút hồ sơ | NTD / SV |
| POST | /api/reviews ; GET /api/reviews/pending | Đánh giá hai chiều | SV, NTD |
| GET | /api/notifications, /unread-count | Thông báo | Đã đăng nhập |
| GET | /api/recommendations | Gợi ý việc làm | SV |
| GET | /api/insights/platform | Thống kê thị trường trên UniJob theo tiêu chí bộ lọc nâng cao | Công khai |
| GET | /api/insights/market, /classify, /salary-suggestion | Chức năng dữ liệu | Công khai |
| GET | /api/admin/stats, /users, /jobs, /export/applications.csv | Quản trị | Admin |
| GET / POST | /api/companies, /api/companies/{id}, /api/companies/{id}/follow | Danh bạ công ty, theo dõi | Công khai / SV |
| GET / POST / PUT / DELETE | /api/job-alerts, /api/job-alerts/{id} | Thông báo việc làm | SV |
| GET / POST | /api/candidates, /api/candidates/{id}/invite | Tìm ứng viên, mời ứng tuyển | NTD |
| GET | /api/jobs/{id}/similar | Việc làm tương tự | Công khai |
| GET | /api/jobs/facets, /api/taxonomy | Số tin theo danh mục nghề; danh mục lĩnh vực | Công khai |
| GET | /api/insights/salary-lookup?category&experience&city | Tra cứu lương | Công khai |

## Ghi chú
`docs/tools/mock_api.py` là server Python giả lập đúng hợp đồng API, chỉ dùng để xem trước giao diện và chụp màn hình khi chưa build được backend. Nó **không** thuộc sản phẩm.
#   c d t n 2 - u n i j o b  
 