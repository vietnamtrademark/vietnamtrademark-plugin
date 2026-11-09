# Báo cáo kết quả tra cứu chuyên sâu và khả năng bảo hộ

Dùng khi người dùng (thường là chuyên viên sở hữu trí tuệ) muốn một báo cáo gửi khách hàng: nhãn X cho sản phẩm/dịch vụ Y có khả năng được bảo hộ ở mức nào, đối chứng là những nhãn nào. File này chỉ nói về **đánh giá và viết báo cáo**; việc tìm đối chứng làm theo `references/similarity-search.md` trước (thang bậc 0–6, so sánh dấu hiệu, so sánh hàng hóa, chọn đối chứng).

Câu mẫu trong file này viết bằng tiếng Việt; báo cáo viết bằng ngôn ngữ người dùng yêu cầu, mặc định tiếng Việt.

## Dữ liệu cần có trước khi viết

1. Kết quả các bậc tìm theo `similarity-search.md`, kèm nguyên văn từng câu truy vấn.
2. `get_trademark_detail` cho từng nhãn sẽ đưa vào phần A (tối đa 5 nhãn, ưu tiên nhãn gây rủi ro cao nhất), để lấy: ngày công bố, ngày cấp, ngày hết hạn, màu sắc, phân loại hình, yếu tố loại trừ, chủ đơn và địa chỉ, đại diện SHCN, tiến trình xử lý, danh mục theo nhóm.
3. Sản phẩm/dịch vụ Y và nhóm Nice chính. Nếu người dùng cho biết khách hàng đã kinh doanh hay chưa, dùng cho phần đề xuất.

## Thang khả năng bảo hộ (6 mức)

| Mức | Tỷ lệ | Tiêu chí |
| --- | --- | --- |
| KHÔNG ĐẠT | 0% | Nhãn thuộc đối tượng không được bảo hộ dưới danh nghĩa nhãn hiệu (xem bước *Dấu hiệu có đăng ký được không* trong `similarity-search.md`). |
| THẤP | <40% | Trùng hoặc tương tự gây nhầm lẫn **rõ ràng** với nhãn có trước; hoặc chính nhãn không đáp ứng điều kiện bảo hộ độc quyền; nguy cơ từ chối cao. |
| TRUNG BÌNH | 55-60% | Có **một số** nhãn đối chứng có trước tương tự; tiềm ẩn rủi ro từ chối; kết quả phụ thuộc quan điểm thẩm định viên. |
| KHÁ | 65-70% | Nhãn có mức độ phân biệt nhất định, nhưng vẫn có nhãn đối chứng **có thể** tương tự cần cân nhắc; tương đối khả quan. |
| CAO | 75-80% | Phân biệt rõ ràng với nhãn có trước; không phát hiện dấu hiệu tương tự **đáng kể**; rủi ro từ chối thấp. |
| RẤT CAO | >85% | Phân biệt mạnh; **không** phát hiện nhãn có trước nào tương tự; gần như chắc chắn được bảo hộ. |

Cách ghi: chữ in hoa kèm nguyên khoảng tỷ lệ của mức, ví dụ `TRUNG BÌNH, 55-60%`. Thang gồm các mức rời rạc; không tự tính một con số phần trăm khác.

Quan hệ với thang rủi ro 3 mức của `similarity-search.md` (gợi ý, không phải ánh xạ cứng):

| Rủi ro | Khả năng bảo hộ |
| --- | --- |
| Dấu hiệu không đăng ký được | KHÔNG ĐẠT |
| Cao | THẤP |
| Trung bình, nhiều đối chứng hoặc hàng hóa trùng | TRUNG BÌNH |
| Trung bình, ít đối chứng, tương tự xa hoặc hàng hóa chỉ liên quan | KHÁ |
| Thấp, còn nhãn tương tự ở mức nhỏ | CAO |
| Không tìm thấy nhãn tương tự sau khi chạy đủ thang | RẤT CAO |

### Chọn mức

Không có ngưỡng định lượng (số đối chứng, điểm tương tự). Chọn mức sau khi cân nhắc tổng thể, và **ghi lý do** trong báo cáo:

1. Mức tương tự dấu hiệu của từng đối chứng (theo mục *So sánh dấu hiệu* trong `similarity-search.md`).
2. Quan hệ hàng hóa/dịch vụ: trùng, tương tự, liên quan xa, khác.
3. Tình trạng pháp lý: cấp bằng còn hiệu lực, đang giải quyết, chưa công bố, hết hiệu lực chưa quá 3 năm, từ chối còn thời hiệu khiếu nại.
4. Số đối chứng cùng hướng rủi ro: nhiều nhãn tương tự ở mức vừa có thể cộng lại thành rủi ro cao hơn một nhãn đơn lẻ.
5. Khả năng phân biệt của chính X: từ tự đặt (mạnh) hay từ thông dụng, mô tả (yếu); phần đặc trưng của X có bị loại trừ bảo hộ ở đối chứng nào không.

Ràng buộc:

