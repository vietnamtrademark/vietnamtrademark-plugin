# Tìm nhãn trùng, tương tự và kiểm tra trước khi nộp đơn

Dùng khi người dùng muốn biết tên X có trùng hoặc tương tự nhãn đã có không, muốn đăng ký X cho sản phẩm/dịch vụ Y, hoặc tự đưa câu truy vấn có `~ * ?`. Tool chính: `search_trademarks`, sau đó `get_trademark_detail` cho vài nhãn gần nhất.

Câu hỏi và câu trả lời mẫu trong file này viết bằng tiếng Việt: khi dùng, dịch sang ngôn ngữ của người dùng. Giá trị tham số giữ nguyên tiếng Việt (ví dụ `status='Cấp bằng'`).

## Cú pháp `query` và `owner` của `search_trademarks` (Lucene)

`query` và `owner` được đưa nguyên văn vào Lucene Classic QueryParser (Lucene 4.8), không escape. Chuyên viên sở hữu trí tuệ dùng chính cú pháp này trên VNTM để tìm nhãn tương tự, nên agent cũng phải dùng đúng. Cú pháp này **không** áp dụng cho `search_gazettes`.

### Dữ liệu được đánh chỉ mục thế nào

- Trường tên nhãn chứa cùng lúc: tên tiếng Việt có dấu, **tên đó bỏ dấu**, tên tiếng Anh và số đơn. Văn bản được tách thành từng từ theo khoảng trắng và dấu câu, chuyển chữ thường, **giữ nguyên dấu**.
  - Gõ không dấu `pho thin` khớp cả "PHỞ THÌN" lẫn "PHO THIN". Gõ có dấu `phở thìn` chỉ khớp bản có dấu. **Khi tìm nhãn tương tự, luôn dùng chữ thường không dấu.**
  - Mỗi từ là một đơn vị: `minhan` không khớp "MINH AN" và ngược lại. Phải thử cả dạng viết liền lẫn dạng tách.
- Kết quả xếp theo độ liên quan (điểm Lucene): nhãn khớp nhiều điều kiện hơn, khớp chính xác hơn đứng trước.

### Toán tử mặc định là OR

`minh an coffee` nghĩa là `minh OR an OR coffee`: mọi nhãn có chữ "coffee" đều khớp. Trang đầu vẫn ưu tiên nhãn khớp nhiều từ, nhưng `total` phình to và nhãn liên quan có thể bị đẩy khỏi trang. Muốn bắt buộc từ phải có, dùng `+` hoặc `AND`; muốn đúng cụm, dùng ngoặc kép.

### Bảng toán tử

| Cú pháp | Ý nghĩa | Ví dụ | Bắt được |
| --- | --- | --- | --- |
| `"a b"` | Cụm từ, đúng thứ tự, liền nhau | `"minh an"` | MINH AN, MINH AN FOOD |
| `+a` hoặc `a AND b` | Từ bắt buộc phải có | `+minh +an` | AN MINH, MINH PHÁT AN |
| `-a` hoặc `NOT a` | Loại trừ | `sun* -sunhouse` | lọc nhiễu; không dùng `-` đứng một mình |
| `a OR b`, `( )` | Gộp nhiều biến thể trong **một** lượt gọi | `minhan OR "minh an"` | |
| `*` | 0 hoặc nhiều ký tự trong một từ | `vina*`, `*mart`, `vina*milk` | VINACAFE; BIGMART; VINAMILK, VINASOYMILK |
| `?` | Đúng 1 ký tự | `?afe`, `vin?milk` | CAFE, KAFE; VINAMILK, VINEMILK |
| `từ~1`, `từ~2` | Gần đúng: khác tối đa 1 hoặc 2 ký tự (thêm, bớt, thay, đảo) | `kodak~1`, `vinamilk~2` | KODAC, CODAK; VINAMIK, VINAMILKS |
| `"a b"~N` | Các từ cách nhau tối đa N vị trí, cho phép đảo thứ tự | `"minh an"~2` | MINH PHÁT AN, AN MINH |
| `^N` | Tăng trọng số để đưa lên đầu | `"minh an"^5 OR minh~1` | trùng chính xác đứng đầu danh sách |

