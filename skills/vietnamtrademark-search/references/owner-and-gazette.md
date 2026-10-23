# Danh mục nhãn của chủ đơn và theo dõi công báo

Dùng khi người dùng muốn liệt kê nhãn của một công ty/cá nhân, theo dõi đơn mới công bố, văn bằng mới cấp, chuyển nhượng, gia hạn, đính chính, Madrid của một chủ đơn hoặc đại diện SHCN, hoặc hỏi về một số công báo. Tool chính: `search_trademarks` (danh mục hiện tại), `search_gazettes` và `get_gazette_detail` (diễn biến theo thời gian).

Câu hỏi và câu trả lời mẫu trong file này viết bằng tiếng Việt: khi dùng, dịch sang ngôn ngữ của người dùng. Giá trị tham số giữ nguyên tiếng Việt (ví dụ `status='Cấp bằng'`, `notice_type='Đơn nộp mới'`).

## Chọn tool

| Người dùng hỏi | Gọi |
| --- | --- |
| Công ty Z có những nhãn nào / bao nhiêu nhãn ở nhóm N / nhãn nào sắp hết hạn | `search_trademarks(owner=..., nice_class=..., status=..., *_date_from/to=...)` |
| Đơn/nhãn được công bố ở số công báo nào, ngày nào | `search_gazettes(application_number=...)` hoặc `registration_number=...` |
| Tháng này Z có đơn mới / văn bằng mới / chuyển nhượng / gia hạn gì | `search_gazettes(owner=... hoặc representative=..., publication_date_from/to=..., gazette_type=...)` |
| Nội dung một số công báo | `search_gazettes(gazette_number='CB467A', ...)` |
| Toàn văn một thông báo (quyết định, chuyển nhượng, đính chính) | `get_gazette_detail(id=...)` |

## Danh mục theo chủ đơn: `search_trademarks`

- `owner` là câu truy vấn Lucene, toán tử mặc định OR. Bỏ loại hình doanh nghiệp ("Công ty TNHH", "Công ty Cổ phần", "JSC", "Co., Ltd"…), giữ phần tên riêng, **đặt trong ngoặc kép**, chữ thường không dấu: "Công ty TNHH Thực phẩm Minh An" → `owner='"minh an"'`. Viết `owner='Minh An'` sẽ khớp mọi chủ đơn có chữ "an".
- Tên riêng quá phổ biến (nhiều công ty "Minh An"): thêm từ đặc trưng khác trong tên làm điều kiện bắt buộc: `owner='"minh an" +"thuc pham"'`.
- Tên viết liền hoặc có hậu tố thay đổi: dùng `*` trong một từ, ví dụ `owner='vingroup* OR "vin group"'`.
- `status`: nhãn tiếng Việt (`Cấp bằng`, `Đang giải quyết`, `Từ chối`, `Hết hiệu lực`), nhiều giá trị cách nhau bằng dấu phẩy.
- Nhãn sắp hết hạn: không có tham số ngày hết hạn. Văn bằng nhãn hiệu có hiệu lực 10 năm tính từ ngày nộp đơn, nên lọc `status='Cấp bằng'` với `filing_date_from/to` lùi khoảng 10 năm (hoặc 20, 30 năm nếu tính cả gia hạn), rồi đọc ngày hết hạn trong kết quả.
- Danh mục lớn: dùng `total` để trả lời "bao nhiêu"; muốn liệt kê thì chia theo `nice_class` hoặc khoảng ngày thay vì lật trang.

## Công báo: `search_gazettes`

`search_gazettes` **không** dùng cú pháp Lucene. Mọi từ trong `owner`, `representative`, `query` đều bắt buộc có mặt (AND), có dấu hay không dấu đều khớp; `* ? ~ " +` không có tác dụng. Chỉ đưa phần tên riêng, không cần ngoặc kép.

- `query`: tìm nhanh theo chủ đơn, đại diện, số đơn (khớp tiền tố) hoặc số văn bằng. **Không** tìm theo tên nhãn; muốn theo tên nhãn thì tìm số đơn bằng `search_trademarks` trước, rồi tra công báo theo `application_number`.
- `gazette_type`: `A` = đơn mới công bố, sửa đổi; `B` = văn bằng cấp mới, Madrid, chuyển nhượng, gia hạn, đính chính.
- `gazette_part`: `PHẦN I` … `PHẦN V`; `notice_type`: ví dụ `Đơn nộp mới`, `Cấp Giấy chứng nhận đăng ký`, `Madrid`.
- `gazette_number`: `CB467`, `CB467A`, `CB467B`.
- `publication_date_from/to`: `YYYY-MM-DD`. Người dùng không nói khoảng ngày khi "theo dõi": mặc định 90 ngày gần nhất và nói rõ.

### Quy trình theo dõi

1. `search_gazettes(owner=..., publication_date_from=..., publication_date_to=...)`. Không có kết quả và tên giống một tổ chức đại diện SHCN (công ty luật, văn phòng sở hữu trí tuệ): gọi lại với `representative=...`.
2. Kết quả nhiều hơn một trang: thu hẹp bằng `gazette_type` hoặc chia khoảng ngày, không lật trang sâu.
3. Thông báo cần toàn văn: `get_gazette_detail(id=...)`, tối đa 3 lượt.
4. Trả lời dạng bảng nhóm theo số công báo và loại thông báo: số đơn/số bằng, tên nhãn, nhóm Nice, ngày công bố, deep-link. Nêu rõ khoảng ngày đã bao phủ và danh sách đã đầy đủ chưa.

## Ví dụ

**Hỏi:** "Công ty CP Sữa Việt Nam có bao nhiêu nhãn đã cấp bằng ở nhóm 29?"

1. `search_trademarks(owner='"sua viet nam"', nice_class='29', status='Cấp bằng')`
2. Trả lời theo `total`, liệt kê trang đầu, nói rõ chủ đơn đã khớp (có thể gồm công ty khác cùng tên; kiểm tra cột chủ sở hữu).

**Hỏi:** "Theo dõi đơn mới của Minh An trong quý 3/2026"

1. `search_gazettes(owner='Minh An', gazette_type='A', publication_date_from='2026-07-01', publication_date_to='2026-09-30')`
2. Bảng theo số công báo; nêu khoảng ngày và số lượt đã dùng.
