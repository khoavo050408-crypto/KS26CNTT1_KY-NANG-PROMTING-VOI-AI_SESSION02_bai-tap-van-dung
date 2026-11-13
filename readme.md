# BƯỚC 1: THIẾT KẾ GIẢI PHÁP (INPUT / OUTPUT)

## 1. Mục tiêu bài toán
Tự động hóa quy trình tiếp nhận và phân loại yêu cầu đặt hàng hoặc khiếu nại từ khách hàng gửi đến cửa hàng. Hệ thống cần chuẩn hóa dữ liệu đầu vào (thường viết tắt, thiếu thông tin hoặc sai định dạng), áp dụng kỹ thuật Few-shot để định hướng cách phân tích, tích hợp cơ chế bắt lỗi dữ liệu thiếu/sai lệch, và xuất kết quả bắt buộc dưới dạng cấu trúc JSON hợp lệ kèm bảng tổng hợp (Markdown Table).

## 2. Định nghĩa Input (Đầu vào)
Dữ liệu đầu vào là tin nhắn/yêu cầu thô từ khách hàng qua các kênh chat trực tuyến (Facebook, Zalo, Website). Các tin nhắn này thường không theo cấu trúc chuẩn, có thể thiếu số điện thoại, thiếu địa chỉ hoặc kèm theo các điều kiện thay thế.

* **Ví dụ Input thực tế cần xử lý:** 
  > "Alo shop ơi, giao gấp cho mình 2 ly trà sữa trân châu đường đen size L đến số 12 Nguyễn Trãi nhé, sđt 0901234567, nhớ bỏ ít đá nha. À mà nếu hết size L thì lấy size M cũng được."

## 3. Định nghĩa Output (Đầu ra mong muốn)
Kết quả trả về từ AI phải tuân thủ nghiêm ngặt hai thành phần theo đúng thứ tự:
* **Phần 1 – Bảng tóm tắt nhanh (Markdown Table):** Giúp nhân viên bán hàng và bộ phận kho kiểm tra nhanh thông tin đơn hàng.
* **Phần 2 – Khối dữ liệu JSON chuẩn (Valid JSON):** Phục vụ tích hợp trực tiếp vào hệ thống phần mềm quản lý (POS/CRM), có trường cảnh báo lỗi (`error_warning`) để xử lý các bẫy dữ liệu.

---

# BƯỚC 2: VIẾT PROMPT HOÀN CHỈNH (CHUẨN HÓA THEO KHUNG CREATE)

Prompt dưới đây được thiết kế chuẩn xác theo 6 thành phần của khung **CREATE** (*Context, Request, Explanation, Action, Tone, Extras*), kết hợp với các ví dụ mẫu (**Few-shot**) bao gồm cả trường hợp bẫy dữ liệu thiếu/sai lệch:

````markdown
# C - Context (Bối cảnh & Vai trò)
Bạn là Trưởng bộ phận Vận hành & Xử lý đơn hàng tự động cho chuỗi cửa hàng bán lẻ. Nhiệm vụ hằng ngày của bạn là tiếp nhận các tin nhắn đặt hàng thô từ khách hàng trên mạng xã hội (thường viết tắt, thiếu thông tin hoặc kèm điều kiện phát sinh) và chuẩn hóa dữ liệu đưa vào hệ thống quản lý.

# R - Request (Yêu cầu cốt lõi)
Hãy phân tích tin nhắn đầu vào của khách hàng, trích xuất đầy đủ các trường thông tin đơn hàng và xuất ra bảng tóm tắt kèm dữ liệu JSON chuẩn xác.

# E - Explanation (Giải thích & Ràng buộc)
- Tuyệt đối không tự ý suy diễn hay bịa đặt thông tin (như số điện thoại, địa chỉ) nếu khách hàng không cung cấp.
- Nếu thiếu Số điện thoại hoặc Địa chỉ giao hàng, bắt buộc điền giá trị "THIẾU_THÔNG_TIN" và ghi chú rõ vào trường error_warning.
- Nếu số điện thoại không đủ 10 chữ số hoặc có dấu hiệu nhập thiếu, phải gắn cờ cảnh báo lỗi.
- Khối JSON đầu ra phải hợp lệ 100% (Valid JSON) để hệ thống backend có thể đọc trực tiếp mà không gặp lỗi cú pháp.

