# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: điền câu trả lời bên dưới mỗi câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Đức Triệu  Mã học viên: 2A202602978

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Tình huống: Khi deploy ứng dụng lên Cloud mà quên cấu hình biến `AGENT_API_KEY`. Nếu để mặc định `"changeme"`, app vẫn chạy thành công và báo Healthy, nhưng bất kỳ ai thử key mặc định này đều gọi được API làm phát sinh chi phí LLM khổng lồ. Việc "chết sớm" khiến app crash ngay lúc khởi động, giúp phát hiện và sửa lỗi ngay trước khi service công khai ra Internet.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Log JSON thu được:
`{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T04:55:05.123456+00:00", "user_id": "sv-test", "tokens_in": 3, "tokens_out": 35, "cost_usd": 0.00002145}`

> Hai việc làm được:
> 1. Lọc và tự động phát cảnh báo (Alert) trên hệ thống thu gom log khi `cost_usd > 0.05` hoặc `level == "error"`.
> 2. Thống kê số liệu số học (Aggregation) như tính tổng chi phí `SUM(cost_usd)` hoặc đo trung bình token `AVG(tokens_out)` theo từng `user_id`.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.02 GB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng cắt giảm được (~750MB) là nhờ đổi sang base image `python:3.11-slim` và dùng kỹ thuật Multi-stage build để loại bỏ toàn bộ bộ công cụ biên dịch (GCC, Make, C headers), cache rác của `pip` ở stage builder, không bị đóng gói vào image runtime.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Layer `COPY requirements.txt` và `RUN pip install` được dùng lại từ cache; các layer từ `COPY app/ ./app/` trở về sau phải chạy lại. Nếu đặt `COPY . .` lên trước `RUN pip install`, mỗi lần sửa một dòng code trong app sẽ làm hỏng cache của `COPY . .`, ép Docker phải cài lại toàn bộ thư viện từ đầu rất tốn thời gian.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi sự kiện: Lỗ hổng RCE trong code Python ➔ Kẻ tấn công thực thi lệnh với quyền root của container ➔ Tận dụng quyền root này để khai thác lỗ hổng container breakout ➔ Chiếm quyền kiểm soát root trên máy host. Lệnh `USER appuser` cắt đứt chuỗi này ngay ở bước thứ 2: kẻ tấn công chỉ có quyền của user thường không có đặc quyền hệ thống trong container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa 20 request trong 2 giây. Người dùng gửi 10 request ở giây 10:00:59 (cuối phút thứ 1) và 10 request ở giây 10:01:00 (đầu phút thứ 2). Cả hai lần gửi đều đúng hạn mức 10 req/phút của từng phút đồng hồ riêng biệt.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> - Khác nhau: Rate limit giới hạn tần suất/số lượng request, Cost guard giới hạn số tiền/ngân sách.
> - Rate limit cho qua, Cost guard chặn: User gửi 1 request/phút (đúng rate limit) nhưng request đó chứa prompt siêu dài tốn $15 (vượt ngân sách $10/tháng).
> - Cost guard cho qua, Rate limit chặn: User mới tiêu $0.01 (còn ngân sách) nhưng bấm gửi liên tục 20 request trong 5 giây (vượt hạn mức 10 req/phút).

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Thứ tự sự kiện: (1) Redis mất kết nối 30s ➔ (2) Endpoint gộp trả về 503 ➔ (3) Orchestrator hiểu rằng container bị hỏng nên kill và restart cả 3 container agent ➔ (4) Cụm ứng dụng bị khởi động lại liên tục và sụp đổ hoàn toàn (cascading failure) mặc dù process Python vẫn khỏe mạnh.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Con số `history_length` sẽ tăng giảm thất thường (ví dụ 0 -> 0 -> 1 -> 0 -> 2) thay vì tăng đều, vì mỗi request được Nginx điều phối ngẫu nhiên sang 1 trong 3 container agent khác nhau, và mỗi container chỉ giữ một dict lịch sử riêng cục bộ.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Thông báo lỗi: `exec: docker-entrypoint.sh: not found` làm container deploy bị crash. Nguyên nhân tìm thấy khi xem `railway logs`: CLI đang link nhầm vào service `Redis` nên lệnh `railway up` đã push code Python đè lên service Redis. Cách sửa: Chạy `railway service link agent` để chuyển link CLI về service `agent` rồi deploy lại.