`AND`, `OR`, `NOT` phải viết HOA; `and`, `or` viết thường bị coi là một từ để tìm.

### Quy tắc dùng `~` (gần đúng)

- Tối đa 2 sửa đổi; `từ~` không ghi số nghĩa là `~2`.
- Từ 1–3 ký tự: **không** dùng `~`, vì `an~1` khớp gần như mọi nhãn có từ 2–3 chữ. Từ 4–5 ký tự: `~1`. Từ 6 ký tự trở lên: `~1`, chỉ dùng `~2` khi cần quét rộng.
- Mỗi từ có `~` chỉ được mở rộng thành tối đa 50 từ gần nhất có trong chỉ mục. Từ quá phổ biến có thể bị sót biến thể; khi đó bổ sung biến thể cụ thể bằng `OR`.
- `~` sau một từ là gần đúng; `~` sau cụm trong ngoặc kép là khoảng cách giữa các từ. Không đặt `~` hay `*` bên trong ngoặc kép, và không ghép `*` với `~` trong cùng một từ.

### Quy tắc dùng `*` và `?`

- Từ có `*`, `?` không được chuẩn hóa: viết chữ thường, không dấu.
- `*` ở đầu từ (`*mart`) được phép nhưng chậm; chỉ dùng khi cần bắt hậu tố.
- Cần ít nhất 3 ký tự cố định: `vi*` hay `a*` quá rộng, vô dụng.
- `?` chỉ thay đúng 1 ký tự. Âm có nhiều cách viết khác độ dài (`ph`/`f`, `gi`/`d`, `oo`/`u`) thì liệt kê bằng `OR`: `phone OR fone`.

### Ký tự đặc biệt và lỗi cú pháp

- Các ký tự `+ - && || ! ( ) { } [ ] ^ " ~ * ? : \ /` là toán tử. Khi chúng là một phần của tên nhãn (A&B, C/O, K+, 3-IN-1, "BLACK AND WHITE"), thay bằng khoảng trắng hoặc đặt cả tên trong ngoặc kép: `"c o"`, `"black and white"`. Riêng `/` (mở biểu thức chính quy) và `:` (chỉ định trường) gần như chắc chắn gây lỗi nếu để trần.
- Không dùng tiền tố trường kiểu `nhanhieu:` hay `spdv:`.
- Câu truy vấn sai cú pháp bị từ chối với lỗi tham số, nêu rõ tham số nào sai và cách sửa, không trừ lượt. Sửa theo thông điệp và gọi lại đúng 1 lần; vẫn lỗi thì báo người dùng.
- Người dùng tự đưa câu truy vấn Lucene: dùng nguyên văn, chỉ sửa khi có lỗi cú pháp rõ ràng, và nói rõ đã sửa gì.

### Tham số khác khi tìm tương tự

- `nice_class`: nhiều nhóm cách nhau bằng dấu phẩy là **HOẶC** (nhãn thuộc ít nhất một nhóm).
- `include_related_classes`: `true` để server mở rộng `nice_class` sang nhóm liên quan; xem mục *Nhóm liên quan*.
- **Không lọc `status`**: cần thấy cả đơn đang chờ và nhãn đã chết để đánh giá rủi ro.
- `owner` cùng cú pháp Lucene; muốn tìm nhãn tương tự của riêng một đối thủ thì thêm `owner='"tên riêng"'`.
- `include_goods_services=true`: mỗi nhãn trả kèm danh mục sản phẩm/dịch vụ theo từng nhóm (giống tùy chọn hiển thị chi tiết SPDV trên website), không tốn thêm lượt. **Luôn bật khi tìm tương tự**, và hạ `page_size` xuống khoảng 15 vì kết quả dài hơn.
- `goods_services`: từ khóa trong danh mục sản phẩm/dịch vụ đã đăng ký, cùng cú pháp Lucene như `query`; chỉ mục có cả bản có dấu, bỏ dấu và bản dịch. Dùng để tìm nhãn có hàng hóa trùng hoặc tương tự **bất kể nhóm Nice**. Ví dụ: `goods_services='"sua chua" OR yogurt OR yaourt* OR "sua len men"'`.
- `goods_services` và `nice_class` kết hợp theo **AND**, nhưng cả hai xét trên **toàn nhãn**, không theo từng nhóm: danh mục mọi nhóm của một nhãn được gộp vào một trường. `nice_class='43'` kèm `goods_services='cafe'` khớp nhãn có nhóm 43 **và** có chữ "cafe" ở bất kỳ nhóm nào (ví dụ nhóm 43 chỉ có khách sạn, còn "cà phê" nằm ở nhóm 30). Vì vậy luôn đối chiếu mục hàng hóa khớp thuộc nhóm nào trong `include_goods_services`.