# A - Action (Các bước hành động)
Thực hiện tuần tự theo 3 bước sau:
1. Đọc và phân loại mục đích tin nhắn (Đặt hàng / Khiếu nại / Tư vấn).
2. Trích xuất và chuẩn hóa các thực thể: Tên khách, SĐT, Địa chỉ, Sản phẩm (Tên, Số lượng, Kích cỡ, Ghi chú riêng) và Điều kiện thay thế nếu hết hàng.
3. Đối chiếu với quy tắc ràng buộc để phát hiện dữ liệu thiếu/sai lệch, sau đó xuất kết quả theo đúng định dạng yêu cầu.

# T - Tone (Giọng điệu văn phong)
Ngắn gọn, khách quan, chính xác theo chuẩn dữ liệu kỹ thuật và nghiệp vụ vận hành.

# E - Extras (Định dạng đầu ra & Bổ sung)
Kết quả trả về bắt buộc gồm 2 phần theo đúng thứ tự dưới đây, không thêm lời chào hay giải thích thừa:

Phần 1: Bảng tóm tắt nhanh (Markdown Table)
| Trường thông tin | Giá trị trích xuất |
| :--- | :--- |
| Loại yêu cầu | ... |
| Khách hàng | ... |
| Số điện thoại | ... |
| Địa chỉ | ... |
| Chi tiết sản phẩm | ... |
| Xử lý khi hết hàng | ... |
| Cảnh báo lỗi | ... |

Phần 2: Khối dữ liệu JSON chuẩn
```json
{
  "request_type": "...",
  "customer_name": "...",
  "phone": "...",
  "address": "...",
  "items": [
    {
      "product_name": "...",
      "quantity": 0,
      "size": "...",
      "note": "..."
    }
  ],
  "fallback_action": "...",
  "error_warning": null
}
```

---
# Few-shot Examples (Ví dụ mẫu huấn luyện)

## Ví dụ 1 (Đơn hàng đầy đủ thông tin):
- Input: "Cho anh Tuấn sđt 0987654321 lấy 1 cái áo phông đen size L giao về 45 Lê Duẩn nhé."
- Output:
| Trường thông tin | Giá trị trích xuất |
| :--- | :--- |
| Loại yêu cầu | DAT_HANG |
| Khách hàng | Anh Tuấn |
| Số điện thoại | 0987654321 |
| Địa chỉ | 45 Lê Duẩn |
| Chi tiết sản phẩm | 1x Áo phông đen (Size: L, Ghi chú: Không) |
| Xử lý khi hết hàng | Không có |
| Cảnh báo lỗi | Không |

```json
{
  "request_type": "DAT_HANG",
  "customer_name": "Anh Tuấn",
  "phone": "0987654321",
  "address": "45 Lê Duẩn",
  "items": [
    {
      "product_name": "Áo phông đen",
      "quantity": 1,
      "size": "L",
      "note": ""
    }
  ],
  "fallback_action": "Không có",
  "error_warning": null
}
```

## Ví dụ 2 (Bẫy dữ liệu: Thiếu địa chỉ & SĐT lỗi):
- Input: "Shop ơi ship cho mình 2 phần cơm gà giòn, nhớ kèm nước ngọt nha, sđt liên hệ 0912... à mà hết cơm gà thì lấy cơm bò."
- Output:
| Trường thông tin | Giá trị trích xuất |
| :--- | :--- |
| Loại yêu cầu | DAT_HANG |
| Khách hàng | Khách lẻ |
| Số điện thoại | 0912... |
| Địa chỉ | THIẾU_THÔNG_TIN |
| Chi tiết sản phẩm | 2x Cơm gà giòn (Size: Tiêu chuẩn, Ghi chú: Kèm nước ngọt) |
| Xử lý khi hết hàng | Thay thế bằng cơm bò nếu hết cơm gà |
| Cảnh báo lỗi | Thiếu địa chỉ giao hàng; Số điện thoại chưa đầy đủ |

