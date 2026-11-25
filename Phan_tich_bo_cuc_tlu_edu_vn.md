# Phân tích bố cục trang chủ tlu.edu.vn (bản nén)

Quy trình: **Nhìn giao diện → khoanh hộp → đặt tên → xác định cha/con → chọn Flex/Grid/Block → nhận diện nội dung → chọn HTML semantic.**

> **Lưu ý:** Phân tích dựa trên nội dung và thứ tự khối của trang, không có CSS hay ảnh render.
> **[Thấy]** = có trong nội dung trang. **[Suy luận]** = đoán kỹ thuật CSS từ cấu trúc, cần kiểm tra bằng DevTools (F12).
> Menu và danh sách tin xuất hiện 2 lần, nhiều khả năng là bản desktop và bản mobile/slider.

---

## 1. Cây bố cục chỉ dùng `div`

```text
div.page
├── div.header
│   ├── div.header-container
│   │   ├── div.brand
│   │   ├── div.nav → div.nav-item × 8 → (div.nav-link + div.dropdown)
│   │   └── div.header-tools → (div.search + div.lang-switch)
│   └── div.mobile-menu
├── div.main
│   ├── div.hero → div.hero-slides → div.hero-slide × n
│   ├── div.news-section → (div.section-header, div.news-slider → div.news-card × 7, div.slider-arrows)
│   ├── div.notice-event-section → (div.notice-box → div.notice-item × 6) + (div.event-box → div.event-item × n)
│   ├── div.stats-section → div.stats-list → div.stat-item × 5
│   ├── div.topic-section → div.topic-card × 4
│   ├── div.units-section → (div.units-tabs → div.unit-tab × 11) + div.unit-detail
│   ├── div.news-group-section × 3 → (div.featured-news + div.news-list)
│   ├── div.partners-section → div.partner-logos
│   └── div.gallery-section → div.gallery-item × 14
└── div.footer
    ├── div.footer-container → (div.footer-brand + div.footer-column × 3 + div.footer-map)
    └── div.footer-copyright
```

---

## 2. Bản đồ Flexbox toàn trang

```text
PAGE
├── header-container        [FLEX row, space-between, align-items:center]
│   ├── nav                 [FLEX row, gap]
│   │   └── nav-item        [position:relative → dropdown absolute]
│   └── header-tools        [FLEX row, gap]
├── hero                    [Position + FLEX track: slide flex:0 0 100%]
├── news-slider             [FLEX row, overflow:hidden]
│   └── news-card           [FLEX column]            ← nested
├── notice-event-section    [FLEX row, 2 cột]
│   ├── notice-item         [FLEX row, space-between]
│   └── event-item          [FLEX row; badge ngày: position absolute]
├── stats-list              [FLEX row, space-around, wrap]
├── topic-section           [GRID 4 cột]  → topic-card [FLEX column]
├── units-body              [FLEX row: tabs 1 : detail 2]  ← nested
├── news-group-body × 3     [FLEX row: featured 1 : list 1]
├── partner-logos           [FLEX row, overflow / marquee]
├── gallery-section         [GRID]
└── footer-container        [FLEX row: brand 2 : column 1 : 1 : 1]
```

**Không cần Flexbox (Normal Flow):** các section xếp chồng trong `main`; `footer-column`; `notice-box`; `brand` chỉ có một ảnh logo.

---

## 3. Từng component: Layout, CSS, Content + HTML

### 3.1 Header + Navigation

- **Layout:** `header-container` → `brand | nav | header-tools`. Menu 8 mục có menu con.
- **CSS:**
```css
.header-container { display: flex; align-items: center; justify-content: space-between; }
.nav { display: flex; gap: 24px; }
.nav-item { position: relative; }
.dropdown { position: absolute; top: 100%; left: 0; display: none; }
.nav-item:hover .dropdown { display: block; }
.header-tools { display: flex; align-items: center; gap: 12px; }
```
- **HTML:** logo `a > img`; menu `nav > ul > li > a`; menu con `ul` lồng trong `li`; tìm kiếm `button` (hoặc `form + input`); VN/EN `button`/`a`.
- **Công thức:** Flex row (3 tầng lồng) + Position cho dropdown → `header > nav > ul > li > a`.