## Quy trình

1. Xác định nhóm Nice của sản phẩm/dịch vụ Y. Nếu chưa rõ, gọi `get_reference_categories(category_type='NiceClass')` (miễn phí). Chưa biết Y và ngữ cảnh không cho biết: hỏi người dùng, hoặc tra trên mọi nhóm và nói rõ điều đó.
   **Trước lượt `search_trademarks` đầu tiên, hỏi phạm vi nhóm** theo mục *Hỏi người dùng về phạm vi nhóm*.
2. Phân tích tên X trước khi gọi tool:
   - Chuyển chữ thường, bỏ dấu.
   - Tách **phần đặc trưng** khỏi từ mô tả, chỉ dẫn chung ("coffee", "food", "group", "pharma", "việt nam", "shop", "tea"…). Xung đột thường nằm ở phần đặc trưng.
   - Ghi ra các biến thể **dấu hiệu**, chia ba mức:
     - **D1 – trùng, gần trùng:** cụm đầy đủ, viết liền và tách (`minhan` / `minh an`), các từ đặc trưng bắt buộc cùng có mặt, và các biến thể âm dễ nhầm cụ thể (c/k/q, ph/f, i/y, s/x, d/gi/z, v/w, ch/tr, nh/ng, oo/u, ee/i), vì `kodak` và `codak` gần như là trùng.
     - **D2 – tương tự chính tả:** `~`, `*`, `?` trên phần đặc trưng, theo quy tắc độ dài từ ở trên.
     - **D3 – tương tự ý nghĩa:** nếu X có nghĩa rõ, bản dịch Anh–Việt ("mat troi" ↔ "sun"), vì tương tự về ý nghĩa cũng là căn cứ từ chối.
   - Ghi ra **từ khóa hàng hóa/dịch vụ** của Y, chia hai mức (chữ thường không dấu, dùng `*` cho dạng biến đổi):
     - **Hẹp:** tên gọi chính của Y và từ đồng nghĩa trực tiếp. Ví dụ quán cà phê: `"quan ca phe" OR cafe OR coffee`; sữa chua: `"sua chua" OR yogurt OR yaourt*`.
     - **Rộng:** thêm hàng hóa cùng bản chất, cùng kênh phân phối hoặc bổ trợ. Ví dụ quán cà phê: `"ca phe" OR cafe OR coffee OR "do uong" OR tra OR banh*`.