```json
{
  "request_type": "DAT_HANG",
  "customer_name": "Khách lẻ",
  "phone": "0912...",
  "address": "THIẾU_THÔNG_TIN",
  "items": [
    {
      "product_name": "Cơm gà giòn",
      "quantity": 2,
      "size": "Tiêu chuẩn",
      "note": "Kèm nước ngọt"
    }
  ],
  "fallback_action": "Thay thế bằng cơm bò nếu hết cơm gà",
  "error_warning": "Thiếu địa chỉ giao hàng; Số điện thoại chưa đầy đủ"
}
```

---
# Input Data (Dữ liệu thực tế cần xử lý)
"Alo shop ơi, giao gấp cho mình 2 ly trà sữa trân châu đường đen size L đến số 12 Nguyễn Trãi nhé, sđt 0901234567, nhớ bỏ ít đá nha. À mà nếu hết size L thì lấy size M cũng được."
````

---

# BƯỚC 3: KẾT QUẢ PHẢN HỒI THỰC TẾ TỪ AI (CHẠY THỬ)

Sau khi chạy thử nghiệm Prompt trên với dữ liệu đầu vào thực tế, hệ thống AI trả về kết quả chính xác theo đúng định dạng yêu cầu:

## Trường hợp 1: Dữ liệu đầy đủ thông tin
**Input:** `"Alo shop ơi, giao gấp cho mình 2 ly trà sữa trân châu đường đen size L đến số 12 Nguyễn Trãi nhé, sđt 0901234567, nhớ bỏ ít đá nha. À mà nếu hết size L thì lấy size M cũng được."`

**Bảng tóm tắt nhanh:**

| Trường thông tin | Giá trị trích xuất |
| :--- | :--- |
| Loại yêu cầu | DAT_HANG |
| Khách hàng | Khách lẻ |
| Số điện thoại | 0901234567 |
| Địa chỉ | 12 Nguyễn Trãi |
| Chi tiết sản phẩm | 2x Trà sữa trân châu đường đen (Size: L, Ghi chú: Ít đá, giao gấp) |
| Xử lý khi hết hàng | Thay thế bằng size M nếu hết size L |
| Cảnh báo lỗi | Không |

**Khối dữ liệu JSON chuẩn:**
```json
{
  "request_type": "DAT_HANG",
  "customer_name": "Khách lẻ",
  "phone": "0901234567",
  "address": "12 Nguyễn Trãi",
  "items": [
    {
      "product_name": "Trà sữa trân châu đường đen",
      "quantity": 2,
      "size": "L",
      "note": "Ít đá; giao gấp"
    }
  ],
  "fallback_action": "Thay thế bằng size M nếu hết size L",
  "error_warning": null
}
```

## Trường hợp 2: Bẫy dữ liệu sai lệch (Số điện thoại thiếu số)
**Input:** `"Alo shop ơi, giao gấp cho mình 2 ly trà sữa trân châu đường đen size L đến số 12 Nguyễn Trãi nhé, sđt 09000, nhớ bỏ ít đá nha. À mà nếu hết size L thì lấy size M cũng được."`

**Bảng tóm tắt nhanh:**

| Trường thông tin | Giá trị trích xuất |
| :--- | :--- |
| Loại yêu cầu | DAT_HANG |
| Khách hàng | Khách lẻ |
| Số điện thoại | 090000 |
| Địa chỉ | 12 Nguyễn Trãi |
| Chi tiết sản phẩm | 2x Trà sữa trân châu đường đen (Size: L, Ghi chú: Ít đá, giao gấp) |
| Xử lý khi hết hàng | Thay thế bằng size M nếu hết size L |
| Cảnh báo lỗi | Số điện thoại không đủ 10 chữ số (chỉ có 6 chữ số), có dấu hiệu nhập thiếu |

**Khối dữ liệu JSON chuẩn:**
```json
{
  "request_type": "DAT_HANG",
  "customer_name": "Khách lẻ",
  "phone": "090000",
  "address": "12 Nguyễn Trãi",
  "items": [
    {
      "product_name": "Trà sữa trân châu đường đen",
      "quantity": 2,
      "size": "L",
      "note": "Ít đá; giao gấp"
    }
  ],
  "fallback_action": "Thay thế bằng size M nếu hết size L",
  "error_warning": "Số điện thoại không đủ 10 chữ số (chỉ có 6 chữ số), có dấu hiệu nhập thiếu"
}
```