- Lý do nêu cụ thể từng đối chứng quyết định mức: số đơn, quan hệ dấu hiệu, quan hệ hàng hóa. Không viết chung chung.
- Mức chọn phải khớp tiêu chí trong bảng: đã có đối chứng trùng dấu hiệu, trùng hàng hóa và còn hiệu lực thì không chọn từ KHÁ trở lên.
- Phân vân giữa hai mức liền kề: chọn mức thấp hơn và nói trong lý do là đang ở ranh giới.

## Cấu trúc báo cáo

Không in số hồ sơ, logo hay màu sắc của đơn vị tư vấn. Mẫu nhãn của X (nếu người dùng cung cấp) và link mẫu nhãn của đối chứng vẫn đưa vào, vì đó là nội dung tra cứu.

### Đầu trang

- `Ngày/tháng/năm tra cứu: dd/mm/yyyy`
- Tiêu đề: `KẾT QUẢ TRA CỨU CHUYÊN SÂU NHÃN HIỆU`

### Bảng tóm tắt

| Dòng | Nội dung |
| --- | --- |
| Mẫu nhãn hiệu | Chữ của X; mô tả hình nếu người dùng cung cấp |
| Sản phẩm/dịch vụ tra cứu¹ | `Nhóm <n>: <danh mục>` |
| Khả năng bảo hộ | Mức và tỷ lệ; lý do chọn mức; câu dẫn; danh sách đối chứng gây rủi ro trực tiếp, mỗi nhãn ghi tên, "(Đối chứng i)" và ghi chú trạng thái nếu đặc biệt ("nộp ngày …, chưa công bố") |
| Lưu ý | Ghi chú khác (ví dụ danh mục cần làm rõ, nhãn nổi tiếng ở nhóm khác phát hiện ở bậc 0), hoặc `-` |
| Đề xuất và khuyến nghị | Hành động khuyến nghị cho khách hàng |

¹ Chú thích cuối trang: "Danh mục nhóm sản phẩm/dịch vụ được liệt kê nêu trên chỉ phục vụ cho mục đích tra cứu và CÓ THỂ phải được điều chỉnh, làm rõ trước khi nộp đơn để tránh từ chối về mặt hình thức."

Danh sách trong bảng tóm tắt chỉ gồm đối chứng gây rủi ro trực tiếp; phần A liệt kê đủ các đối chứng chủ yếu đã xem xét.

### Câu dẫn và đề xuất

Mẫu cho mức TRUNG BÌNH, dùng nguyên văn:

> Tuỳ thuộc vào quan điểm của từng thẩm định viên mà Nhãn hiệu của Quý Khách có thể bị từ chối do tương tự gây nhầm lẫn với các nhãn hiệu sau đây:

> Nếu chưa triển khai kế hoạch kinh doanh, Quý Khách nên cân nhắc lựa chọn nhãn hiệu khác để tra cứu và đăng ký. Trường hợp vẫn muốn nộp đơn, chúng tôi sẽ tư vấn phương án trả lời nếu có từ chối và chi phí phát sinh tương ứng.

Các mức khác: tự viết, giữ giọng văn của mẫu (xưng "chúng tôi", gọi "Quý Khách", câu trung tính, không cam kết kết quả thẩm định), nội dung bám mức đã chọn:

- **KHÔNG ĐẠT:** không có danh sách đối chứng; câu dẫn nêu căn cứ dấu hiệu không được bảo hộ; đề xuất đổi dấu hiệu hoặc bỏ phần bị loại trừ.
- **THẤP:** câu dẫn nói rõ nguy cơ bị từ chối cao do trùng/tương tự rõ ràng với các nhãn liệt kê; đề xuất chọn nhãn khác hoặc thay đổi phần đặc trưng.
- **KHÁ:** câu dẫn nêu các nhãn có thể tương tự cần cân nhắc; đề xuất có thể nộp đơn, kèm cách giảm rủi ro (thu hẹp danh mục, thêm yếu tố phân biệt).
- **CAO, RẤT CAO:** câu dẫn nói chưa phát hiện (hoặc không phát hiện) nhãn tương tự đáng kể; đề xuất nộp đơn sớm để giữ ngày ưu tiên. Không có đối chứng thì bỏ danh sách.

Đề xuất gắn với hoàn cảnh khách hàng khi biết (đã kinh doanh hay chưa). Không viết câu khẳng định chắc chắn được hoặc không được bảo hộ.

### Phần A. Danh sách nhãn hiệu đối chứng chủ yếu

Mỗi đối chứng một khối `ĐỐI CHỨNG i`, bảng 3 cột `INID | Tên mục dữ liệu | Nội dung`, chỉ in trường có dữ liệu. Trường lấy từ `get_trademark_detail` (và kết quả `search_trademarks`):