3. Tìm theo thang **từ hẹp đến rộng trên cả hai trục** dấu hiệu và hàng hóa/nhóm. Mỗi bậc chỉ mở rộng một trục so với bậc trước, là **một** lượt `search_trademarks` với `include_goods_services=true`, `page_size=15`, gộp biến thể bằng `OR`:

   | Bậc | Dấu hiệu | Hàng hóa / nhóm | Tham số |
   | --- | --- | --- | --- |
   | 1 | D1 | Hàng hóa trùng Y trong nhóm chính | `query` D1, `nice_class=<nhóm chính>`, `include_related_classes=false`, `goods_services` hẹp |
   | 2 | D1 | Cả nhóm chính | như bậc 1, bỏ `goods_services` |
   | 3 | D2 | Cả nhóm chính | `query` D2, `nice_class=<nhóm chính>`, `include_related_classes=false` |
   | 4 | D1 + D2 | Nhóm liên quan theo lựa chọn (2) hoặc (3), **không gồm nhóm chính** | `query` D1 `OR` D2, `include_related_classes=false`, `nice_class` = các nhóm liên quan (thêm nhóm chiều ngược nếu chọn (3)) lấy từ `get_reference_categories`, bỏ nhóm chính vì đã tra ở bậc 2–3 |
   | 5 | D1 + D2 | Mọi nhóm, lọc theo hàng hóa tương tự | `query` như bậc 4, bỏ `nice_class`, `goods_services` rộng |
   | 6 | D3 | Nhóm chính và phạm vi người dùng đã chọn | `query` D3, tham số nhóm theo bảng ở mục *Hỏi người dùng về phạm vi nhóm*; chỉ chạy khi X có nghĩa rõ |

   Bậc 5 không mở rộng từ bậc 4 mà lọc theo một chiều khác: bắt nhãn tương tự đăng ký hàng hóa giống Y nhưng ở nhóm ngoài phạm vi đã chọn. Bậc này vẫn phải chạy kể cả khi bậc 4 không có kết quả.

   **Điều kiện dừng và bỏ qua bậc:**
   - **Dừng sớm** khi đã có nhãn `Cấp bằng` hoặc `Đang giải quyết` mà dấu hiệu trùng/tương tự **và** hàng hóa trùng/tương tự (rủi ro cao). Câu trả lời ghi rõ các bậc đã bỏ qua.
   - **`total = 0` hoặc ít kết quả không phải lý do dừng**: chạy tiếp bậc sau.
   - **Bỏ bậc 1** khi tên X hiếm (từ ghép tự đặt như "vinamilk", "kodak"): chạy thẳng bậc 2, vì trong nhóm chính đã ít kết quả. Tên phổ biến (từ thông dụng, tên người như "minh an") thì luôn chạy bậc 1, để nhãn có hàng hóa trùng không bị nhãn cùng tên ở hàng hóa khác đẩy khỏi trang 1.
   - **Bỏ bậc 4** khi người dùng chọn phạm vi (1) chỉ nhóm chính; câu trả lời nhắc một dòng rằng chưa xét nhóm liên quan.
   - **Không có nhóm chính** (chưa biết Y và người dùng muốn tra mọi nhóm): chạy bậc 1–3 không có `nice_class` (bậc 1 vẫn giữ `goods_services` hẹp nếu biết từ khóa hàng hóa), bỏ bậc 4, và nói rõ điều đó trong câu trả lời.
4. Đọc kết quả:
   - Chỉ xem trang 1 (đã xếp theo độ liên quan). `total` vài trăm trở lên là truy vấn quá rộng: siết lại ngay tại bậc đó (`~2` → `~1`, thêm `+`, bỏ wildcard ngắn) thay vì lật trang; ở bậc 4 có thể bỏ phần D2, chỉ giữ D1. Nói rõ đã siết thế nào.
   - `total = 0` ở bậc gần đúng là tín hiệu tốt, nhưng không phải bảo đảm. Nêu rõ những biến thể đã tra.
   - Ở bậc có `goods_services`, kiểm tra mục hàng hóa khớp nằm ở nhóm nào (xem lưu ý về `goods_services` ở trên).
5. **So sánh hàng hóa/dịch vụ** của từng nhãn gần với X, theo mục *So sánh hàng hóa, dịch vụ* bên dưới. Không kết luận trùng hàng hóa chỉ vì trùng nhóm Nice.
6. `get_trademark_detail` chỉ khi cần thông tin ngoài danh mục: tiến trình xử lý, gia hạn, yếu tố loại trừ (tối đa 3 nhãn). Link mẫu nhãn đã có sẵn trong kết quả tìm.
7. Đánh giá rủi ro theo thang dưới đây.

## So sánh hàng hóa, dịch vụ

