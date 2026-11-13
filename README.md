

### BÀI LÀM: SO SÁNH TRADE-OFF CÁC GIẢI PHÁP PROMPTING

**Bài toán:** Trích xuất tin nhắn đặt hàng thành JSON và Table, có chặn lỗi khách nhắn thiếu thông tin.

---

### Bước 1: 2 mẫu Prompt đề xuất

**Cách 1: Ngắn gọn, trực tiếp**


Trích xuất tin nhắn sau thành JSON (trang_thai, ten, sdt, dia_chi, san_pham) và 1 bảng Table tóm tắt. Thiếu thông tin thì ghi "TRONG".
Tin nhắn: "shop ơi lấy mình 2 áo hoodie đen size XL với 1 quần jogger xám, sđt 091234567, giao ở Thủ Đức"



**Cách 2: Chi tiết (CREATE + Few-shot + Bẫy lỗi)**


Vai trò: Nhân viên nhập đơn shop quần áo.
Ví dụ mẫu: Input: "1 váy hoa size M về 10 Trần Hưng Đạo Q1, tên Lan, đt 0909112233" -> Output JSON: {"trang_thai": "OK", "ten": "Lan", "sdt": "0909112233", "dia_chi": "10 Trần Hưng Đạo Q1", "san_pham": [{"ten_sp": "Váy hoa", "size": "M", "sl": 1}], "loi": []}
Yêu cầu: Xử lý tin nhắn: "shop ơi lấy mình 2 áo hoodie đen size XL với 1 quần jogger xám, sđt 091234567, giao ở Thủ Đức"
Ràng buộc: 
1. Thiếu thông tin (tên, sđt, địa chỉ, size) thì ghi "TRONG", không tự đoán.
2. SĐT không đủ 10 số thì ghi vào "loi" và đổi trang_thai thành "CAN_CHECK".
3. Chỉ xuất 1 cục JSON và 1 bảng Table bên dưới.



---

### Bước 2: Bảng so sánh Trade-off

| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) |
| --- | --- | --- |
| **Độ dễ hiểu / Viết** | Cao (Gõ nhanh, dễ đọc) | Trung bình (Mất công thiết kế khung và ví dụ) |
| **Tốn token** | Ít (Ngắn gọn, tiết kiệm chi phí) | Nhiều hơn (Tốn thêm token cho phần ví dụ và điều kiện) |
| **Chặn lỗi biên** | Trung bình (Dễ bỏ sót khi khách nhắn thiếu SĐT hoặc gộp món) | Cực tốt (Bắt chuẩn xác lỗi thiếu SĐT, thiếu size) |
| **Độ ổn định format** | Thấp (Dễ bị dính chữ thừa ngoài JSON) | Cao (Ép đúng khuôn mẫu nhờ Few-shot) |

---

### Bước 3: Lựa chọn & Chạy thử

**Chọn Giải pháp 2.** Vì dữ liệu này dùng để đẩy vào phần mềm quản lý, nếu AI trả sai format JSON hoặc tự bịa size áo thì hệ thống sẽ lỗi. Tốn thêm chút token nhưng đổi lại dữ liệu sạch và an toàn tuyệt đối.

**Kết quả chạy thử Giải pháp 2:**

json

{

  "trang_thai": "CAN_CHECK",
  
  "ten": "TRONG",
  
  "sdt": "091234567",
  
  "dia_chi": "Thủ Đức (chưa rõ địa chỉ)",
  
  "san_pham": [
  
  
  { "ten_sp": "Áo hoodie đen", "size": "XL", "sl": 2 },
  
  { "ten_sp": "Quần jogger xám", "size": "TRONG", "sl": 1 }
  
  ],
  
  "loi": ["Thiếu tên khách", "SĐT thiếu 1 số", "Quần jogger chưa có size"]
}



| Thông tin | Kết quả trích xuất | Ghi chú |
| --- | --- | --- |
| **Tên khách** | `TRONG` | Hỏi lại tên |
| **SĐT** | 091234567 | **Lỗi: Thiếu 1 số** |
| **Địa chỉ** | Thủ Đức | Xin địa chỉ cụ thể |
| **Sản phẩm** | 2 Áo hoodie đen (XL), 1 Quần jogger xám (`TRONG`) | **Lỗi: Thiếu size quần** |