| INID | Tên mục | Nguồn trong tool |
| --- | --- | --- |
| (210) | Số đơn | Số đơn |
| (220) | Ngày nộp đơn | Ngày nộp đơn |
| (111) | Số bằng | Số bằng, nếu đã cấp |
| | Ngày cấp bằng | Ngày cấp bằng |
| (181) | Thời hạn hiệu lực | Ngày hết hạn |
| (441) | Công bố A | Ngày công bố |
| (540) | Nhãn hiệu | Tên nhãn; thêm ", hình" khi có phân loại hình |
| (550) | Loại nhãn hiệu | Loại đơn 1/2 nếu tool trả về |
| (591) | Màu sắc | Màu sắc |
| (731)/(732) | Người nộp đơn / Chủ GCN | Chủ đơn, địa chỉ chủ đơn |
| (740) | Đại diện | Đại diện SHCN |
| (531) | Phân loại hình | Phân loại hình (mã Vienna) |
| | Tiến trình | Các bước trong tiến trình xử lý, kèm ngày; ghi riêng bước từ chối, lý do từ chối và đối chứng của Cục nếu có |
| | Giới hạn | Yếu tố loại trừ |
| | Mẫu nhãn | Link `[xem](url)` từ dòng "Mẫu nhãn" |
| (511) | Nhóm SP/DV | Nhóm Nice và danh mục theo từng nhóm, trích nguyên văn |
| | Deep-link VNTM | Deep-link tool trả về |

Ghi chú đặc biệt đặt ngay dưới tiêu đề khối, ví dụ "đơn đăng ký mới được nộp và chưa được công bố". Trường tool không trả (số công báo, mã tiến trình kiểu "221(26/10/2022)") thì bỏ, không bịa.

### Phần B. Nguồn, phạm vi và quy ước tra cứu

1. **Quy ước về khả năng bảo hộ:** in bảng thang 6 mức ở trên.
2. **Nguồn tra cứu:** cơ sở dữ liệu VNTM (vietnamtrademark.net), gồm đơn đăng ký nhãn hiệu quốc gia và văn bằng đã cấp tại Cục Sở hữu trí tuệ, cùng đơn quốc tế có chỉ định Việt Nam đã công bố (Madrid), cập nhật theo từng số công báo. Ghi ngày tra cứu; không có ngày chốt dữ liệu riêng theo nguồn thì không tự đặt ra. Nêu rõ cơ sở dữ liệu **không** bao gồm: đơn nộp ở nước ngoài có yêu cầu hưởng quyền ưu tiên nhưng chưa nộp vào Việt Nam; đơn đã nộp tại Cục SHTT nhưng chưa có trong dữ liệu tại ngày tra cứu; tác phẩm được bảo hộ quyền tác giả của người khác.
3. **Phạm vi đã tra:** nguyên văn từng câu truy vấn, nhóm Nice và lựa chọn phạm vi nhóm, các bậc đã bỏ qua và lý do, số lượt đã dùng. Tra cứu chỉ dựa trên chữ; phần hình của nhãn chưa được so sánh trừ khi nêu khác.
4. **Phạm vi của kết quả:** kết quả có thể khác thẩm định thực tế do (a) khác biệt quan điểm giữa các thẩm định viên; (b) dữ liệu tại Cục SHTT có sai sót hoặc chưa cập nhật; (c) thay đổi quy định pháp luật hoặc thực tiễn thẩm định; (d) tình tiết ngoài dự kiến như phản đối cấp, tranh chấp, thông tin cung cấp không đầy đủ.

### Phần C. Bảo lưu và miễn trừ

1. Kết quả chỉ mang tính tham khảo trong phạm vi tra cứu, không phải kết luận chính thức của Cục Sở hữu trí tuệ; không khẳng định chắc chắn khả năng bảo hộ.
2. Không chịu trách nhiệm về tính hợp lệ, chính xác của tài liệu, thông tin khách hàng cung cấp.
3. Giải thích, tư vấn chỉ nhằm đánh giá khả năng bảo hộ, không loại trừ rủi ro kinh doanh thực tế; khuyến nghị tham vấn chuyên môn khi điều chỉnh nhãn.
4. Báo cáo chuẩn bị riêng cho khách hàng; miễn trừ trách nhiệm khi dùng cho mục đích khác.

## Ví dụ rút gọn

Nhãn **La Solie**, nhóm 25: quần áo. Sau thang tìm (xem ví dụ cuối `similarity-search.md`): không có nhãn trùng; ba nhãn tương tự cho hàng hóa trùng còn hiệu lực hoặc đang giải quyết (LA SOIE DU VIETNAM 4-2026-52162 chưa công bố, LASOL 4-2022-38159, MAILÉI SOLIE 4-2025-20191); hai nhãn khác liệt kê ở phần A nhưng không đưa vào kết luận (DE LA SOIE có yếu tố loại trừ "DE LA SOIE"; SOLIS tương tự xa hơn).

Kết luận: **TRUNG BÌNH, 55-60%**. Lý do: không có nhãn trùng hoàn toàn nên không ở mức THẤP; nhưng MAILÉI SOLIE chứa nguyên phần đặc trưng "SOLIE", LA SOIE DU VIETNAM chỉ khác một ký tự và LASOL trùng phần đầu khi đọc liền, cả ba đều đăng ký quần áo nhóm 25, nên không ở mức KHÁ. Câu dẫn và đề xuất dùng nguyên mẫu TRUNG BÌNH.