Nhóm Nice chỉ là công cụ phân loại: cùng nhóm chưa chắc hàng hóa tương tự (nhóm 09 có cả phần mềm lẫn kính mắt), khác nhóm vẫn có thể tương tự (cà phê nhóm 30 và quán cà phê nhóm 43). Với mỗi nhãn gần với X, đọc danh mục sản phẩm/dịch vụ trả về và xếp quan hệ với Y:

- **Trùng:** cùng loại hàng hóa/dịch vụ (ví dụ "sữa chua" và "sữa chua uống").
- **Tương tự:** cùng hoặc gần bản chất; cùng chức năng, mục đích sử dụng; cùng kênh phân phối, cùng đối tượng mua; hoặc bổ trợ nhau đến mức người tiêu dùng nghĩ cùng một doanh nghiệp cung cấp (ví dụ cà phê hạt và dịch vụ quán cà phê; xe máy và dịch vụ sửa chữa xe máy).
- **Khác:** không có mối liên hệ nào ở trên, dù cùng nhóm.

Trích nguyên văn mục hàng hóa làm căn cứ (ví dụ "nhóm 29: sữa chua; sữa tươi"), không diễn giải chung chung kiểu "cùng nhóm 29". Danh mục dài thì chỉ trích các mục liên quan tới Y.

## Nhóm liên quan

Nhãn tương tự ở nhóm khác nhưng hàng hóa/dịch vụ có liên quan (cùng công dụng, cùng kênh phân phối, bổ trợ cho nhau) vẫn có thể là căn cứ từ chối. Phạm vi tra rộng hay hẹp là quyết định chuyên môn của người dùng, nên **hỏi trước, không tự quyết**.

### Hỏi người dùng về phạm vi nhóm

**Chuẩn bị:** gọi `get_reference_categories(category_type='NiceClass', class_number='<nhóm chính>')` (miễn phí, kết quả ngắn). Mỗi nhóm trả về kèm hai danh sách:
- **Nhóm liên quan:** đúng các nhóm server thêm khi `include_related_classes=true`.
- **Chiều ngược:** các nhóm có liệt kê nhóm chính là nhóm liên quan của chúng. Server **không** tự thêm các nhóm này, vì bảng của VNTM chỉ mở rộng một chiều (ví dụ nhóm 43 không tự kéo theo đồ uống nhóm 32).

Chỉ liệt kê cho người dùng các danh sách lấy từ tool, không tự suy ra.

**Hỏi một lần**, ngay trước lượt `search_trademarks` đầu tiên (gộp với câu hỏi khác nếu có). Lựa chọn này chỉ áp dụng cho bậc 4 và 6; bậc 1–3 luôn chạy trên nhóm chính. Lựa chọn (1) bỏ được bậc 4; (2) và (3) cùng số lượt, chỉ khác độ rộng và lượng kết quả cần xem.
- (1) Chỉ nhóm chính.
- (2) Nhóm chính và các nhóm liên quan theo bảng VNTM (liệt kê danh sách *Nhóm liên quan*).
- (3) Như (2) và thêm các nhóm chiều ngược chưa có trong (2). Chỉ đưa lựa chọn này khi phần thêm khác rỗng.

Ví dụ:

> Quán cà phê thuộc nhóm 43. Anh/chị muốn tra: (1) chỉ nhóm 43; (2) nhóm 43 và các nhóm liên quan theo bảng VNTM: 35–42, 44, 45; hay (3) như (2) và thêm các nhóm liên quan theo chiều ngược: 29, 30, 31, 32, 33 (đồ ăn, đồ uống)?

Nhóm chính là 35 (bán lẻ, quảng cáo) hoặc 42 (thiết kế, công nghệ): chiều ngược gồm gần như mọi nhóm hàng hóa, nên ở lựa chọn (3) chỉ thêm các nhóm hàng hóa người dùng nêu (ví dụ bán lẻ mỹ phẩm → 03).

Ánh xạ câu trả lời sang tham số ở bậc 6 (tham số bậc 4 nằm trong bảng thang ở mục *Quy trình*):

