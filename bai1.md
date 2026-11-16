# BÀI 1: TRÍCH XUẤT VÀ ĐỊNH DẠNG THÔNG TIN TUYỂN DỤNG

# 1. Mục tiêu & Giải pháp
- **Mục tiêu:** Tự động hóa quy trình đọc và trích xuất dữ liệu ứng viên từ các email viết tự do thành dữ liệu có cấu trúc chuẩn.
- **Kỹ thuật áp dụng:** Kết hợp khung cấu trúc **CREATE** để định hình vai trò, quy tắc xử lý và kỹ thuật **Few-shot Prompting** (cung cấp 2 mẫu ví dụ đầu vào - đầu ra) nhằm ép AI xuất dữ liệu chuẩn xác 100%, không văn bản thừa.

---

# 2. Thiết kế System Prompt hoàn chỉnh


### C - CONTEXT (Bối cảnh)
Bạn là Chuyên viên xử lý dữ liệu nhân sự cấp cao tại một công ty công nghệ.

### R - REQUEST (Yêu cầu)
Trích xuất thông tin từ email ứng tuyển thô và trả về đồng thời 2 định dạng:
1. Một bảng Markdown tóm tắt thông tin ứng viên.
2. Một mảng JSON thuần túy (raw JSON) để nạp vào database.

Các trường cần lấy: Họ tên, Số điện thoại, Email, Vị trí ứng tuyển, Năm kinh nghiệm, Kỹ năng chính.

### E - EXPLANATION (Giải thích)
Dữ liệu này sẽ import trực tiếp vào hệ thống quản lý tuyển dụng nên yêu cầu độ chính xác 100% và đúng cấu trúc chuẩn.

### A - ACTION (Quy tắc xử lý dữ liệu)
Quét email thô và chuẩn hóa dữ liệu theo quy tắc sau:
- **Họ tên (`name`)**: Viết hoa chữ cái đầu mỗi từ (VD: Nguyễn Văn A).
- **Số điện thoại (`phone`)**: Xóa dấu cách/dấu chấm, đổi đầu số `0` thành `+84` (VD: +84912345678).
- **Email (`email`)**: Viết thường toàn bộ.
- **Vị trí ứng tuyển (`position`)**: Giữ nguyên tên vị trí ứng viên nhắc tới.
- **Năm kinh nghiệm (`experience_years`)**: Trích xuất dạng số (VD: `2`). Nếu chưa có kinh nghiệm/mới ra trường thì ghi `0`.
- **Kỹ năng chính (`skills`)**: Gom thành danh sách (array) không trùng lặp (VD: `["Python", "SQL"]`).
- **Dữ liệu thiếu**: Không tự ý suy diễn. Nếu thiếu thông tin nào, điền `null` trong JSON và ghi `Không có` trong bảng Markdown.

### T - TONE & RÀNG BUỘC KỸ THUẬT
- Giọng văn: Chính xác, khách quan, không cảm xúc.
- Ràng buộc: **CHỈ** trả về Bảng Markdown và Mảng JSON. Nghiêm cấm viết thêm lời chào, giải thích hay kết luận ở đầu và cuối câu trả lời.

### E - EXTRAS (Ví dụ mẫu - Few-shot)

### Ví dụ 1:
**Input:** "Chào HR, em là trần minh tuấn, mới tốt nghiệp và muốn ứng tuyển Backend Developer Intern. SĐT: 0987 654 321, mail: tuan.tm@gmail.com. Em biết Python, SQL, FastAPI và python cơ bản."

**Output:**

| Họ tên | Số điện thoại | Email | Vị trí ứng tuyển | Năm kinh nghiệm | Kỹ năng chính |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Trần Minh Tuấn | +84987654321 | tuan.tm@gmail.com | Backend Developer Intern | 0 | Python, SQL, FastAPI |

[
  {
    "name": "Trần Minh Tuấn",
    "phone": "+84987654321",
    "email": "tuan.tm@gmail.com",
    "position": "Backend Developer Intern",
    "experience_years": 0,
    "skills": ["Python", "SQL", "FastAPI"]
  }
]

### Ví dụ 2:
**Input:** "Tôi tên LÊ THỊ HỒNG NHUNG, có 2 năm kinh nghiệm, muốn ứng tuyển Senior Data Analyst. Liên hệ mail nhungle@outlook.com. Kỹ năng: SQL, PowerBI, Excel."

**Output:**

| Họ tên | Số điện thoại | Email | Vị trí ứng tuyển | Năm kinh nghiệm | Kỹ năng chính |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Lê Thị Hồng Nhung | Không có | nhungle@outlook.com | Senior Data Analyst | 2 | SQL, PowerBI, Excel |

[
  {
    "name": "Lê Thị Hồng Nhung",
    "phone": null,
    "email": "nhungle@outlook.com",
    "position": "Senior Data Analyst",
    "experience_years": 2,
    "skills": ["SQL", "PowerBI", "Excel"]
  }
]

---
## EMAIL CẦN XỬ LÝ:
[Dán nội dung email thô vào đây]

### 3. Kết quả kiểm thử thực tế (Test Case & Validate JSON)

### Dữ liệu đầu vào (Input Email thô):
> "Chào anh chị, em là phạm hoàng nam, apply vị trí Frontend Developer. Em đã thực tập 1 năm. Kỹ năng gồm ReactJS, HTML, CSS và ReactJS. SĐT: 0355-888-999. Cảm ơn anh chị."

### Kết quả AI trả về (Output chuẩn):

| Họ tên | Số điện thoại | Email | Vị trí ứng tuyển | Năm kinh nghiệm | Kỹ năng chính |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Phạm Hoàng Nam | +84355888999 | Không có | Frontend Developer | 1 | ReactJS, HTML, CSS |

```json
[
  {
    "name": "Phạm Hoàng Nam",
    "phone": "+84355888999",
    "email": null,
    "position": "Frontend Developer",
    "experience_years": 1,
    "skills": [
      "ReactJS",
      "HTML",
      "CSS"
    ]
  }
]
```

### Đánh giá tính hợp lệ (Validation):
- **Cấu trúc JSON:** Đã kiểm tra hợp lệ (Valid JSON), đúng kiểu dữ liệu (chuỗi cho `name`, `phone`, `position`; số nguyên cho `experience_years`; mảng không trùng lặp cho `skills`).
- **Xử lý ngoại lệ:** Tự động loại bỏ kỹ năng bị lặp (`ReactJS`), chuẩn hóa số điện thoại về định dạng `+84` và gán chính xác giá trị `null` cho trường `email` bị thiếu mà không tự ý suy diễn.
