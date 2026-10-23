# Tra cứu một nhãn hiệu: theo số đơn, số bằng hoặc tên

Dùng khi người dùng hỏi tình trạng, chủ sở hữu, nhóm Nice, ngày nộp/cấp/hết hạn hoặc hồ sơ đầy đủ của **một** nhãn. Tool chính: `get_trademark_detail`, `search_trademarks`.

Câu hỏi và câu trả lời mẫu trong file này viết bằng tiếng Việt: khi dùng, dịch sang ngôn ngữ của người dùng. Giá trị tham số giữ nguyên tiếng Việt (ví dụ `status='Cấp bằng'`).

## Nhận dạng đầu vào

| Dạng | Thường trông như | Xử lý |
| --- | --- | --- |
| Số đơn | `4-2023-12345` (4 – năm – 5 chữ số) | `get_trademark_detail(id=...)` |
| Số văn bằng | `4-0123456-000`, `123456` | `get_trademark_detail(id=...)` |
| Số công báo | `CB467`, `CB467A` | việc tra công báo: `references/owner-and-gazette.md` (qua tool: `get_search_guide('gazette')`) |
| Số đơn quốc tế Madrid | số WIPO, ví dụ `1234567` | `search_gazettes(query=<số>)`; tool này không dùng cú pháp Lucene |
| Tên nhãn | "VINAMILK", "Phở Thìn" | `search_trademarks(query=...)` như dưới đây |

Người dùng có thể gõ thiếu dấu gạch, thừa khoảng trắng hoặc khác số chữ số; chuẩn hóa trước khi gọi. Detail trả `found=false` thì thử `search_trademarks(query=<số>)` một lần rồi dừng.

## Tra theo tên

`query` của `search_trademarks` là câu truy vấn Lucene với toán tử mặc định OR: `minh an` = `minh OR an`, trả về rất nhiều nhãn không liên quan. Khi chỉ cần tìm đúng nhãn người dùng nói:

- Tên nhiều từ: đặt trong ngoặc kép, chữ thường không dấu: `query='"pho thin"'` (khớp cả "PHỞ THÌN" và "PHO THIN").
- Tên một từ: dùng nguyên từ, chữ thường: `query='vinamilk'`.
- Tên có ký tự đặc biệt (`& / : + - ( ) !`…): thay bằng khoảng trắng trong ngoặc kép: "C/O" → `'"c o"'`.
- Có nhiều nhãn cùng tên ở nhiều nhóm hoặc nhiều chủ: thu hẹp bằng `nice_class` hoặc `owner='"tên riêng"'` nếu người dùng cho biết.

Kết quả gồm cả nhãn chứa thêm từ khác ("PHỞ THÌN BỜ HỒ"). Không thấy nhãn đúng tên thì nói rõ, không tự mở rộng sang tìm tương tự; nếu người dùng muốn, đó là việc tìm nhãn tương tự: `references/similarity-search.md` (qua tool: `get_search_guide('similarity')`).

## Tham số lọc thường dùng

- `status`: luôn truyền nhãn **tiếng Việt**: `Cấp bằng`, `Đang giải quyết`, `Từ chối`. Nhóm đã hết hiệu lực (rút đơn, hết hạn, mất hiệu lực, từ bỏ): `Hết hiệu lực`. Nhiều giá trị cách nhau bằng dấu phẩy. Ý nghĩa từng nhóm: `get_reference_categories(category_type='Status')`.
- `registration_number`: tìm theo số bằng khi `get_trademark_detail` không nhận dạng được.
- Ngày theo `YYYY-MM-DD`.

## Đọc hồ sơ chi tiết

`get_trademark_detail` trả về: chủ đơn và địa chỉ, đại diện SHCN, danh mục sản phẩm/dịch vụ theo nhóm Nice, tiến trình xử lý đơn, ngày nộp/công bố/cấp/hết hạn, mẫu nhãn, yếu tố loại trừ, dữ liệu ưu tiên, Madrid.

- "Còn hiệu lực không": dựa vào trạng thái **và** ngày hết hạn so với ngày hiện tại. `Cấp bằng` nhưng đã qua ngày hết hạn mà không có gia hạn trong tiến trình thì nói rõ là có thể đã hết hiệu lực, cần kiểm tra thêm.
- Diễn biến gần nhất: lấy các bước cuối trong tiến trình, kèm số văn bản và link tài liệu nếu có.
- Không dịch hay diễn giải lại danh mục sản phẩm/dịch vụ; trích nguyên văn theo nhóm.

## Ví dụ

**Hỏi:** "Check status of 4-2021-01234"

1. `get_trademark_detail(id='4-2021-01234')`
2. Trả lời bằng tiếng Anh: trạng thái, chủ sở hữu, nhóm, các ngày quan trọng, diễn biến gần nhất, link mẫu nhãn `[view](url)`, deep-link.

**Hỏi:** "Nhãn Phở Thìn của ai?"

1. `search_trademarks(query='"pho thin"')`
2. Nếu có nhiều chủ sở hữu khác nhau, liệt kê tất cả kèm nhóm và trạng thái; `get_trademark_detail` cho nhãn người dùng quan tâm nhất nếu cần thêm thông tin.