| Lựa chọn | Tham số |
| --- | --- |
| (1) Chỉ nhóm chính | `nice_class=<nhóm chính>`, `include_related_classes=false` |
| (2) Thêm nhóm liên quan | `nice_class=<nhóm chính>`, `include_related_classes=true` |
| (3) Thêm cả chiều ngược | `include_related_classes=false`, `nice_class` = nhóm chính + nhóm liên quan + nhóm chiều ngược đã chọn, liệt kê đủ |

Không hỏi khi:
- Người dùng đã nói rõ phạm vi ("chỉ nhóm 43", "tra cả nhóm liên quan", tự đưa danh sách nhóm), hoặc đã trả lời câu này trước đó trong cuộc trò chuyện: dùng lại lựa chọn đó.
- Người dùng yêu cầu chạy luôn, không hỏi: chọn (2) và nói rõ ở đầu câu trả lời.

Trong câu trả lời luôn nêu phạm vi nhóm đã tra. Nếu đã chọn (1), nhắc một dòng rằng chưa xét nhóm liên quan.

### Lưu ý khi mở rộng nhóm

- `include_related_classes=true` không tốn thêm lượt. Không vừa tự liệt kê nhiều nhóm vào `nice_class` vừa bật `true`: mỗi nhóm trong `nice_class` cũng bị mở rộng tiếp và kết quả sẽ quá rộng.
- Không tách thêm lượt theo từng nhóm liên quan.
- Mỗi nhãn tìm được đánh dấu là trùng nhóm chính hay chỉ trùng nhóm liên quan, để người dùng dễ đọc; còn mức rủi ro dựa trên so sánh hàng hóa cụ thể, không dựa trên nhóm.

## Thang rủi ro

Rủi ro phụ thuộc đồng thời vào mức tương tự của **dấu hiệu** và của **hàng hóa/dịch vụ** (theo mục trên), chỉ xét nhãn `Cấp bằng` chưa hết hạn hoặc `Đang giải quyết`:

- **Cao**: dấu hiệu trùng hoặc gần trùng (chữ, cách đọc) **và** hàng hóa trùng hoặc tương tự.
- **Trung bình**: dấu hiệu tương tự (cách đọc, ý nghĩa, phần đặc trưng) và hàng hóa trùng/tương tự; hoặc dấu hiệu trùng nhưng hàng hóa chỉ liên quan xa.
- **Thấp**: hàng hóa khác hẳn (kể cả khi cùng nhóm); hoặc chỉ còn nhãn `Từ chối` hay thuộc nhóm `Hết hiệu lực` (rút đơn, hết hạn, mất hiệu lực, từ bỏ).

Luôn nêu lý do cho mức đánh giá, kèm số đơn của nhãn gây rủi ro và mục hàng hóa trùng/tương tự trích từ danh mục.

## Trình bày

Câu đầu là mức rủi ro và lý do. Bảng kết quả gồm: tên nhãn, số đơn, chủ sở hữu, nhóm Nice (đánh dấu nhóm chính hay nhóm liên quan), trạng thái, mẫu nhãn, deep-link, và cột **hàng hóa trùng/tương tự** (mục trích từ danh mục và quan hệ: trùng/tương tự/khác). Mẫu nhãn dẫn dạng link `[xem](url)` lấy từ dòng "Mẫu nhãn" của kết quả, không nhúng ảnh; bạn không nhìn thấy ảnh nên không nhận xét phần hình. Phạm vi đã tra: nguyên văn từng câu truy vấn, lựa chọn phạm vi nhóm, số lượt đã dùng, các bậc bỏ qua và lý do. Thêm một dòng giới hạn: tra cứu chỉ dựa trên chữ (không bắt được logo/nhãn hình tương tự; người dùng tự xem mẫu nhãn qua link để đánh giá phần hình), dữ liệu có thể chậm hơn công báo mới nhất, thẩm định cuối cùng thuộc Cục Sở hữu trí tuệ, và đây không phải ý kiến pháp lý.

## Ví dụ

**Hỏi:** "Tôi muốn đăng ký tên MINH AN COFFEE cho quán cà phê, có trùng không?"

