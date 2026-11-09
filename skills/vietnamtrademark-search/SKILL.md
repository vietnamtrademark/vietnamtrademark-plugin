---
name: vietnamtrademark-search
description: 'Tra cứu nhãn hiệu và công báo sở hữu công nghiệp Việt Nam qua VietnamTrademark MCP (vietnamtrademark.net). Dùng khi người dùng hỏi về một nhãn hiệu/thương hiệu/tên thương mại cụ thể tại Việt Nam (đã đăng ký chưa, của ai, nhóm Nice nào, tình trạng, ngày nộp/cấp/hết hạn), muốn tìm nhãn trùng hoặc tương tự, kiểm tra tên trước khi nộp đơn, tra theo số đơn hoặc số văn bằng, liệt kê nhãn của một chủ đơn, hoặc hỏi về công báo, Madrid, chuyển nhượng, gia hạn. | Look up Vietnamese trademarks and the IP gazette through VietnamTrademark MCP. Use whenever the user asks about a specific trademark/brand in Vietnam, wants identical or similar marks, a pre-filing conflict check, gives an application or registration number, lists an owner''s portfolio, or asks about gazette notices, Madrid, assignments or renewals.'
---

# Tra cứu nhãn hiệu Việt Nam (VNTM)

Skill này dùng các tool của MCP server `vietnamtrademark` (trong Claude Code: `mcp__vietnamtrademark__<tên tool>`) để trả lời câu hỏi về nhãn hiệu Việt Nam bằng dữ liệu thật. Mọi tool đều chỉ đọc.

File này chứa nguyên tắc chung và bảng định tuyến. Hướng dẫn chi tiết cho từng loại việc nằm trong thư mục `references/`; **chỉ đọc file tương ứng với việc đang làm**, đọc trước khi gọi tool.

## Nguyên tắc dùng dữ liệu

1. Tình trạng, chủ sở hữu, nhóm Nice và ngày tháng của nhãn hiệu Việt Nam thay đổi theo từng số công báo, kể cả với thương hiệu nổi tiếng. Kết quả tool VNTM phản ánh hồ sơ hiện hành, nên câu trả lời về một nhãn, số đơn, số văn bằng hoặc thông báo công báo cụ thể dựa trên kết quả tool.
2. Khi tool báo lỗi, thông điệp lỗi đi kèm `code` và `message` để nêu lại cho người dùng. Lỗi tham số có hướng dẫn sửa (ví dụ cú pháp truy vấn) thường chỉ cần sửa theo hướng dẫn và gọi lại một lần.
3. `found=false` nghĩa là cơ sở dữ liệu VNTM không có bản ghi. Không phải lỗi tham số, và **không** có nghĩa tên đó còn trống.
4. Mỗi bản ghi có deep-link VNTM để người dùng mở hồ sơ gốc.
5. Dữ liệu, nhãn trường và thông điệp lỗi của VNTM đều bằng tiếng Việt; deep-link luôn là link mặc định (không có tiền tố /en hay /zh). Server không có tham số ngôn ngữ; câu mẫu trong skill viết bằng tiếng Việt.
6. Câu hỏi chung về luật, thủ tục, lệ phí sở hữu trí tuệ không gắn với nhãn cụ thể nằm ngoài phạm vi dữ liệu VNTM.

## Hạn mức

- Mỗi lượt `search_*` hoặc `get_*_detail` thành công trừ 1 lượt của người dùng. `get_reference_categories` và `get_search_guide` miễn phí. Lượt bị từ chối vì lỗi tham số không bị trừ.
- Dùng `page_size` lớn nhất server cho phép thay vì lật trang; thu hẹp bằng điều kiện lọc.
- Ngân sách mặc định cho một yêu cầu: **tối đa 4 lượt search và 3 lượt detail**. Riêng tìm nhãn trùng/tương tự theo thang trong `references/similarity-search.md`: **tối đa 7 lượt search**. Lập báo cáo khả năng đăng ký theo `references/registrability-report.md`: thang tìm tương tự cộng **tối đa 5 lượt detail** cho các nhãn đối chứng chủ yếu. Cần nhiều hơn thì nói ước tính số lượt và hỏi người dùng trước.

## Định tuyến theo ý định

| Người dùng muốn | Dấu hiệu | Đọc |
| --- | --- | --- |
| Tìm nhãn trùng/tương tự, kiểm tra tên trước khi nộp đơn, đánh giá rủi ro | "có trùng không", "đăng ký X cho Y được không", "nhãn tương tự", "giống", "gần giống", có toán tử `~ * ?` | `references/similarity-search.md` |
| Lập **báo cáo** kết quả tra cứu chuyên sâu, đánh giá khả năng bảo hộ theo thang 6 mức cho khách hàng | "lập báo cáo", "kết quả tra cứu chuyên sâu", "khả năng bảo hộ bao nhiêu phần trăm", "đánh giá khả năng đăng ký để gửi khách hàng" | `references/similarity-search.md` để tìm, rồi `references/registrability-report.md` để đánh giá và viết báo cáo |
| Tình trạng, chủ sở hữu, ngày tháng, hồ sơ của **một** nhãn | số đơn `4-YYYY-NNNNN`, số bằng, "X đã đăng ký chưa", "của ai", "còn hiệu lực không" | `references/record-lookup.md` |
| Danh mục nhãn của một chủ đơn; đơn mới, văn bằng, chuyển nhượng, gia hạn trên công báo | tên công ty/đại diện SHCN, "theo dõi", "công báo", số `CB467`, Madrid, khoảng ngày | `references/owner-and-gazette.md` |
| Mã nhóm Nice của sản phẩm, ý nghĩa trạng thái | "nhóm mấy", "trạng thái X là gì" | gọi thẳng `get_reference_categories` |

