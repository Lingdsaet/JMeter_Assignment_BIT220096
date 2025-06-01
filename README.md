# Báo cáo kiểm thử hiệu năng bằng JMeter

## Thông tin sinh viên

- Họ tên: Phan Hoài Linh
- MSSV: BIT220096
- Website được kiểm thử: https://github.com

## Mô tả kịch bản kiểm thử

1. **Kịch bản 1: Cơ bản**
   - Số lượng người dùng: 10 + (96 % 10) = 16
   - Hành vi: Gửi yêu cầu GET đến `https://github.com/?student_id=BIT220096`
   - Loop Count: 5

2. **Kịch bản 2: Tải nặng**
   - Số lượng người dùng: 50 + (96 % 20) = 66
   - Ramp-up Period: 30 giây
   - Hành vi: Gửi yêu cầu GET đến `https://github.com/` và `https://github.com/features`

3. **Kịch bản 3: Tùy chỉnh**
   - Số lượng người dùng: 20 + (96 % 15) = 26
   - Thời gian chạy: 60 giây
   - Hành vi: Gửi yêu cầu GET đến `https://github.com/features` và `https://github.com/contact`

## Kết quả kiểm thử

- **Kịch bản 1: Cơ bản**
  - Response Time trung bình: 2812 ms  
  - Throughput: 2.71 yêu cầu/giây  
  - Error Rate: 0.00%

- **Kịch bản 2: Tải nặng**
  - Response Time trung bình: 634 ms  
  - Throughput: 4.22 yêu cầu/giây  
  - Error Rate: 0.00%

- **Kịch bản 3: Tùy chỉnh**
  - Response Time trung bình: 807 ms  
  - Throughput: 31.90 yêu cầu/giây  
  - Error Rate: 65.99%

## Nhận xét

- **Kịch bản 1 (Cơ bản)**  
  - Thời gian phản hồi trung bình là 2812 ms, tương đối cao so với trang chủ GitHub. Nguyên nhân có thể do:  
    1. Mỗi Thread (16 threads) chạy 5 vòng liên tiếp, gây áp lực đồng thời lên server.  
    2. GitHub có cơ chế giới hạn tốc độ (rate limiting) cho các truy vấn không xác thực.  
  - Throughput ~2.71 req/s cho thấy số request gửi đi không quá lớn, nhưng Response Time cao dẫn đến hiển thị chậm trên JMeter.  
  - Error Rate 0% nghĩa tất cả request đều thành công (HTTP 200).

- **Kịch bản 2 (Tải nặng)**  
  - Response Time trung bình giảm xuống còn 634 ms so với kịch bản 1. Có thể lý giải:  
    1. Chỉ gửi mỗi 1 vòng lặp (Loop Count = 1), nên không tạo áp lực lâu dài.  
    2. Việc truy cập kèm trang con `/features` giúp JMeter lưu lại kết nối (keep-alive) hiệu quả hơn.  
  - Throughput ~4.22 req/s – tăng nhẹ, phù hợp với việc chỉ chạy 1 vòng.  
  - Error Rate 0%: Server vẫn đáp ứng bình thường dưới tải 66 threads ramp-up 30s.

- **Kịch bản 3 (Tùy chỉnh)**  
  - Response Time trung bình khoảng 807 ms; cao hơn kịch bản 2 nhưng thấp hơn kịch bản 1. Điều này do:  
    1. Kịch bản chạy liên tục trong 60 giây, tạo nhiều request hơn.  
    2. Mặc dù số Thread (26 threads) thấp hơn kịch bản 2, nhưng độ lặp vô hạn trong 60 giây tạo ra lưu lượng lớn.  
  - Throughput ~31.90 req/s – cao nhất trong ba kịch bản, vì JMeter đẩy request nhanh liên tục.  
  - Error Rate ~65.99%: Tỷ lệ lỗi rất cao, chứng tỏ nhiều request bị server trả về lỗi (xác suất 404 hoặc 429). Nguyên nhân có thể:  
    1. URL `/contact` hoặc `/features` không tồn tại dưới dạng GET mà cần cấu hình khác (GitHub có routing khác, không trả về 200).  
    2. GitHub giới hạn request tần suất cao, dẫn đến nhiều response 429 “Too Many Requests” hoặc 404 “Not Found”.  

---

**Tóm lại:**  
- GitHub xử lý ổn với kịch bản cơ bản khi số thread và vòng lặp không quá lớn.  
- Khi tăng tải lâu dài (kịch bản 3), trang giới hạn tốc độ và nhiều request thất bại, dẫn đến Error Rate cao.  
- Để kiểm thử hiệu năng hợp lý, cần:  
  1. Giảm tốc độ gửi request trong kịch bản tùy chỉnh (thêm Timer).  
  2. Chọn đúng endpoint tồn tại (ví dụ `/features`, `/topics`) hoặc sử dụng API chính thức nếu muốn đo throughput liên tục.  
  3. Xem xét cơ chế xác thực hoặc dùng token GitHub để tránh rate-limiting.