1. Quán cà phê là dịch vụ nhóm 43. "COFFEE" là từ mô tả; phần đặc trưng là "minh an".
2. `get_reference_categories(category_type='NiceClass', class_number='43')`, rồi hỏi phạm vi nhóm như ví dụ ở mục *Hỏi người dùng về phạm vi nhóm*. Giả sử người dùng chọn (3).
3. "minh an" là tên người phổ biến nên chạy cả bậc 1. Mọi lượt đều kèm `include_goods_services=true, page_size=15`.
   - D1: `"minh an"^5 OR minhan OR (+minh +an) OR "ming an"`
   - D2: `minhan~1 OR "minh an"~2 OR (+minh~1 +an)`. Không dùng `an~1` vì "an" chỉ có 2 ký tự.
4. Bậc 1: `search_trademarks(query=<D1>, nice_class='43', goods_services='"quan ca phe" OR cafe OR coffee')`. Không có nhãn nào ở nhóm 43 có hàng hóa trùng.
5. Bậc 2 (`query=<D1>`) và bậc 3 (`query=<D2>`), cùng `nice_class='43'`. Có nhãn "MINH AN" nhóm 43 chỉ đăng ký "dịch vụ khách sạn": cùng nhóm nhưng khác loại dịch vụ, rủi ro thấp hơn.
6. Bậc 4: `query='<D1> OR <D2>'`, `nice_class` = các nhóm của lựa chọn (3) trừ 43 (đã tra ở bậc 2–3), `include_related_classes=false`. Có nhãn "MINH AN" nhóm 30 đăng ký "cà phê; cà phê hạt": **tương tự** (bổ trợ) với dịch vụ quán cà phê, tức rủi ro cao, nên dừng.
7. Bỏ bậc 5–6: đã có rủi ro cao; "minh an" là tên riêng, không cần tra theo nghĩa.

Trả lời: "Rủi ro **cao**: nhãn 'MINH AN' (đơn 4-20xx-xxxxx, Cấp bằng) trùng phần đặc trưng của tên bạn định dùng, đăng ký cho 'cà phê; cà phê hạt' (nhóm 30), là hàng hóa bổ trợ với dịch vụ quán cà phê." Sau đó là bảng có cột hàng hóa trùng/tương tự, phạm vi đã tra (lựa chọn nhóm (3), 4 câu truy vấn nguyên văn, 4 search, bậc 5–6 bỏ qua và lý do) và dòng giới hạn.

**Hỏi:** "Tìm các nhãn tương tự VINAMILK ở nhóm 29, tra cả nhóm liên quan"

1. Tên 8 ký tự, viết liền, tự đặt nên hiếm: bỏ bậc 1. Người khác hay tách "VINA MILK", đổi v/w hoặc chèn chữ. Người dùng đã nói rõ phạm vi (2) nên không hỏi lại.
   - D1: `vinamilk^5 OR "vina milk" OR winamilk`
   - D2: `vinamilk~2 OR vina*milk OR vin?milk OR (+vina* +milk)`
2. Bậc 2: `search_trademarks(query=<D1>, nice_class='29', include_goods_services=true, page_size=15)`
3. Bậc 3: như bậc 2 với `query=<D2>`.
4. Bậc 4: `query='<D1> OR <D2>'`, `nice_class='05, 30, 31, 32, 33, 35, 42, 43'` (các nhóm liên quan của 29 lấy từ `get_reference_categories`, không gồm 29), `include_related_classes=false`. Nếu `total` quá lớn, siết lại: bỏ `(+vina* +milk)`, đổi `~2` thành `~1`.
5. Bậc 5: bỏ `nice_class`, `query` như bậc 4, `goods_services='sua OR milk OR "sua chua" OR yogurt OR "do uong"'`.
6. Bỏ bậc 6: "vina milk" chỉ mang nghĩa mô tả.
7. Với từng nhãn, so danh mục với "sữa và sản phẩm từ sữa": nhãn ở nhóm 32 đăng ký "nước khoáng" là **khác**, dù nhóm 32 là nhóm liên quan.
