# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Đỗ Đức Cường - Mã học viên: 2A202601455

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> *Nếu mặc định "changeme" → app chạy bình thường, endpoint sống, health check xanh — nhưng ai cũng gọi được /ask miễn phí bằng khóa mặc định đó, tới lúc phát hiện thì hóa đơn LLM đã tính*

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> *{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud.","user_id":"sv-test","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}. Log có field history_length, tokens, cost_usd để track tính tổng chi phí hoặc thiết lập cảnh báo tự động.*

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
| 1 stage (bản đầu) | 67.9MB |
| Multi-stage | 63.7MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> *multi-stage bỏ được build tool không mang sang stage runtime nên dung lượng thấp hơn.*

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> *Sửa 1 ký tự app/main.py, build lại, layer COPY requirements.txt + RUN pip install báo CACHED, chỉ layer COPY app/ trở đi rebuild. Nếu đảo COPY . . lên trước pip install: mọi thay đổi code làm invalidate cache ngay từ đầu, pip install chạy lại full mỗi lần*

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> *Container cho quyền root: docker exec -it <container_name_or_id> sh cho phép hacker truy cập thẳng vào shell của image đang chạy với quyền root, có toàn quyền thay đổi image đang chạy. USER appuser chỉ cho quyền user khi exec vào image, không đọc ghi được file hệ thống, kết hợp với multi stage chỉ lưu file đã build khiến việc can thiệp vào image đang chạy khó khăn hơn.*

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> *Đồng hồ reset ở giây 00: gửi 10 request lúc phút X giây 59, rồi 10 request nữa lúc phút X+1 giây 00 dẫn đến 20 request trong 2 giây thực tế mà mỗi cửa sổ đều hợp lệ (≤10/cửa sổ). Sliding window không có lỗ hổng này vì luôn đếm 60 giây gần nhất tính từ hiện tại.*

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> *Rate limit đếm số lượng request/thời gian, cost guard đếm tiền. Rate limit cho qua nhưng cost guard chặn: user gửi ít request (trong hạn mức) nhưng câu hỏi rất dài, token nhiều dẫn đến 1 request đã tốn hết ngân sách tháng. Ngược lại, user gửi request nhỏ, rẻ, nhưng gửi dồn dập vượt quota/phút khiến rate limit chặn dù tiền còn dư.*

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> *Nếu gộp làm 1 và cả 3 container check Redis: Khi Redis rớt 30s, cả 3 container đồng loạt fail check này. Orchestrator coi cả 3 unhealthy nên rút traffic hoặc restart, dẫn đến toàn bộ service down dù code app vẫn chạy tốt (chỉ mất Redis)*

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> *Nếu lưu trong dict Python (in-process), history_length mỗi request có thể rơi vào 1 trong 3 container khác nhau (round robin). Có lúc thấy history_length tăng đều, có lúc đột ngột về thấp/reset vì rơi vào container "chưa biết" cuộc hội thoại đó. Với Redis (shared state), history_length tăng đều đặn bất kể request rơi vào container nào.*

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Không gặp lỗi*