### 3.2 Hero (banner)

- **[Thấy]** 7 ảnh banner, 2 video `.mp4`, không có chữ. **[Suy luận]** slider.
- **CSS:**
```css
.hero { position: relative; overflow: hidden; }
.hero-slides { display: flex; }
.hero-slide { flex: 0 0 100%; }       /* không co, không giãn, rộng đúng 1 khung */
.hero-controls { position: absolute; bottom: 16px; }
```
- **HTML:** `section > a > img`, `video`, mũi tên/chấm là `button`.

### 3.3 Tin tức (carousel thẻ tin)

- **[Thấy]** 7 thẻ: ảnh (link), chuyên mục, tiêu đề (badge "Hot"), ngày, nút ‹ ›.
- **CSS:**
```css
.news-slider { display: flex; gap: 24px; overflow: hidden; }
.news-card { flex: 0 0 calc(25% - 18px); display: flex; flex-direction: column; }
.news-body { display: flex; flex-direction: column; gap: 8px; }
.news-date { margin-top: auto; }       /* đẩy ngày xuống đáy */
```
- **HTML:** thẻ `article`; ảnh `a > img`; chuyên mục `span`; tiêu đề `h3 > a`; ngày `time datetime="2026-11-25"`.

### 3.4 Thông báo + Sự kiện

- **[Thấy]** 6 dòng thông báo (tiêu đề + ngày); sự kiện có ảnh, badge "19 Tháng 11", tiêu đề.
- **CSS:**
```css
.notice-event-section { display: flex; gap: 32px; }
.notice-box, .event-box { flex: 1; }   /* tỷ lệ thật cần đo bằng DevTools */
.notice-item { display: flex; justify-content: space-between; gap: 16px; }
.event-item { display: flex; gap: 12px; align-items: center; }
.event-image { position: relative; }
.date-badge { position: absolute; top: 8px; left: 8px; }
```
- **HTML:** `ul > li`; tiêu đề `a`; ngày `time`; ảnh + badge `figure > img` + `time`.

### 3.5 Những con số ấn tượng

- **[Thấy]** 5 nhãn (Người học, Đối tác, SV có việc làm, Ngành học, GV Tiến sĩ). Số liệu có thể do JS đếm.
- **CSS:**
```css
.stats-list { display: flex; justify-content: space-around; flex-wrap: wrap; gap: 24px; }
.stat-item { display: flex; flex-direction: column; align-items: center; }
```
- **HTML:** `h2`; cặp nhãn–giá trị dùng `dl > div > dt + dd`.

### 3.6 Khối chuyên mục (Tuyển sinh, Đào tạo, Sinh viên, Cựu SV)

- **Layout:** lưới 4 thẻ **[Suy luận]**.
- **CSS:** Grid cho lưới đều; Flex cho bên trong thẻ.
```css
.topic-section { display: grid; grid-template-columns: repeat(4, 1fr); gap: 24px; }
.topic-card { display: flex; flex-direction: column; }
.topic-body { flex: 1; }               /* nút luôn nằm đáy thẻ */
```
- **HTML:** `article`; tiêu đề chuyên mục `h2 > a`; tiêu đề bài `h3 > a`; mô tả `p`; Đại học/Thạc sĩ/Tiến sĩ `ul > li > a`; "Đăng ký ngay" dùng **`a`** (dẫn sang trang khác), không dùng `button`.

### 3.7 Các đơn vị đào tạo, KHCN (master-detail)

- **[Thấy]** 11 đơn vị + mô tả đơn vị đang chọn + link "Khám phá". **[Suy luận]** tab.
- **CSS (Nested Flex):**
```css
.units-body { display: flex; gap: 32px; }
.units-tabs { flex: 1; display: flex; flex-direction: column; gap: 8px; }
.unit-detail { flex: 2; }              /* tỷ lệ 1 : 2 → tab 1/3, nội dung 2/3 */
```
- **HTML:** danh sách `ul > li > a` (hoặc `button role="tab"` nếu là tab thật); mô tả `p`; "Khám phá" `a`.

