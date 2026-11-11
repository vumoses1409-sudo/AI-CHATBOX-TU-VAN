# Lạc Lạc — AI Chat Box Tâm Lí (HTML + OpenAI API + MySQL)

Dự án nâng cấp từ giao diện Lạc Lạc HTML ban đầu. Giao diện giữ phong cách lá mầm / vân gỗ và có các mục Trang chủ, Trò chuyện, Đánh giá ban đầu, Kết nối chuyên gia, Góc bình yên, Quyền riêng tư.

## 1) Kiến trúc

```
Browser (public/index.html)
  └── POST /api/chat  ──► Node.js Express (server.js) ──► OpenAI Responses API
                                 │
                                 └──► MySQL (chỉ ghi nếu bật "Lưu lịch sử lên MySQL")
```

**Không đặt API key ở HTML.** Mã bí mật chỉ ở phía máy chủ (`.env` trong máy cá nhân, Environment Variables trên Vercel). API và ChatGPT được thanh toán riêng: gói Plus không kèm tín dụng API; một số gói Pro/Business đủ điều kiện có thể nhận khoản tín dụng API khuyến mại riêng.

## 2) Chạy trên máy tính

Yêu cầu Node.js 20+ và (nếu muốn lưu chat) MySQL 8.x.

1. Mở terminal tại thư mục `lac_lac_openai_mysql`.
2. Chạy `npm install`.
3. Sao chép `.env.example` thành `.env`; thay `OPENAI_API_KEY`, `COOKIE_SECRET`, thông tin MySQL (không chia sẻ tệp `.env`).
4. Mở MySQL Workbench hoặc phpMyAdmin; chạy `sql/schema.sql` để tạo DB và bảng.
5. Chạy `npm start`.
6. Mở `http://localhost:3000` (không mở trực tiếp `public/index.html` bằng `file://` nếu muốn dùng API).
7. Truy cập `http://localhost:3000/api/health` kiểm tra AI (`aiConfigured`) và MySQL (`database`).

Cần có API key hợp lệ và tín dụng hoặc quyền sử dụng API để nhận câu trả lời AI thật. Không có API key, API trả thông báo cấu hình còn thiếu. App không tự động sử dụng hạn mức ChatGPT Plus.

## 3) Cách dùng MySQL

- Mặc định **không ghi tin nhắn vào MySQL**. Bật tùy chọn "Lưu tin mới lên MySQL (tự chọn)" ở trang Trò chuyện mới lưu các tin nhắn tiếp theo; ứng dụng yêu cầu bạn đồng ý.
- "Lưu nội dung trên thiết bị này" và "Lưu lịch sử lên MySQL" là hai lựa chọn **độc lập**.
- Nút "Tải lịch sử MySQL" tải tối đa 80 tin nhắn đã lưu cho phiên trình duyệt hiện tại.
- "Xóa lịch sử MySQL" xóa tin nhắn của phiên hiện tại. Trong trang Quyền riêng tư, "Xóa toàn bộ dữ liệu" cũng gửi yêu cầu xóa bản ghi MySQL của phiên nếu có.
- Phiên được nhận diện bằng signed HttpOnly cookie (hạn 30 ngày). **Chưa có tài khoản đăng ký/đăng nhập thực**; nút nhập tên vẫn là chức năng mô phỏng.
- Nếu xóa cookie, bạn không còn truy cập được lịch sử cũ bằng trình duyệt đó. Cần chính sách xóa dữ liệu định kỳ trên DB trước khi đưa vào sản xuất.
- Phần nhật ký / kết quả đánh giá KHÔNG gửi lên MySQL, chỉ được giữ trên trình duyệt khi người dùng chọn lưu cục bộ.

## 4) Triển khai Vercel

Vercel hỗ trợ Express: `server.js` xuất `app` mặc định, còn tệp tĩnh nằm trong `public/`.

1. Đưa **cả thư mục dự án** lên một GitHub repository (không đưa `.env` lên GitHub).
2. Vào Vercel và import repository dưới dạng Node.js/Express project.
3. Trong Project → Settings → Environment Variables, thêm `OPENAI_API_KEY`, `OPENAI_MODEL`, `COOKIE_SECRET`, `NODE_ENV=production`, cùng `MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_SSL`.
4. MySQL phải là **máy chủ database có thể truy cập từ Vercel** (MySQL trong XAMPP/localhost trên laptop không kết nối trực tiếp được với website Vercel).
5. Chạy SQL tạo bảng trên MySQL cloud của bạn, deploy và thử `/api/health`.
6. Kiểm tra API có thật sự dùng đúng tên miền và cookie `Secure` hoạt động qua HTTPS.

## 5) API endpoints

- `GET /api/health` — trạng thái cấu hình (không lộ khóa và mật khẩu)
- `POST /api/chat` — `{ "messages": [{ "role": "user", "content": "Hôm nay mình hơi lo" }], "saveToMySQL": false }`
- `GET /api/history` — đọc lịch sử cho signed session cookie
- `DELETE /api/history` — xóa lịch sử cho signed session cookie

API mặc định dùng **`gpt-6-luna`**, thay bằng `OPENAI_MODEL` nếu muốn. Các yêu cầu Responses API đặt `store: false` để tránh dùng chế độ lưu trữ phản hồi API. Nếu chọn lưu MySQL, dữ liệu được lưu trên máy chủ DB của bạn, độc lập với tùy chọn lưu Responses API.

## 6) Quan trọng trước khi phát hành công khai

Đây là ứng dụng tham khảo để học tập/trình diễn, **chưa phải nền tảng tư vấn trị liệu hoặc hệ thống bảo mật/đăng nhập sản xuất**. Trước khi mở công khai, cần thêm đăng nhập/xác thực người dùng thực, giới hạn lượt theo tài khoản, CAPTCHA hoặc biện pháp chống lạm dụng, giới hạn chi phí API, bảo mật MySQL trên mạng, thông báo thu thập dữ liệu / đồng ý, chính sách thời hạn giữ và xóa dữ liệu, kiểm thử bảo mật và quy trình hỗ trợ khủng hoảng phù hợp địa phương. Không hiển thị danh sách hay nhật ký của người dùng này cho người dùng khác.

**Trang nguồn:** https://chatbot-tvtl.vercel.app/ (tham chiếu giao diện/chức năng công khai), không kết nối tài khoản hay dữ liệu của website đó.