Một yêu cầu có thể cần nhiều file (ví dụ tra hồ sơ rồi tìm nhãn tương tự với nó): đọc lần lượt, ngân sách lượt vẫn tính chung.

### Khi nào hỏi lại người dùng

Đoán ý định khi dấu hiệu rõ, và nói ngắn gọn đã hiểu thế nào ở đầu câu trả lời. **Chỉ hỏi** khi hai cách hiểu dẫn tới cách dùng lượt khác nhau, điển hình là người dùng chỉ đưa một cái tên ("Minh An", "tra giúp VINAMILK"). Khi đó hỏi một câu, đưa lựa chọn kèm số lượt ước tính, ví dụ:

> Anh/chị muốn: (1) xem tình trạng nhãn "MINH AN" đã đăng ký (khoảng 1–2 lượt), hay (2) tìm các nhãn trùng/tương tự để đánh giá rủi ro trước khi nộp đơn (khoảng 3–6 lượt)? Nếu là (2), cho biết thêm sản phẩm/dịch vụ dự định dùng.

Không hỏi khi người dùng đã đưa số đơn, số bằng, hoặc nói rõ mục đích.

Riêng việc tìm nhãn trùng/tương tự có thêm **câu hỏi về phạm vi nhóm**: có mở rộng sang nhóm liên quan hay không là quyết định chuyên môn của người dùng, nên thường được hỏi trước lượt tìm đầu tiên (chi tiết và mẫu câu hỏi ở `references/similarity-search.md`). Gộp chung với các câu hỏi trên nếu có, để chỉ hỏi một lần.

## Trình bày câu trả lời

- Câu đầu là kết luận: đã đăng ký / không có bản ghi trong VNTM / tìm thấy N nhãn trùng hoặc tương tự.
- Bảng ngắn: tên nhãn, số đơn, số bằng, chủ sở hữu, nhóm Nice, trạng thái, ngày quan trọng, mẫu nhãn, deep-link.
- Mẫu nhãn: dẫn dạng link `[xem](url)` từ dòng "Mẫu nhãn" tool trả về, không nhúng ảnh, vì nhiều client không hiển thị ảnh ngoài. Kết quả `search_trademarks` đã có link này, không gọi `get_trademark_detail` chỉ để lấy logo. Bạn không nhìn thấy ảnh, nên không nhận xét phần hình của nhãn; để người dùng tự xem.
- Phạm vi đã tra: **nguyên văn từng câu truy vấn** đã gửi, nhóm, khoảng ngày, số lượt đã dùng; nói rõ nếu kết quả bị cắt do trần phân trang.

## Nếu không thấy file trong `references/`

Skill có thể được cài thiếu thư mục `references/`, hoặc bạn đang đọc file này qua tool `get_search_guide`. Khi đó gọi `get_search_guide` (miễn phí) với `topic` tương ứng: `similarity` → `references/similarity-search.md`, `record` → `references/record-lookup.md`, `gazette` → `references/owner-and-gazette.md`. File `references/registrability-report.md` chưa có `topic` riêng, chỉ tải được qua URL. Các file này đều có tại `https://vietnamtrademark.net/mcp-skill/vietnamtrademark-search/file?path=references/<tên file>` (danh sách đầy đủ: `https://vietnamtrademark.net/mcp-skill/vietnamtrademark-search/files`). Các nguyên tắc trong file này áp dụng chung cho mọi chủ đề.

## Nếu không thấy tool của server `vietnamtrademark`

MCP server chưa được cấu hình hoặc chưa kết nối. Cách kết nối: trên Claude (web, Desktop, mobile) thêm connector VietnamTrademark.net; trên Claude Code và các công cụ lập trình khác, lấy API key ở trang "Cài đặt MCP" trên vietnamtrademark.net và thêm server `vietnamtrademark` vào cấu hình MCP (hướng dẫn tại `https://vietnamtrademark.net/mcp-guide`). Khi chưa có dữ liệu VNTM, thông tin về một nhãn cụ thể chưa được kiểm chứng với hồ sơ hiện hành.

## Prompt dựng sẵn

Server có 3 prompt (trong Claude Code gõ `/mcp__vietnamtrademark__<tên>`), mỗi prompt ứng với một file tham chiếu: `check_trademark name [goods]` → `similarity-search.md`; `trademark_status number` → `record-lookup.md`; `monitor_gazette owner [date_from] [date_to]` → `owner-and-gazette.md`. Mỗi prompt nêu các bước của tình huống đó; hướng dẫn trong file tương ứng và ngân sách lượt ở trên áp dụng kèm.