### 3.8 Nhóm tin (Hợp tác quốc tế, Nghiên cứu, Vì cộng đồng)

- **Layout:** mẫu lặp 3 lần: tiêu đề → tin nổi bật có ảnh + danh sách tin còn lại.
- **CSS:**
```css
.news-group-body { display: flex; gap: 32px; }
.featured-news { flex: 1; }
.news-list { flex: 1; display: flex; flex-direction: column; gap: 16px; }
```
- **HTML:** `section > h2`; nổi bật `article` (`img`, `h3 > a`, `time`); danh sách `ul > li`. Dùng chung một class cho cả 3 nhóm.

### 3.9 Mạng lưới đối tác

- **[Thấy]** tiêu đề + đoạn giới thiệu; logo không có trong nội dung (có thể JS/marquee).
```css
.partner-logos { display: flex; align-items: center; gap: 40px; overflow: hidden; }
.partner-logos img { flex: 0 0 auto; }
```
- **HTML:** `h2`, `p`, `ul > li > img`.

### 3.10 Khoảnh khắc TLU (gallery)

- **[Thấy]** 14 ảnh. **CSS:** Grid phù hợp hơn Flex vì ô đều theo hàng × cột.
```css
.gallery-section { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }
.gallery-item img { width: 100%; height: 100%; object-fit: cover; }
```
- **HTML:** `figure > img` (+ `figcaption` nếu hiển thị chú thích).

### 3.11 Footer

- **[Thấy]** logo; địa chỉ 175 Tây Sơn, P. Kim Liên, Hà Nội; email; điện thoại; 3 cột link (Giới thiệu, Truy cập nhanh, Theo dõi TLU); nút "Tải bản đồ tương tác"; bản quyền.
- **CSS:**
```css
.footer-container { display: flex; justify-content: space-between; gap: 32px; }
.footer-brand { flex: 2; }
.footer-column { flex: 1; }            /* bên trong: Normal Flow, không cần Flex */
```
- **HTML:** `footer`; địa chỉ/email/điện thoại `address` (`a href="mailto:"`, `a href="tel:"`); cột link `nav > h3 + ul > li > a`; mạng xã hội `ul > li > a` (+ `aria-label`); bản quyền `small`; "Tải bản đồ" `button` nếu mở/tải tại chỗ.

---

## 4. Flexbox hay Grid?

| Vùng | Chọn | Lý do |
|---|---|---|
| Header, tool, footer | Flex | 1 chiều, dồn trái–phải, căn giữa dọc |
| Slider hero/tin | Flex | dải ngang, phần tử không co |
| Notice+Event, Units, News-group | Flex | 2 cột có tỷ lệ |
| Card bên trong | Flex column | xếp dọc, đẩy ngày/nút xuống đáy |
| Topic 4 thẻ, Gallery | **Grid** | lưới hàng × cột đều |
| Stats 5 mục | Flex (hoặc Grid) | mục đồng nhất, cần wrap |
| Các section trong `main` | Normal Flow | vốn xếp dọc |

---

## 5. Responsive (dự kiến)

```text
Desktop: A | B | C | D        Tablet: A | B      Mobile: A
                                       C | D              B
                                                          C
                                                          D
```
```css
.units-body { display: flex; flex-wrap: wrap; }
.units-tabs, .unit-detail { flex: 1 1 300px; }
.topic-section { grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); }
@media (max-width: 768px) {
  .nav { display: none; }                        /* thay bằng mobile-menu */
  .footer-container { flex-direction: column; }
}
```

---

## 6. DOM semantic toàn trang

