# ORIGINALS — Hướng dẫn cài đặt

Website gồm **2 phần dùng chung 1 dữ liệu**:

- **Cửa hàng (khách):** `index.html`, `collection.html`, `product.html`, `cart.html`, `order.html`
- **Trang quản trị (admin):** `admin/index.html`

Web chạy ở **2 chế độ**, tự nhận biết qua file `assets/config.js`:

| | Chế độ DEMO (mặc định) | Chế độ THẬT (Supabase) |
|---|---|---|
| Cấu hình | Không cần gì | Dán URL + key Supabase |
| Lưu dữ liệu | localStorage (1 trình duyệt) | Cloud, đồng bộ mọi thiết bị |
| Admin & khách khác máy thấy chung? | ❌ Không | ✅ Có |
| Đăng nhập admin | Nút "đăng nhập demo" | Google thật (chỉ email admin) |
| Upload ảnh | Lưu tạm trong trình duyệt | Lưu trên Supabase Storage |

---

## A. Xem thử ngay (chế độ demo)

1. Mở `index.html` bằng trình duyệt → mua hàng, đặt đơn, nhận **mã đơn**.
2. Mở `order.html` → nhập mã để xem trạng thái.
3. Mở `admin/index.html` → bấm đăng nhập (demo) → vào quản trị, bấm **"Nhập dữ liệu mẫu"** để có 30 sản phẩm, thử đổi trạng thái đơn, thêm/sửa sản phẩm.

> Demo chỉ lưu trên đúng trình duyệt đó. Để admin và khách (máy khác) dùng chung, làm tiếp phần B.

---

## B. Bật chế độ THẬT với Supabase (miễn phí)

### Bước 1 — Tạo project
1. Vào https://supabase.com → đăng ký → **New project** (chọn region Singapore cho nhanh).
2. Đặt mật khẩu database (lưu lại), bấm tạo, đợi ~2 phút.

### Bước 2 — Tạo bảng dữ liệu
1. Mở **SQL Editor** → **New query**.
2. Mở file `supabase/schema.sql`, copy toàn bộ, dán vào, bấm **Run**.
   - Nếu email admin của bạn KHÁC `anhcoming@gmail.com`, sửa email đó trong phần `is_admin()` của file trước khi chạy.

### Bước 3 — Lấy khoá API và dán vào web
1. Trong Supabase: **Project Settings → API**.
2. Copy **Project URL** và **anon public key**.
3. Mở `assets/config.js`, điền:
   ```js
   SUPABASE_URL: "https://xxxx.supabase.co",
   SUPABASE_ANON_KEY: "eyJhbGciOi....",
   ADMIN_EMAILS: ["anhcoming@gmail.com"],
   ```
   > anon key để công khai là **bình thường** — dữ liệu được bảo vệ bằng RLS trong SQL.

### Bước 4 — Bật đăng nhập Google cho admin
1. Supabase: **Authentication → Providers → Google → Enable**.
2. Tạo OAuth ở Google Cloud (https://console.cloud.google.com):
   - APIs & Services → Credentials → **Create OAuth client ID** → Web application.
   - **Authorized redirect URI**: dán đúng dòng Supabase hiện ở mục Google provider, dạng
     `https://xxxx.supabase.co/auth/v1/callback`
   - Copy **Client ID** + **Client Secret** dán ngược lại vào Supabase, Save.
3. Supabase: **Authentication → URL Configuration** → đặt **Site URL** = địa chỉ web của bạn (xem Bước 5) và thêm vào **Redirect URLs**.

### Bước 5 — Đưa web lên mạng (bắt buộc cho Google login)
Đăng nhập Google **không chạy được khi mở file trực tiếp** (`file://`) — cần địa chỉ http/https. Chọn 1 cách miễn phí:

- **Netlify Drop** (dễ nhất): vào https://app.netlify.com/drop → kéo-thả cả thư mục `originals-store` → có link ngay.
- **Vercel** hoặc **GitHub Pages** cũng được.
- Hoặc chạy thử tại máy: mở terminal trong thư mục, chạy `python -m http.server 8000` rồi vào `http://localhost:8000` (nhớ thêm địa chỉ này vào Redirect URLs ở Bước 4).

### Bước 6 — Khởi tạo dữ liệu
Vào `…/admin/` → đăng nhập Google (đúng email admin) → bấm **"Nhập dữ liệu mẫu"** (hoặc tự thêm sản phẩm). Xong! Khách vào cửa hàng sẽ thấy sản phẩm, đặt hàng; bạn quản lý đơn ở admin.

---

## C. Dùng trang admin

- **Đơn hàng:** xem danh sách, lọc theo trạng thái, đổi trạng thái (Chờ xác nhận → Đã xác nhận → Đang giao → Hoàn thành / Đã huỷ), bấm **Xem** để xem chi tiết. Khách dùng mã đơn để tự tra ở `order.html` — trạng thái cập nhật theo bạn.
- **Sản phẩm:** Thêm/Sửa/Xoá, đặt **tồn kho** (hết hàng tự khoá nút mua), **upload ảnh** (nếu không có ảnh, hệ thống tự vẽ ảnh áo theo màu), bật/tắt hiển thị.

## D. Đổi thương hiệu
- Tên, hotline, email, danh sách email admin: sửa trong `assets/config.js` và `assets/data.js` (khối `BRAND`).
- Màu sắc, font: sửa trong `assets/styles.css` (phần `:root`).

## E. Cấu trúc thư mục
```
originals-store/
├─ index.html, collection.html, product.html, cart.html, order.html
├─ admin/        → index.html, admin.js, admin.css   (trang quản trị)
├─ assets/       → config.js, data.js, store-api.js, app.js, styles.css
└─ supabase/     → schema.sql   (chạy 1 lần trên Supabase)
```

## F. Bảo mật
- Chỉ email trong `ADMIN_EMAILS` **và** trong hàm `is_admin()` (SQL) mới sửa được sản phẩm/đơn.
- Khách chỉ tra được đơn nếu có **đúng mã đơn** (không xem được đơn người khác).
- Muốn thêm admin: thêm email vào cả `config.js` và `is_admin()` rồi chạy lại đoạn `is_admin` trong SQL.