```text
body
├── header → a.brand > img | nav > ul > li > a (+ ul dropdown) | div.header-tools > button × 2
├── main
│   ├── section.hero
│   ├── section.news > article × 7
│   ├── section.notice-event > (section.notice > ul) + (section.event > ul > li)
│   ├── section.stats > dl
│   ├── section.topics > article × 4
│   ├── section.units > ul (tabs) + article (detail)
│   ├── section.news-group × 3 > article.featured + ul
│   ├── section.partners > ul > li > img
│   └── section.gallery > figure × 14
└── footer → address | nav × 3 > ul > li > a | p > small
```

---

## 7. Bảng tổng hợp

| Component | Layout Box | Kỹ thuật | Flex Container? | Flex Items | HTML semantic |
|---|---|---|---|---|---|
| Header | header-container | Flex row | Có | brand, nav, tools | header, nav, ul, li, a, button |
| Dropdown | dropdown | Position | Không | – | ul > li > a |
| Hero | hero-slides | Flex + Position | Có | hero-slide | section, img, video |
| Tin tức | news-slider | Flex (+ column ở card) | Có | news-card | article, img, h3, time |
| Thông báo | notice-box | Normal Flow + Flex ở item | Item: có | tiêu đề, ngày | ul, li, a, time |
| Sự kiện | event-box | Flex row (item) | Có | ảnh, tiêu đề | ul, li, figure, time |
| Con số | stats-list | Flex row | Có | stat-item | dl, dt, dd |
| Chuyên mục | topic-section | Grid + Flex trong card | Card: có | ảnh, body | article, h2, h3, a |
| Đơn vị | units-body | Flex nested | Có | tabs, detail | ul, li, a, p |
| Nhóm tin | news-group-body | Flex row | Có | featured, list | article, ul, li, time |
| Đối tác | partner-logos | Flex / marquee | Có | logo | ul, li, img |
| Gallery | gallery-section | Grid | Không | – | figure, img |
| Footer | footer-container | Flex row | Có | brand, column × 3 | footer, address, nav, ul, a |

---

## 8. Checklist cho sinh viên

```text
Nhìn thấy một vùng giao diện
        ↓
Khoanh hộp → Đặt tên theo VAI TRÒ (news-card, không phải box1)
        ↓
Xác định cha – con
        ↓
Các con đứng ngang một hàng?
   ├── Có → Flexbox
   └── Không → Có lưới hàng × cột đều?
                 ├── Có → Grid
                 └── Không → Normal Flow
        ↓
Có phần tử đè lên/bám góc phần tử khác? → Position (cha relative, con absolute)
        ↓
Xác định kiểu nội dung → chọn HTML semantic
```

**Đặt tên class:** theo mẫu `[khối]-[vai trò]` (`news-card`, `news-title`, `unit-tab`, `footer-column`). Khối lặp dùng chung một class. Tránh `box1`, `blue-box`, `left-area`.

---

## 9. Lỗi tư duy thường gặp

1. Đặt `display: flex` cho phần tử con thay vì cha.
2. Dùng Flex cho mọi thứ, kể cả danh sách dọc vốn đã đúng bằng Normal Flow.
3. Dùng Flex làm lưới 4×N làm thẻ cuối hàng bị giãn lệch. Dùng Grid.
4. Quên các lớp Flex lồng nhau (slider là hàng, card là cột).
5. Nhầm `a` và `button`: điều hướng dùng `a`, hành động tại chỗ dùng `button`.
6. Dùng `div` cho tiêu đề, ngày, danh sách thay vì `h2/h3`, `time`, `ul/li`.
7. Đặt tên theo hình dạng/màu thay vì vai trò.
8. Khẳng định chắc chắn là slider/tab/marquee khi chưa kiểm tra HTML/tương tác.

---

## 10. Cần kiểm tra bằng DevTools

1. `display` thực tế của header, thẻ tin, khối tuyển sinh/đào tạo (flex hay grid).
2. Hero có phải slider không, mỗi lần hiển thị bao nhiêu slide.
3. Khối đơn vị đào tạo là tab thật hay danh sách link.
4. Khối đối tác và gallery là grid, flex hay marquee.
